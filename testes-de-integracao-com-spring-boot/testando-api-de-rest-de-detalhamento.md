# Testando API REST  de Detalhamento de Dados

As APIs de detalhamento de dados tem como objetivo exibir de maneira fragamentada as informações de um recurso. Estas APIs normalemnte contém o fluxo de sucesso, que é quando as informações são retornadas. E o fluxo onde o recurso a qual foi requisitado o detalhamento não existe.  Nestes casos, optaremos para o caso de sucesso validar se o Status HTTP da Resposta e as informações do Corpo correspondem ao esperado. Já quando um recurso não existe deveremos validar a razão e Status HTTP da resposta.

A melhor forma de aprender a destinguir cenarios e escrever bons testes é na pratica, observe abaixo o **controller** de Detalhamento de uma Cidade e  vamos testa-lo utilizando o **`Spring Test`**.

```java
public class DetalharCidadeResponse{
    private String nome;
    private String estado;
    private LocalDate dataFundacao;
    private BigDecimal habitantes;

    public CidadeResponse(){}

    public CidadeResponse(Cidade cidade){
        this.nome=cidade.getNome();
        this.estado=cidade.getEstado().getNome();
        this.dataFundacao=cidade.getDataFundacao();
        this.habitantes=cidade.getHabitantes();
    }
    //getters omitidos
}

@RestController
@RequestMapping("/cidades/{id}")
public class DetalharCidadeController{
    private final CidadeRepository repository;

    public DetalharCidadeController(CidadeRepository repository){
        this.repository=repository;
    }


    @GetMapping
    public ResponseEntity<?> detalhar(@PathVariable Long id){
        Cidade cidade = repository.findById(id)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Cidade nao cadastrada"));
        
        return ResponseEntity.ok(new DetalharCidadeResponse(cidade));
    }
}
```

Esta aplicação é bastante simples, se consiste em dois casos, um para quando uma cidade cadastrada é detalhada, e outro para quando é solicitado o detalhamento de uma cidade não cadastrada.

Então trabalharemos com os seguintes cenarios:

1. deve detalhar uma cidade
2. nao deve detalhar uma cidade não cadastrada

Agora que já temos o cenarios, vamos criar nossa classe de teste e configurar o ambiente para teste de integração.


```java
@SpringBootTest
@AutoConfigureMockMvc(printOnlyOnFailure = false)
@ActiveProfiles("test")
class  DetalharCidadeControllerTest {
    @Autowired
    private ObjectMapper mapper;
    @Autowired
    private CidadeRepository cidadeRepository;
    @Autowired
    private MockMvc mockMvc;
    @Autowired
    private EstadoRepository estadoRepository;
    private Estado estado;

    @BeforeEach
    void setUp() {
        cidadeRepository.deleteAll();
        estadoRepository.deleteAll();
        this.estado= new Estado("Minas Gerais","MG");
        estadoRepository.save(estado);
    }

}
```

Agora que temos o suporte a serialização e desserialização de Json com `ObjectMapper`, e cliente HTTP para fazer as chamadas a nossa API com MockMvc. Vamos para nosso primeiro cenario, que é onde vamos detalhar os dados de uma cidade. 

## Test: **`deve detalhar uma cidade`**

Para realizar este testes, vamos precisar disponibilizar a cidade que iremos detalhar, então iniciaramos nosso teste preparando o ambiente. Em seguinda iremos realizar a chamada HTTP, e verificar o `HTTP Status`, por fim iremos desserializar o json de resposta e fazer os asserts.


