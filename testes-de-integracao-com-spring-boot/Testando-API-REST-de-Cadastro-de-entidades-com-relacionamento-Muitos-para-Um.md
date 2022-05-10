# Construindo Testes de Integração para API REST de Cadastro de entidades com relacionamento Muitos para Um

Criar testes de integração para API REST  é uma forma de garantir a corretude do comportamento de cada endpoint, e também auxilia a encontrar anomalias e bugs no sistema.

O Spring Test oferece suporte ao uso de toda complexidade do `ApplicationContext` do Spring para criação de testes, inclusive varias ferramentas para criação de **`asserts`**  e um cliente HTTP adaptado para lidar com as nuances de implementação do Spring.

Sabemos que ao elaborar um testes de integração é nossa responsabilidade identificar quais são os pontos de integração do serviço que estamos testando. Um bom exemplo é verificar quais são as classes que se conectao afim de resolver um problema, e quais são os sitemas que se integram como Banco de Dados (BD), ou qualquer API externa.

Quando testamos endpoint para cadastro simples, nosso objetivo é verificar se o servidor atende as requisições para determinado verbo, se os contratos são respeitados, e caso algum efeito colateral seja disparado, devemos verificar se de fato foi executado, como por exemplo um cadastro no banco de dados.

Para testar cadastros mais elaborados e que as entidades contam relacionamentos muitos para um, Um para Muitos ou Muitos para Muitos, não tem segredo iremos nos basear nestes conceitos e garantir a cobertura dos fluxos da nossas APIS.

## Testando um Serviço de Cadastro de Entidade com Relacionamento Muitos para Um

Para criar bons testes é essecial entender o dominio e as relações, observe abaixo  as entidades.

```java
@Entity
public class Pessoa{
    @Id
    @GeneratedValue
    private Long id;
    @Column(nullable = false)
    private String nome;
    @Column(nullable = false)
    private String cpf;

    public Pessoa(String nome, String cpf){
        this.nome=nome;
        this.cpf=cpf;
    }

    @Deprecated
    public Pessoa(){

    }

    public Long getId(){
        return id;
    }
}
@Entity
public class Endereco{
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(nullable = false)
    private String endereco;
    
    @Column(nullable = false)
    private String cep;

    @ManyToOne
    private Pessoa pessoa;


    public Endereco(String endereco, String cep, Pessoa pessoa){
        this.endereco=endereco;
        this.cep=cep;
        this.pessoa=pessoa;
    }

    @Deprecated
    public Endereco(){

    }

    public Long getId(){
        return id;
    }
}
```

Por fim iremos compreender o nosso endpoint, e o  DTO responsavel por receber os dados de entrada.

```java
public class EnderecoRequest{
    @NotBlank
    private String endereco;
    @NotBlank
    private String cep;

    public EnderecoRequest(String endereco, String cep){
        this.endereco=endereco;
        this.cep=cep;
    }

    public EnderecoRequest(){

    }

    public Endereco paraEndereco(Pessoa pessoa){
        return new Endereco(endereco,cep,pessoa);
    }

    //getters omitidos aqui

}

@RestController
public class CadastrarEnderecoAPessoaController {
    private final PessoaRepository pessoaRepository;
    private final EnderecoRepository enderecoRepository;

    public CadastrarEnderecoAPessoaController( PessoaRepository pessoaRepository, EnderecoRepository enderecoRepository) {
        this.pessoaRepository = pessoaRepository;
        this.enderecoRepository = enderecoRepository;
    }


    @PostMapping("/pessoas/{id}/enderecos")
    @Transactional
    public ResponseEntity<?> cadastrar(@RequestBody @Valid EnderecoRequest request, @PathVariable Long id, UriComponentsBuilder uriComponentsBuilder){


        Pessoa pessoa = pessoaRepository.findById(id)
                .orElseThrow(()-> new ResponseStatusException(HttpStatus.NOT_FOUND,"Pessoa não cadastrado no sistema"));

        Endereco endereco = request.paraEndereco(pessoa);

        enderecoRepository.save(endereco);

        URI location = uriComponentsBuilder.path("/pessoas/{id}/enderecos/{id}")
                .buildAndExpand(pessoa.getId(),endereco.getId())
                .toUri();

        return created(location).build();
    }
}

```

Quando observamos este **`Controller`** temos ideia de 3 fluxos, dois estão explicitamente descritos no codigo. O primeiro é quando não existe cadastro de pessoa para o id informado no **`path`**. Neste caso é quando identificamos que existe um **`IF`** implicito, e quando temos mudança de fluxo, devemos validar cada um dos caminhos.

O Segundo fluxo explicito é o caso de sucesso, quando o cadastro é executado sem problemas, e é retornado a URI do recuros na resposta.

O terceiro fluxo é o que esta demostrado de maneira implicita,  se olhamos para os argumentos do metodo **cadastrar**, encontramos a anotação **`@Valid`**, que dispara as validações da Bean Validation no EnderecoRequest, e o Exception Handler acumula os erros e entrega no corpo da Resposta `HTTP com Status Bad Request`.


### Extraindo cenarios dos fluxos

