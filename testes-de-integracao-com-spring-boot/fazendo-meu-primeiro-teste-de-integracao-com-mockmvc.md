# Construindo Testes de Integração com Mockmvc do Spring Test

Nos como desenvolvedores buscamos cada vez mais construir software se maneira mais produtiva e segura, durante este processo nos esbarramos em diversas metodologias, como o uso de frameworks como Spring e Spring Boot que oferecem diversos modulos e configurações que permitem que o desenvolverdor se restrinja a codificar o que gere mais valor ao cliente. Já pelo lado da segurança do que foi implementado é possivél aplicar testes que revelem bugs e que garantam que a funcionalidade é executada de acordo com o projetado.

Ao definir um conjuto de testes para determinado sistema, o desenvolvedor olha sempre para duas dimensões, os testes de unidade, que focam em verificar que uma unidade de codigo, funciona corretamente apartir de determinado cenario. E os Testes de Integração que tem como responsabilidade garantir que as unidades trabalham corretamente quando executam juntas.

Olhando para um Sistema Web no padrão MVC ou uma API REST na hora de produzir testes de integração devemos nos atentar em garantir que não apenas as classes escritas funcionem como esperado, mas que também o serviço esteja diponivel em determinado endereço junto a semântica de um verbo `HTTP`. E que se seja feito alguma operação junto a um Banco de Dados (BD), a mesma funcioe de acordo com o esperado.

## Testando um Serviço de Cadastro de Pessoas

Abaixo você encontrará uma API REST que é responsavel por cadastrar informações de uma Pessoa.


```java
public class PessoaRequest{
    @NotBlank
    private String nome;

    @NotBlank
    @CPF(message="deve estar no formato valido")
    private String cpf;

    //construtor, getters e setters otimitos
}




@RestController
@RequestMapping("/api/pessoas")
public class CadastrarPessoaController{
    private final PessoaRepository repository;

    public CadastrarPessoaController(PessoaRepository repository){
        this.repository=repository;
    }

    @PostMapping
    public ResponseEntity<?> cadastrar(@RequestBody @Valid PessoaRequest request, UriComponentsBuilder uriComponentsBuilder){
        Pessoa pessoa = request.paraPessoa();

        repository.save(pessoa);

        URI location = uriComponentsBuilder.path("/api/pessoas/{id}")
            .buildExpand(pessoa.getId())
            .toUri();

        return ResponseEntity.created(location).build();
    }
}
```

Olhando para esta classe, podemos inferir o seguinte: 

1. A funcionalidade de cadastrar esta disponivel no endereco: `/api/pessoas`, através do verbo `POST HTTP`.

2. A funcionalidade de cadastrar retorna um header HTTP chamado location com a URI do recurso que foi criado, e a resposta contem um status `HTTP Created`.

3. Quando um recurso é criado, é inserido um registro no banco de dados com as informações do mesmo.

4. Caso aconteça erro de validações, teremos mensagens personalizadas, e por padrão um status `HTTP Bad Request`.


Com as informações apresentadas acima podemos decidir que um teste de integração é o cenario correto a se aplicar, e para que a gente consiga fazer acesso a camada Web precisaremos de um Cliente HTTP e que o `Application Context` esteja de pé para fazer uso do servidor.

Para que o `Application Context` esteja de pé e disponibilize os recursos de nossa API podemos utilizar o Spring Test Context, e como é de se esperar ele também já oferece um client HTTP propio para testes de integração considerando toda complexidade do ecossistema Spring. O MockMvc oferece de forma simples estrutura para testar `beans` referentes aos seguintes esteriotipos: `@Controller, @RestController, @RestControllerAdvice, @ControllerAdvice"`. Também ofererce facilidade para testar com  Filtros do Spring Habilitados e desabilitados.


## Utilizando o MockMvc

Para que construimos testes de integração utilizando o MockMvc devemos configura-lo, e isto é feito anotando com **`@AutoConfigureMockMvc`** a nivel de classe. Veja um Exemplo abaixo:


```java
@SpringBootTest
@AutoConfigureMockMvc
class CadastrarPessoaControllerTest{
    @Autowired
    private MockMvc mockMvc;
}
```


## Criando Teste de Integracao com MockMvc 

