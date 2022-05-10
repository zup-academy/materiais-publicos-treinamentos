# Testando API REST  de Listagem de Dados

As APIs de consulta de dados podem divergir da maioria dos outros casos, como escrita e atualização, porque esta se referenciam que se não houver registro devemos retornar uma coleção vazia, e caso tenha, validamos a ordem se necessário, o tamanho da coleção, e as informações de cada objeto. Caso a logica de negocio seja complexa e cotenha varios fluxos, devemos verificar cada um destes branch (caminhos).

Agora que estamos cientes dos principios para testar coleções podemos aplica-los em nossos testes de integração de listagem, veja abaixo o **controller** e vamos testa-lo utilizando o **`Spring Test`**.

```java
public class CidadeResponse{
    private String nome;
    private BigDecimal habitantes;

    public CidadeResponse(){}

    public CidadeResponse(Cidade cidade){
        this.nome=cidade.getNome();
        this.habitantes=cidade.getHabitantes();
    }
    //getters omitidos
}

@RestController
@RequestMapping("/estado/{id}/cidades")
public class ListarCidadesDoEstadoController{
    private final CidadeRepository cidadeRepository;
    private final EstadoRepository estadoRepository;

    public ListarCidadesDoEstadoController(CidadeRepository cidadeRepository, EstadoRepository estadoRepository){
        this.cidadeRepository=cidadeRepository;
        this.estadoRepository=estadoRepository;
    }


    @GetMapping
    public ResponseEntity<?> listar(@PathVariable Long id){

        if(!estadoRepository.existsById(id)) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Estado nao cadastrado");
        }

        List<CidadeResponse> cidades = cidadeRepository.findAllByEstadoId(id)
                    .stream()
                    .map(CidadeResponse::new)
                    .collect(Collectors.toList());
        
        return ResponseEntity.ok(cidades);
    }
}
```

O Objetivo desta API é listar as cidades que pertecem a um estado cadastrado no sistema, um ponto interessante é que dado que um estado não exista simplismente não pode ser retornado uma coleção vazia, pois daria margem ao usuario interpretar que este Estado não possui Cidades, quando na verdade não existe Estado cadastrado para este id.

Então trabalharemos com os seguintes cenarios:

1. deve listar as cidades de um estado
2. deve retornar uma coleção vazia para um estado sem cidades
3. nao deve retornar as cidades de um estado não cadastrado.

Agora que já temos o cenarios, vamos criar nossa classe de teste e configurar o ambiente para teste de integração.