```java
@Test
@DisplayName("deve detalhar uma cidade")
void test(){
    
    Cidade beloHorizonte = new Cidade("Belo Horizonte", LocalDate.of(1897,12,12), new BigDecimal(" 2722000"),estado);

    cidadeRepository.save(beloHorizonte);

    MockHttpServletRequestBuilder request = get("/cidades/{id}", beloHorizonte.getId()).
                contentType(MediaType.APPLICATION_JSON);
    
    String payload = mockMvc.perform(request)
                .andExpect(
                        status().isOk()
                )
                .andReturn()
                .getResponse()
                .getContentAsString(StandardCharsets.UTF_8);

    

    DetalharCidadeResponse response = mapper.readValue(
        payload, DetalharCidadeResponse.class
    );

    assertThat(response)
             .extracting("nome","estado","dataFundacao","habitantes")
             .contains(
                beloHorizonte.getNome(),
                beloHorizonte.getEstado().getNome(),
                beloHorizonte.getDataFundacao(),
                beloHorizonte.getHabitantes().setScale(2)
             )
             ;   
}
```

Primeiramente validamos com `MockMvc` se o Status da Resposta corresponde ao `Ok (200)`, dado isso proseguimos e atribuimos o `Json` do corpo a string payload. Fazemos a desserialização do `Json` através do `ObjectMapper` em um objeto `DetalharCidadeResponse`.

Agora fazemos os asserts através do `AssertJ` para validar se os campos: nome, estado, dataFundacao e habitantes correspondem aos valores que foram persistidos no Banco de Dados.


## Test: **`nao deve detalhar uma cidade não cadastrada`**

Neste teste o objetivo é verificar se quando um id inexistente é informado, o retorno da nossa API corresponde a uma Resposta HTTP com Status Not Found (404). Neste teste encontraremos um pequeno desafio pois o MockMvc não executa de fato um Container Servelet, ele apenas simula a execução e dado este motivo, o ResolverException default do Spring não é executado, e a mensagem de erro definida nas execeptions, não são informadas ao corpo da Resposta HTTP, e sim no campo `erro mensage`, para saber mais leia este [artigo](https://github.com/spring-projects/spring-framework/issues/17290).

Dado a esta limitação do MockMvc, teremos que mudar a maneira que executamos as verificações em nossos assert's.


```java
@Test
@DisplayName("nao deve detalhar uma cidade não cadastrada")
void test2(){
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
    assertEquals("Cidade não cadastrada",((ResponseStatusException) resolvedException).getReason());
}
```

Para verificar se nosso fluxo foi executado, iremos trabalhar com o metodo `getResolvedException` do MockMvc, este metodo retorna em uma Exception generica, qual a exceção que foi tratada pelo `ExceptionHandler` do MockMvc.

Dado que uma exceção  pode nunca ser lançada, é necessário verificar se a resolvedException não é nula. Após esta verificação podemos partir para uma validação de tipo, o `Controller` lança uma `ResponseStatusException`, com Reason: `Cidade nao cadastrada`, e Status 404, que foi verificado pelo MocvMvcResultMatchers anteriormente.

Através do assertEquals, validamos que nossa exceção foi lançada corretamente, agora é validar se a razão (Reason) corresponde ao esperado. O campo Reason não é acessivel pelo tipo Exception, então com  a garantia que a Exceção é do tipo `ResponseStatusException`, devemos realizar um casting (mudança de tipo)  para aplicar o assert.


### Soluções possiveis para Limitação do Container Servelet Simulado do MockMvc

Implementar um ExceptionHandler a nivel de contexto de Test, ou se preferir implementar no ExceptionHandler do seu aplicativo um metodo para tratar `ResponseStatusException`. 

Por simplicidade resolvemos abraçar a limitação do MockMvc e validar se a nossa exception foi lançada, porém, você deverá discutir com sua equipe e juntos tomara a decisão de qual caminho tomar. 

Para ajudar a visualizar outras soluções, observe estes links: 

- [MockMvc fails to capture the Exception message](https://github.com/spring-projects/spring-framework/issues/17290)
- [MockMvc support for testing errors](https://github.com/spring-projects/spring-framework/issues/17290)
- [Cannot properly test ErrorController Spring Boot](https://stackoverflow.com/questions/52925700/cannot-properly-test-errorcontroller-spring-boot)