Para melhor compreensão de como criar testes com MockMvc iremos conhecer dois componentes que iram nos ajudar no momento de elaborar a requisição **`(request)`** **`HTTP`** e no momento de fazer as verificações da resposta **`(response)`** **`HTTP`**. 


## Conhecendo o MockMvcRequestBuilders e MockMvcResultMatchers

Para criar requisições devemos utilizar o `MockMvcRequestBuilders` para definir o metodo HTTP que iremos consumir, e informar o caminho, e também definir headers, inserir o corpo `(body)` da `requisição HTTP`. 

Já o `MockMvcResultMatchers` é reponsavel por auxiliar no momento da verificação dos elementos da `resposta HTTP`, como por exemplo, verificar se determinado cabeçalho foi enviado, se o status code é condizente aos casos de uso descritos no `controllador`, ou se o corpo `(body)` é de fato conforme o esperado.

Para melhor compreensão destas ferramentas iremos testar o serviço de cadastro de pessoas que foi demostrado acima. Além do MockMvc vamos precisar solicitar ao Spring uma instância do ObjectMapper para poder serializar e desserializar informações. E também do `Repository` ou `EntityManager` para validar se houve cadastro no bando de dados.

Como desejamos fazer um teste que revele a maior quantidade de Bugs possível, temos como obrigação oferecer um cenário mais proximo do real que conseguimos, isto implica que devemos utilizar uma instância do mesmo SGBD, e deixar que os registros sejam escritos no banco de dados, para forçar que possiveis trigers ou violações de constraints sejam disparadas.

Dado a isso iremos utilizar um ambiente para Testes definidos por via de configuração de propriedade. Para atigir este objetivo iremos criar um arquivo de configuração chamado de **`application-test.properties`**, onde redirecionaremos o Spring a se connectar a um schema de testes. Veja abaixo um o arquivo de configuração.


```java
#BD
spring.datasource.driverClassName=org.postgresql.Driver
spring.datasource.url=${DB_URL:jdbc:postgresql://localhost:5432/nutricionista?currentSchema=pessoas_test}
spring.datasource.username=${DB_USERNAME:postgres}
spring.datasource.password=${DB_PASSWORD:password}

#JPA
spring.jpa.hibernate.ddl-auto=${DDL_MODE:update}
spring.jpa.show-sql=${SHOW_SQL:true}
spring.jpa.properties.hibernate.format_sql=${FORMAT_SQL:true}
spring.jackson.serialization.indent_output=true
```

Agora que estamos com o ambiente configurado, basta anotar a classe de test com `@ActiveProfiles("test")`, e o Spring cuidará de executar os testes no banco de dados preparado para testes.

Para fazer bons testes, devemos garantir que estes sejam indepententes, ou seja, que o resultado de um não interfirá no outro. Uma forma de garantir esta independencia é utilizar uma estratégia de limpeza de ambiente,  onde antes de executar cada teste faremos a limpeza do banco aplicando a operação de deleteAll().

O proximo passo é estipular três etapas para nosso testes, que são: 

1. Cenario: Nesta etapa preparamos todos os componentes para que o cenário de testes seja executado.
2. Ação: Nesta etapa é onde disparamos a ação que iremos validar.
3. Corretude: É nesta etapa que verificamos se comportamento da ação, retorna o que era esperado. 

Veja como elaborar um Test completo para o controller: **`CadastrarPessoaController`**


```java
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class CadastrarPessoaControllerTest{
    @Autowired
    private MockMvc mockMvc;
    @Autowired
    private ObjectMapper mapper;
    @Autowired
    private PessoaRepository repository;


    @BeforeEach
    void setUp() {
        repository.deleteAll();
    }


    @Test
    void deveCadastrarUmaPessoaComDadosValidos() throws Exception{
       //cenario
       PessoaRequest pessoaRequest = new PessoaRequest("Yuri Matheus","382.269.280-87");
       
       String payload = mapper.writeValueAsString(livroRequest); //transforma em Json
       
        MockHttpServletRequestBuilder request = MockMvcRequestBuilders.post("/api/v1/pessoas")
            .contentType(MediaType.APPLICATION_JSON)
            .content(payload)
            .header("Accept-Language", "pt-br");//criando a requisicao
        
        //acao
        mockMvc.perform(request)
            .andExpect(
                MockMvcResultMatchers.status().isCreated() //corretude
            )
            .andExpect(
                MockMvcResultMatchers.redirectedUrlPattern("http://localhost/api/v1/pessoas/*")//corretude
            )
        
        //corretude
        List<Pessoa> pessoas = repository.findAll();
        assertEquals(1,pessoas.size());
    }
}
```

