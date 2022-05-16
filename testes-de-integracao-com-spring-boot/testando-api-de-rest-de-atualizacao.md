# Testando API REST Remoção de Entidades

Para alguns Sistemas executar uma logica de remoção pode ser altamente complexo, principalmente quando essa remoção depende de integração entre sistemas, seja uma mensagem em uma fila, uma linha em um arquivo de texto ou simplesmente um registro no banco de dados. Independente da integração que o sistema utiliza, um bom teste deverá verificar se o efeito colateral é executado de fato.

Em APIs REST é comum que a integração seja feita através de uma chamada HTTP, que deve respeitar a semantica de um verbo e ter um Path bem definido. Também se espera que um efeito colateral seja lançado ao servidor ou não. Ao observar estes detalhes modelar testes de integração pode se tornar simples, pois, devemos direcionar nossa atenção a realizar uma chamada HTTP para verbo DELETE, para um determinado caminho, caso o recuso não exista garantiremos que o fluxo definido seja respeitado, caso contrario garantiremos que o fluxo de sucesso foi executado e que o efeito colateral corresponde ao esperado.

Um bom exemplo de Teste de Integração em API REST  é quando desejamos remover uma Pessoa. Para melhor compreensão, veja o controller abaixo e vamos escrever bons testes utilizando o **`Spring Test`**.

```java
@RestController
@RequestMapping("/pessoas/{id}")
public class RemoverPessoaController{
    private final PessoaRepository repository;

    public RemoverPessoaController(PessoaRepository repository){
        this.repository=repository;
    }


    @PatchMapping
    @Transactional
    public ResponseEntity<?> remover(@PathVariable Long id){
        Pessoa pessoa = repository.findById(id)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Pessoa não cadastrada"));

        repository.delete(pessoa);
        
        return ResponseEntity.noContet().build();
    }
}
```
Para testar um sistemas simples como o acima, devemos nos atentar aos seguintes casos: não é possivel deletar um registro não existente e quando um registro é removido. Dado a estas condições trabalharemos com os seguintes cenarios:

1. nao deve remover uma pessoa não cadastrada
2. deve remover uma pessoa

Agora que já temos o cenarios, vamos criar nossa classe de teste e configurar o ambiente para teste de integração.


```java
@SpringBootTest
@AutoConfigureMockMvc(printOnlyOnFailure = false)
@ActiveProfiles("test")
class  RemoverPessoaControllerTest {
    @Autowired
    private ObjectMapper mapper;
    @Autowired
    private PessoaRepository repository;
    @Autowired
    private MockMvc mockMvc;
    

    @BeforeEach
    void setUp() {
        repository.deleteAll();
    }

}
```

Agora que temos o suporte a serialização e desserialização de Json com `ObjectMapper`, e cliente HTTP para fazer as chamadas a nossa API com MockMvc. Vamos para nosso primeiro cenario, que é onde vamos detalhar os dados de uma cidade. 

## Test: **`deve remover uma pessoa`**

A construção deste cenario é bastante simples, como pré requisito do nosso testes precisamos de uma pessoa cadastrada no banco de dados e o seu id. 


```java
@Test
@DisplayName("deve remover uma pessoa")
void test() throws Exception{

    Pessoa pessoa = new ("Jordi Henrique", "12312312345", "jordi.silva@email.com");

    repository.save(pessoa);


    MockHttpServletRequestBuilder request = delete("/pessoas/{id}", pessoa.getId())
                .contentType(MediaType.APPLICATION_JSON);
            
     mockMvc.perform(request)
                .andExpect(
                        status().isNoContent()
                );
                

    
    assertFalse(repository.existsById(id),"nao deve existir registro de pessoa pra esse id");
}
```

Após persistir a pessoa que sera removida, nos concentramos em montar a requisição `HTTP`, informado o path no metodo referente ao verbo **`DELETE`** utilizando a API  do MockMvc. Após invocar a requisição nosso objetivo é validar se o `Status HTTP` retornado corresponde ao `noContent (204)`.


Após a validação do status da resposta, devemos averiguar se o efeito colateral desta chamada HTTP é efetuado, ou seja, se o registro da Pessoa é removido do Banco de Dados. Então através do Junit validamos se o não existe um registro na tabela de pessoa com id informado. 

## Test: **`nao deve remover uma pessoa não cadastrada`**

Neste teste o objetivo é verificar se quando um id inexistente é informado, o retorno da nossa API corresponde a uma Resposta HTTP com Status Not Found (404). Neste teste encontraremos um pequeno desafio pois o MockMvc não executa de fato um Container Servelet, ele apenas simula a execução e dado este motivo, o ResolverException default do Spring não é executado, e a mensagem de erro definida nas execeptions, não são informadas ao corpo da Resposta HTTP, e sim no campo `erro mensage`, para saber mais leia este [artigo](https://github.com/spring-projects/spring-framework/issues/17290).

Dado a esta limitação do MockMvc, teremos que mudar a maneira que executamos as verificações em nossos assert's.


```java
@Test
@DisplayName("nao deve remover uma pessoa não cadastrada")
void test2()  throws Exception {
    MockHttpServletRequestBuilder request = delete("/pessoas/{id}", Integer.MAX_VALUE).
                contentType(MediaType.APPLICATION_JSON);
    
    Exception resolvedException = mockMvc.perform(request)
                .andExpect(
                        status().isNotFound()
                )
                .andReturn()
                .getResolvedException();

    assertNotNull(resolvedException);
    assertEquals(ResponseStatusException.class,resolvedException.getClass());
    assertEquals("Pessoa não cadastrada",((ResponseStatusException) resolvedException).getReason());
}
```

Para verificar se nosso fluxo foi executado, iremos trabalhar com o metodo `getResolvedException` do MockMvc, este metodo retorna em uma Exception generica, qual a exceção que foi tratada pelo `ExceptionHandler` do MockMvc.

Dado que uma exceção  pode nunca ser lançada, é necessário verificar se a resolvedException não é nula. Após esta verificação podemos partir para uma validação de tipo, o `Controller` lança uma `ResponseStatusException`, com Reason: `Pessoa não cadastrada`, e Status 404, que foi verificado pelo MocvMvcResultMatchers anteriormente.

Através do assertEquals, validamos que nossa exceção foi lançada corretamente, agora é validar se a razão (Reason) corresponde ao esperado. O campo Reason não é acessivel pelo tipo Exception, então com  a garantia que a Exceção é do tipo `ResponseStatusException`, devemos realizar um casting (mudança de tipo)  para aplicar o assert.



### Soluções possiveis para Limitação do Container Servelet Simulado do MockMvc

Implementar um ExceptionHandler a nivel de contexto de Test, ou se preferir implementar no ExceptionHandler do seu aplicativo um metodo para tratar `ResponseStatusException`. 

Por simplicidade resolvemos abraçar a limitação do MockMvc e validar se a nossa exception foi lançada, porém, você deverá discutir com sua equipe e juntos tomara a decisão de qual caminho tomar. 

Para ajudar a visualizar outras soluções, observe estes links: 

- [MockMvc fails to capture the Exception message](https://github.com/spring-projects/spring-framework/issues/17290)
- [MockMvc support for testing errors](https://github.com/spring-projects/spring-framework/issues/17290)
- [Cannot properly test ErrorController Spring Boot](https://stackoverflow.com/questions/52925700/cannot-properly-test-errorcontroller-spring-boot)

