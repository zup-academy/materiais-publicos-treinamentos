# O que são Objetos de Transferencia de Dados e como devemos utiliza-los?


Quando estamos falando de Programação Orientada a Objetos é normal que uma classe seja utilizada em varias funcionalidades ou pontos do codigo, e que para cada funcionalidade apenas algumas informações sejam relevantes. Quando falamos de WebSevices é muito comum que recebemos informações de um determinado recurso através de uma requisição HTTP, realizamos algum processamento e persistimos ao banco de dados. 

Um software moderno hoje é bem organizado, e normalmente esta organização é feita por responsabilidades, um dos padrões que foi e continua sendo bem utilizado é o Model View Controller (MVC). Sua premissa é as classes sejam separadas em pacotes (packages) pelas seguintes responsablidades:

1. Modelo (Model): Dentro deste pacote ficam as classes que fazem referência ao modelo de dominio do problema que esta sendo resolvido.

2. Controller (Controlador): Dentro deste pacote ficam as classes que são responsaveis por controlar o fluxo das informações, fazendo a conexão entre a view e a model.

3. Visualização (View): Aqui ficam as classes que são responsaveis por gerar a visualização dos dados.

Para entender melhor, imagine um fluxo de transferência da dados no sistema, onde é feito uma chamada HTTP para um endereço, ou seja, uma pagina de visualização onde existe um formulario que o cliente ira inserir as informações, que são enviadas ao controller, onde é aplicado uma serie de validações e caso as informações estejam de acordo com esperado, o controller envia para alguma classe que fará a persistencia desse model no banco de dados.

Observando este cenario podemos notar que as mesmas informações que serão recebidas da view serão persistidas no banco de dados, caso atendam as restrições de validação. Ao trabalhar com visualizações é comum utilizar a entidade com ponto de entrada de dados. Porém, no contexto de REST APIs esta decisão pode vunerabilizar o sistema, já que o cliente poderá informar dados invalidos, e até mesmo tentar atribuir valores a propriedades não relacionadas a feateure de negocio. Caso o sistema não possua um Error Handler bem implementando informações como nome e versão do framework podem sejam retornadas, facilitando que usuarios mal intecionado elaborarem ataques ao sistema.

Uma forma simples de evitar esta possivel falha de segurança é criar um outro objeto que referêncie a entidade na funcionalidade de negocio, este objeto irá agrupar apenas so dados, que são utéis para a função, e terá como responsabilidade transferir estas informações entre as camadas do sistema. Esta solução é conhecido com um Designer Pattern Data Transfer Object (DTO) ou Objeto de Transferencia de Dados. 

Agora que esta contextualizado, iremos aplicar a utilização de um DTO para recebimento de informação através de uma API REST.

## Utilizando DTO em uma API REST

No texto abaixo sera apresentada uma API REST que recebe a propria entidade em seu controlador, e iremos aplicar o Padrão de Projeto DTO para segregar a responsabilidade de recebimento de dados.


``` java
@Entity
public class Pessoa {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String nome;
    private String cpf;
    private String email;
    private String rg;
    private LocalDate dataNascimento;

    //construtores e demais metodos omitidos aqui
}
```

``` java
@RestController
public class CadastrarNovaPessoaController {
    private final PessoaRepository repository;

    public CadastrarNovaPessoaController(PessoaRepository repository) {
        this.repository = repository;
    }

    @PostMapping("/pessoas")
    public void cadastrar(@RequestBody  Pessoa pessoa){
        repository.save(pessoa);
    }
}
```
### Utilizando um DTO para Receber informações

Criaremos uma classe responsavel por receber as informações de uma pessoa atráves da requisição e adicionaremos um metodo com a responsabilidade de instanciar um objeto do tipo Pessoa com as informações que foram recebidas.


``` java
public class PessoaRequest {
    private String nome;
    private String cpf;
    private String email;
    private String rg;
    private LocalDate dataNascimento;

   public Pessoa paraPessoa(){
       return new Pessoa(nome,cpf,email,rg,dataNascimento);
   }

   //construtores e demais metodos omitidos aqui

}
```

Agora iremos rescrever nosso controlador para utilizar esta classe como ponto de entrada de dados.

``` java
@RestController
public class CadastrarNovaPessoaController {
    private final PessoaRepository repository;

    public CadastrarNovaPessoaController(PessoaRepository repository) {
        this.repository = repository;
    }

    @PostMapping("/pessoas")
    public void cadastrar(@RequestBody  PessoaRequest request){

        Pessoa pessoa= request.paraPessoa();
        
        repository.save(pessoa);
    }
}
```