```java
@SpringBootTest
@AutoConfigureMockMvc(printOnlyOnFailure = false)
@ActiveProfiles("test")
class  ListarCidadesDoEstadoControllerTest {
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

Agora que temos o suporte a serialização e desserialização de Json com `ObjectMapper`, e cliente HTTP para fazer as chamadas a nossa API com MockMvc. Vamos para nosso primeiro cenario, que é onde vamos verificar se as Cidades do Estado estão sendo listadas. 

## Test: **`deve listar as cidades de um estado`**

Para realizar este testes, vamos precisar disponibilizar as pre condições, que é existir cidades cadastradas para este estado, então iniciaramos nosso teste preparando o ambiente. Em seguinda iremos realizar a chamada HTTP, e verificar o `HTTP Status`, por fim iremos desserializar o json de resposta e fazer os asserts.
```java
@Test
@DisplayName("deve listar as cidades de um estado")
void test(){
    Cidade ibia = new Cidade("Ibia", new BigDecimal("15000"),estado);
    Cidade beloHorizonte = new Cidade("Belo Horizonte", new BigDecimal(" 2722000"),estado);

    List<Cidade> cidades = List.Of(ibia, beloHorizonte);

    cidadeRepository.saveAll(cidades);

    MockHttpServletRequestBuilder request = get("/estado/{id}/cidades", estado.getId()).
                contentType(MediaType.APPLICATION_JSON);
    
    String payload = mockMvc.perform(request)
                .andExpect(
                        status().isOk()
                )
                .andReturn()
                .getResponse()
                .getContentAsString(StandardCharsets.UTF_8);

    TypeFactory typeFactory = mapper.getTypeFactory();

    List<CidadeResponse> response = mapper.readValue(
        payload,
        typeFactory.constructCollectionType(List.class, CidadeResponse.class)
    );

    assertThat(response)
             .hasSize(2)
             .extracting("nome","habitantes")
             .contains(
                 new Tuple(ibia.getNome(), ibia.getHabitantes().setScale(2)),
                 new Tuple(beloHorizonte.getNome(), beloHorizonte.getHabitantes().setScale(2))
             );   
}
```

Como precisamos instanciar uma coleção de Objetos, precisamos dizer ao Jackson qual a `Collection` e qual o tipo da mesma. O objeto responsavel por definir estas construções é o TypeFactory.

O `assetJ` nos permite informar qual objeto queremos realizar os asserts, e definir de maneira personalizada qual verificação e quais campos queremos testar. Primeiro verificamos se a resposta contém 2 cidades, e logo apos validamos se o valor de `nome` e `habitantes` correspondem aos valores dos objetos que foram cadastrados no Banco de Dados.

## Test: **`deve retornar uma coleção vazia para um estado sem cidades`**


No cenario que iremos testar, os pre requisitos são, existir um estado cadastrado, e este estado não deve conter cidades. Nosso objetivo é verificar se uma coleção vazia é retornada na Response HTTP.


```java
@Test
@DisplayName("deve retornar uma coleção vazia para um estado sem cidades")
void test1(){
    MockHttpServletRequestBuilder request = get("/estado/{id}/cidades", estado.getId()).
                contentType(MediaType.APPLICATION_JSON);
    
    String payload = mockMvc.perform(request)
                .andExpect(
                        status().isOk()
                )
                .andReturn()
                .getResponse()
                .getContentAsString(StandardCharsets.UTF_8);

    TypeFactory typeFactory = mapper.getTypeFactory();

    List<CidadeResponse> response = mapper.readValue(
        payload,
        typeFactory.constructCollectionType(List.class, CidadeResponse.class)
    );

    assertTrue(response.isEmpty());
}
```

## Test: **`nao deve retornar as cidades de um estado não cadastrado`**

Neste teste o objetivo é verificar se quando um id inexistente é informado, o retorno da nossa API corresponde a uma Resposta HTTP com Status Not Found (404). Neste teste encontraremos um pequeno desafio pois o MockMvc não executa de fato um Container Servelet, ele apenas simula a execução e dado este motivo, o ResolverException default do Spring não é executado, e a mensagem de erro definida nas execeptions, não são informadas ao corpo da Resposta HTTP, e sim no campo `erro mensage`, para saber mais leia este [artigo](https://github.com/spring-projects/spring-framework/issues/17290).

Dado a esta limitação do MockMvc, teremos que mudar a maneira que executamos as verificações em nossos assert's.


```java
@Test
@DisplayName("nao deve retornar as cidades de um estado não cadastrado")
void test2(){
    MockHttpServletRequestBuilder request = get("/estado/{id}/cidades", Integer.MAX_VALUE).
                contentType(MediaType.APPLICATION_JSON);
    
    Exception resolvedException = mockMvc.perform(request)
                .andExpect(
                        status().isNotFound()
                )
                .andReturn()
                .getResolvedException();

    assertNotNull(resolvedException);
    assertEquals(ResponseStatusException.class,resolvedException.getClass());
    assertEquals("Zupper nao cadastrado",((ResponseStatusException) resolvedException).getReason());
}
```

Para verificar se nosso fluxo foi executado, iremos trabalhar com o metodo `getResolvedException` do MockMvc, este metodo retorna em uma Exception generica, qual a exceção que foi tratada pelo `ExceptionHandler` do MockMvc.

Dado que uma exceção  pode nunca ser lançada, é necessário verificar se a resolvedException não é nula. Após esta verificação podemos partir para uma validação de tipo, o `Controller` lança uma `ResponseStatusException`, com Reason: `Estado nao cadastrado`, e Status 404, que foi verificado pelo MocvMvcResultMatchers anteriormente.

Através do assertEquals, validamos que nossa exceção foi lançada corretamente, agora é validar se a razão (Reason) corresponde ao esperado. O campo Reason não é acessivel pelo tipo Exception, então com  a garantia que a Exceção é do tipo `ResponseStatusException`, devemos realizar um casting (mudança de tipo)  para aplicar o assert.


### Soluções possiveis para Limitação do Container Servelet Simulado do MockMvc

Implementar um ExceptionHandler a nivel de contexto de Test, ou se preferir implementar no ExceptionHandler do seu aplicativo um metodo para tratar `ResponseStatusException`. 

Por simplicidade resolvemos abraçar a limitação do MockMvc e validar se a nossa exception foi lançada, porém, você deverá discutir com sua equipe e juntos tomara a decisão de qual caminho tomar. 

Para ajudar a visualizar outras soluções, observe estes links: 

- [MockMvc fails to capture the Exception message](https://github.com/spring-projects/spring-framework/issues/17290)
- [MockMvc support for testing errors](https://github.com/spring-projects/spring-framework/issues/17290)
- [Cannot properly test ErrorController Spring Boot](https://stackoverflow.com/questions/52925700/cannot-properly-test-errorcontroller-spring-boot)

