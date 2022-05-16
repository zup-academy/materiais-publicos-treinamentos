# Testando API REST de Atualizacao de Dados


Os testes de integração tem como objetivo validar se a junção das partes trabalham de maneira correta e harmoniosa, isto implica que validar se o comportamento de cada componente que se integra é executado conforme o esperado quando trabalham juntos. 
 
APIs de atualização de dados tem um papel fundamental em um sistema, é através destas que conseguimos obter determinado recurso e alterar seu estado. O que é muito comum é que o estado dos recursos ficam armazenados em repositorios de dados, então a API REST recebe uma solicitação de alteração para determinado recurso pela web, faz a consulta do mesmo em Banco de Dados, processa a logica e armazena o novo estado do recurso.

Quando observamos o comportamento dito acima, podemos inferir o seguinte, dado a construção de um Teste de Integração em um fluxo de sucesso, é indispensavel a validação que garanta que o valor alterado corresponda ao esperado.

Um bom exemplo de Teste de atualização é quando dado um Produto e queremos alterar o seu preco.Para melhor compreensão, veja o controller abaixo e vamos escrever bons testes utilizando o **`Spring Test`**.

```java
public class AlterarPrecoProdutoRequest{
    @NotNull
    @Postive
    private BigDecimal preco;

    public AlterarPrecoProdutoRequest(){}

    public AlterarPrecoProdutoRequest(BigDecimal preco){
      this.preco=preco;
    }
    
    public BigDecimal getPreco(){
        return this.preco;
    }
}

@RestController
@RequestMapping("/produtos/{id}")
public class AlterarPrecoProdutoController{
    private final ProdutoRepository repository;

    public AlterarPrecoProdutoController(ProdutoRepository repository){
        this.repository=repository;
    }


    @PatchMapping
    @Transactional
    public ResponseEntity<?> atualizar(@PathVariable Long id, @RequestBody @Valid AlterarPrecoProdutoRequest request){
        Produto produto = repository.findById(id)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Produto nao cadastrado"));

        produto.setPreco(request.getPreco());
        
        return ResponseEntity.noContet().build();
    }
}
```

Esta aplicação é bastante simples, consiste em três casos, um para quando o preco do produto é atualizado, o proximo caso é para validar se caso os dados sejam invalidos uma mensagem de erro amigavel é retornada ao nosso cliente,  e por fim um caso para quando é solicitado a alteração de preço de um produto não cadastrado.

Então trabalharemos com os seguintes cenarios:

1. nao deve alterar o preco de um produto nao cadastrado
2. nao deve alterar o preco de um produto com dados invalidos
3. nao deve alterar o preco de um produto para o valor menor que zero
4. deve alterar o preco de um produto 

Agora que já temos o cenarios, vamos criar nossa classe de teste e configurar o ambiente para teste de integração.


```java
@SpringBootTest
@AutoConfigureMockMvc(printOnlyOnFailure = false)
@ActiveProfiles("test")
class  AlterarPrecoProdutoControllerTest {
    @Autowired
    private ObjectMapper mapper;
    @Autowired
    private ProdutoRepository repository;
    @Autowired
    private MockMvc mockMvc;
    

    @BeforeEach
    void setUp() {
        repository.deleteAll();
    }

}
```

Agora que temos o suporte a serialização e desserialização de Json com `ObjectMapper`, e cliente HTTP para fazer as chamadas a nossa API com MockMvc. Vamos para nosso primeiro cenario, que é onde vamos detalhar os dados de uma cidade. 

## Test: **`deve alterar o preco de um produto`**


```java
@Test
@DisplayName("deve alterar o preco de um produto")
void test() throws Exception{
    
    AlterarPrecoProdutoRequest precoRequest = new AlterarPrecoProdutoRequest(new BigDecimal(" 2722"));

    Produto produto = new ("ChromeBook", new BigDecimal("2515"));

    repository.save(produto);

    String payload = mapper.writeValueAsString(precoRequest);

    MockHttpServletRequestBuilder request = patch("/produtos/{id}", produto.getId())
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload);
    
     mockMvc.perform(request)
                .andExpect(
                        status().isNoContent()
                );
                

    Optional<Produto> possivelProduto = repository.findById(produto.getId());
    assertTrue(possivelProduto.isPresent());

    Produto produtoAposAtualizacao = possivelProduto.get();

    assertEquals(precoRequest.getPreco(),produtoAposAtualizacao.getPreco());
}
```