Para melhor entidimento do codigo, observamos que  primeiro criamos o cenario,  na seguinte ordem:

1. Definimos o payload da requisição através do objeto PessoaRequest.
2. Logo após solicitamos ao Jackson que transforme o objeto pessoaRequest em Json
3. Definimos a requisição HTTP, onde a chamada sera endereçada ao endereço: **`/api/v1/pessoas`** , com verbo `POST`, nossa requisição produz a mensagem do corpo no formato JSON, e também inserimos no corpo da requisição o Json do payload.

Já na parte da ação, nos disparamos a requisição.

Por fim testamos a resposta do controller, e o seu efeito colateral que é a persistencia no banco, fazemos na seguinte ordem:

1. verificamos se o Status da Resposta HTTP é `201 Created`
2. verificamos se o header location respeita o padrão de URI do recurso
3. verficimos se o cadastro foi feito, dado que o banco esta limpo deve existir apenas este registro.

O proximo passo é testar se informamos dados invalidos é necessário que não aconteça o cadastro.

```java
    @Test
    void naoDeveCadastrarUmaPessoaComDadosInvalidos() throws Exception{
       //cenario
       PessoaRequest pessoaRequest = new PessoaRequest(null,"");
       
       String payload = mapper.writeValueAsString(livroRequest); //transforma em Json
       
        MockHttpServletRequestBuilder request = MockMvcRequestBuilders.post("/api/v1/pessoas")
            .contentType(MediaType.APPLICATION_JSON)
            .content(payload)
            .header("Accept-Language", "pt-br");//criando a requisicao
        
        //acao
      String payloadResponse =  mockMvc.perform(request)
            .andExpect(
                MockMvcResultMatchers.status().isBadRequest() //corretude
            )
            .andReturn()
            .getResponse()
            .getContentAsString(UTF_8);
        
        //corretude
        MensagemDeErro mensagemDeErro = mapper.readValue(payloadResponse, MensagemDeErro.class);

        assertEquals(2, mensagemDeErro.getMensagens().size());
        assertThat(mensagemDeErro.getMensagens(), containsInAnyOrder(
                "O campo nome não deve estar em branco",
                "O campo cpf não deve estar em branco"
        ));
    }
```
Observamos que agora no momento da corretude, usamos também o **`assertThat`** do Hamcrest, para validar se as mensagens de erro condizem com as mensagens da anotação `@NotBlank` da Bean Validation.

Para finalizar precisamos validar se caso apenas o cpf esteja fora, se o cadastro não sera realizado, usando a logica similar a desenvolvida acima.

```java
    @Test
    void naoDeveCadastrarUmaPessoaComCpfInvalido() throws Exception{
       //cenario
       PessoaRequest pessoaRequest = new PessoaRequest(null,"");
       
       String payload = mapper.writeValueAsString(livroRequest); //transforma em Json
       
        MockHttpServletRequestBuilder request = MockMvcRequestBuilders.post("/api/v1/pessoas")
            .contentType(MediaType.APPLICATION_JSON)
            .content(payload)
            .header("Accept-Language", "pt-br");//criando a requisicao
        
        //acao
      String payloadResponse =  mockMvc.perform(request)
            .andExpect(
                MockMvcResultMatchers.status().isBadRequest() //corretude
            )
            .andReturn()
            .getResponse()
            .getContentAsString(UTF_8);
        
        //corretude
        MensagemDeErro mensagemDeErro = mapper.readValue(payloadResponse, MensagemDeErro.class);

        assertEquals(1, mensagemDeErro.getMensagens().size());
        assertThat(mensagemDeErro.getMensagens(), containsInAnyOrder(
                "O campo cpf deve estar no formato valido",
                
        ));
    }
```


## links para aprofundamento

- [Configurando o mockMvc](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.spring-mvc-tests)
- [MockMvc Java doc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html)