Olhando para os fluxos descritos acimas iremos esboçar nossos cenarios de teste, onde para cada cenario sera criado um teste.

Começando pelo fluxo de validação, somos capazes de identificar que as validações no EndereçoRequest garatem apenas que as informações são obrigatorias, então para o Status 400, temos o cenario:  **`não deve cadastar um endereço para dados invalidos`**. 

Observe o codigo condizente ao cenario e sua explicação:

```java
    @Test
    @DisplayName("nao deve cadastrar um endereco com dados invalidos")
    void test3() throws Exception {
        //1
        this.pessoa = new Pessoa("Jordi H M Silva", "12345678929");
        pessoaRepository.save(this.pessoa);

        //2
        EnderecoRequest enderecoRequest = new EnderecoRequest(null, null);
        String payload = mapper.writeValueAsString(enderecoRequest);

        //3
        MockHttpServletRequestBuilder request = post("/pessoas/{id}/enderecos", aluno.getId())
                .contentType(MediaType.APPLICATION_JSON)
                .header("Accept-Language","pt-br")
                .content(payload);

        //4
        String payloadResponse = mockMvc.perform(request)
                .andExpect(
                        status().isBadRequest()
                )
                .andReturn()
                .getResponse()
                .getContentAsString(UTF_8);

        //5
        MensagemDeErro mensagemDeErro = mapper.readValue(payloadResponse, MensagemDeErro.class);


        //6
        assertEquals(2, mensagemDeErro.getMensagens().size());
        assertThat(mensagemDeErro.getMensagens(), containsInAnyOrder(
                "O campo endereco não deve estar em branco",
                "O campo cep não deve estar em branco"
                )
        );

    }
```
 Para garantir que o comportamento do fluxo de dados invalidos é executado, fazemos os seguintes passos:

 1. É cadastrado uma pessoa que ira receber o endereço
 2. É criado um Json referente ao EnderecoRequest com dados invalidos
 3. É construido uma requisição para o verbo POST no endereco "/pessoas/{id}/enderecos", e no corpo é enviado o Json criado anteriormente.
 4. É executada a requisição, e validado se o status é BadRequest, o corpo da response é atribuido a uma String com nome de PayloadResponse.
 5. o Json de response é desserializado no Objeto que agrupa as mensagens de erro
 6. É verificado se existem duas mensagens de erro, e se as mensagens informam que os campos endereco e cep não devem estar em branco.

 O proximo cenario visa garantir que uma requisição disparada para um id de Pessoa não existente recebera o Status de NotFoud. Este cenario sera chamado de:  **`Não deve cadastrar um endereco caso a Pessoa não exista`**.
 
Veja o codigo abaixo: 


 ```java
    @Test
    @DisplayName("nao deve cadastrar um endereco caso a pessoa nao exista")
    void test3() throws Exception {
        EnderecoRequest enderecoRequest = new EnderecoRequest(null, null);
        String payload = mapper.writeValueAsString(enderecoRequest);

      
        MockHttpServletRequestBuilder request = post("/pessoas/{id}/enderecos", 1000000)
                .contentType(MediaType.APPLICATION_JSON)
                .header("Accept-Language","pt-br")
                .content(payload);

        
        String payloadResponse = mockMvc.perform(request)
                .andExpect(
                        status().isNotFound()
                );
    }
```

E para finalizar iremos testar o cenario de sucesso, que é onde recebemos junto a resposta um status 201, um header chamado location seguindo um padrão e como efeito colateral é escrito um registro no banco com as informações do endereço. Para este caso vamos descrever o cenario como: **`deve cadastrar um endereco com dados validos`**.



 ```java
    @Test
    @DisplayName("deve cadastrar um endereco com dados validos")
    void test3() throws Exception {
        /
        this.pessoa = new Pessoa("Jordi H M Silva", "12345678929");
        pessoaRepository.save(this.pessoa);

        
        EnderecoRequest enderecoRequest = new EnderecoRequest(null, null);
        String payload = mapper.writeValueAsString(enderecoRequest);

      
        MockHttpServletRequestBuilder request = post("/pessoas/{id}/enderecos", 1000000)
                .contentType(MediaType.APPLICATION_JSON)
                .header("Accept-Language","pt-br")
                .content(payload);

        
        String payloadResponse = mockMvc.perform(request)
                .andExpect(
                        status().isCreated()
                )
                .andExpect(
                        redirectedUrlPattern("http://localhost/pessoas/*/enderecos/*")
                );
        
        List<Endereco> enderecos = enderecoRepository.findAll();

        assertEquals(1, enderecos.size());
    }
```
    
Para finalizar devemos manter as boas praticas, e antes da execução de cada metodo limpar o banco de dados, desta forma garantimos que os testes são indepentendes.

```java
@BeforeEach
void setUp() {
    enderecoRepository.deleteAll();
    pessoaRpository.deleteAll();
}
```

E também configurar o ambiente para testes, onde exista um Schema no BD exclusivo para execução dos testes, então anotaremos nossa classe de testes com `@ActiveProfiles("test")`.

```java
@SpringBooTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class CadastrarEnderecoAPessoaControllerTest {
   //dependencias e metodos omitidos
}
```