Primeiramente construimos o ambiente para teste, iniciando pelo json do corpo da requisição, e persistimos o produto a qual sera alterado. Em seguida é realizada a chamada através  do `MockMvc`, onde validamos se o Status da Resposta corresponde ao `noContent (204)`.

Após a validação do status da resposta, devemos averiguar se o efeito colateral desta chamada HTTP é efetuado, ou seja, se o estado do produto comporta o novo preco. Então através do Junit validamos se o preco informado no corpo da request, corresponde ao preço do objeto que foi buscado no Banco de dados após a chamada HTTP.


## Test: **`nao deve alterar o preco de um produto nao cadastrado`**

Neste teste o objetivo é verificar se quando um id inexistente é informado, o retorno da nossa API corresponde a uma Resposta HTTP com Status Not Found (404). Neste teste encontraremos um pequeno desafio pois o MockMvc não executa de fato um Container Servelet, ele apenas simula a execução e dado este motivo, o ResolverException default do Spring não é executado, e a mensagem de erro definida nas execeptions, não são informadas ao corpo da Resposta HTTP, e sim no campo `erro mensage`, para saber mais leia este [artigo](https://github.com/spring-projects/spring-framework/issues/17290).

Dado a esta limitação do MockMvc, teremos que mudar a maneira que executamos as verificações em nossos assert's.


```java
@Test
@DisplayName("nao deve alterar o preco de um produto nao cadastrado")
void test2()  throws Exception {
    MockHttpServletRequestBuilder request = get("/cidades/{id}", Integer.MAX_VALUE).
                contentType(MediaType.APPLICATION_JSON);
    
    Exception resolvedException = mockMvc.perform(request)
                .andExpect(
                        status().isNotFound()
                )
                .andReturn()
                .getResolvedException();

    assertNotNull(resolvedException);
    assertEquals(ResponseStatusException.class,resolvedException.getClass());
    assertEquals("Produto não cadastrado",((ResponseStatusException) resolvedException).getReason());
}
```

Para verificar se nosso fluxo foi executado, iremos trabalhar com o metodo `getResolvedException` do MockMvc, este metodo retorna em uma Exception generica, qual a exceção que foi tratada pelo `ExceptionHandler` do MockMvc.

Dado que uma exceção  pode nunca ser lançada, é necessário verificar se a resolvedException não é nula. Após esta verificação podemos partir para uma validação de tipo, o `Controller` lança uma `ResponseStatusException`, com Reason: `Produto não cadastrado`, e Status 404, que foi verificado pelo MocvMvcResultMatchers anteriormente.

Através do assertEquals, validamos que nossa exceção foi lançada corretamente, agora é validar se a razão (Reason) corresponde ao esperado. O campo Reason não é acessivel pelo tipo Exception, então com  a garantia que a Exceção é do tipo `ResponseStatusException`, devemos realizar um casting (mudança de tipo)  para aplicar o assert.

## Test: **`nao deve alterar o preco de um produto com dados invalidos`**

O objetivo deste teste é a garantir que se caso o campo preco do corpo da requisição HTTP seja informado nulo, o software ira retornar um Status 400 e a mensagem amigavel para o cliente.

```java
@Test
@DisplayName("nao deve alterar o preco de um produto com dados invalidos")
void test3()  throws Exception {
    
    AlterarPrecoProdutoRequest precoRequest = new AlterarPrecoProdutoRequest(null);

    Produto produto = new ("ChromeBook", new BigDecimal("2515"));

    repository.save(produto);

    String payload = mapper.writeValueAsString(precoRequest);

    MockHttpServletRequestBuilder request = patch("/produtos/{id}", produto.getId())
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload);
    
    String payloadResponse mockMvc.perform(request)
                .andExpect(
                        status().isBadRequest()
                )
                .andReturn()
                .getResponse()
                .getContentAsString(StandardCharsets.UTF_8);

    MensagemDeErro mensagemDeErro = mapper.readValue(payloadResponse, MensagemDeErro.class);

     assertThat(mensagemDeErro)
                .hasSize(1)
                .contains("O campo Preco nao deve ser nulo");
                
}
```

Para atingir nosso objetivo, iniciamos o testes criando o cenario, criando um json com dados invalidos e persistindo no banco o produto. Em seguida disparamos a requisição para o verbo Patch e o endereço do serviço através do `MockMvc`, dando sequencia ao testes validamos se o status retornado é `Bad Request (400)`, e atribuimos o json que esta no corpo da Resposta HTTP a uma variavel chamda de payloadResponse. O proximo passo é desserializar o json no objeto que definimos o tratamento das exceções da Bean Validation, que por vez acumula as mensagens. Por fim verificamos se apenas uma mensagem existe na lista, e que ela corresponde a validação que foi violada.


## Test: **`nao deve alterar o preco de um produto para um valor abaixo de zero`**

O objetivo deste teste é a garantir que se caso o campo preco do corpo da requisição HTTP seja informado com valor abaixo de zero, o software ira retornar um Status 400 e a mensagem amigavel para o cliente.

```java
@Test
@DisplayName("nao deve alterar o preco de um produto para um valor abaixo de zero")
void test3()  throws Exception {
    
    AlterarPrecoProdutoRequest precoRequest = new AlterarPrecoProdutoRequest(new BigDecimal("-1"));

    Produto produto = new ("ChromeBook", new BigDecimal("2515"));

    repository.save(produto);

    String payload = mapper.writeValueAsString(precoRequest);

    MockHttpServletRequestBuilder request = patch("/produtos/{id}", produto.getId())
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload);
    
    String payloadResponse mockMvc.perform(request)
                .andExpect(
                        status().isBadRequest()
                )
                .andReturn()
                .getResponse()
                .getContentAsString(StandardCharsets.UTF_8);

    MensagemDeErro mensagemDeErro = mapper.readValue(payloadResponse, MensagemDeErro.class);

     assertThat(mensagemDeErro)
                .hasSize(1)
                .contains("O campo Preco deve conter um valor acima de zero");
                
}
```

Para atingir nosso objetivo, iniciamos o testes criando o cenario, criando um json com preco abaixo de zero, em seguida é persistino no banco o produto que sera atualizado pela requisição. O proximo passo é construir a requisição para o verbo Patch e o endereço do serviço através do `MockMvc`, e ao disparar a chamada, iremos validar se o status retornado é `Bad Request (400)`, e atribuimos o json que esta no corpo da Resposta HTTP a uma variavel chamda de payloadResponse. O proximo passo é desserializar o json no objeto que definimos o tratamento das exceções da Bean Validation, que por vez acumula as mensagens. Por fim verificamos se apenas uma mensagem existe na lista, e que ela corresponde a validação que foi violada.

### Soluções possiveis para Limitação do Container Servelet Simulado do MockMvc

Implementar um ExceptionHandler a nivel de contexto de Test, ou se preferir implementar no ExceptionHandler do seu aplicativo um metodo para tratar `ResponseStatusException`. 

Por simplicidade resolvemos abraçar a limitação do MockMvc e validar se a nossa exception foi lançada, porém, você deverá discutir com sua equipe e juntos tomara a decisão de qual caminho tomar. 

Para ajudar a visualizar outras soluções, observe estes links: 

- [MockMvc fails to capture the Exception message](https://github.com/spring-projects/spring-framework/issues/17290)
- [MockMvc support for testing errors](https://github.com/spring-projects/spring-framework/issues/17290)
- [Cannot properly test ErrorController Spring Boot](https://stackoverflow.com/questions/52925700/cannot-properly-test-errorcontroller-spring-boot)

