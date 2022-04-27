# Conhecendo o Spring Test Context e extraindo o maximo para fazer seus testes

Não tem como discutir que uma das grandes vantagens de construir software utilizando Spring Framework é poder se apropiar do container de Inversão de Controle (IoC) e Injeção de Dependências (DI). O container de **`IoC`** é responsavel por gerenciar o ciclo de vida das classes que são denominadas **`beans`**, isso quer dizer que o Spring define quando uma classe é instanciada e também quando ela é inserida nas outras classes que dependente dela. 

Um bom exemplo para definição de um `bean` é quando você cria um controller, ou um service, ou até mesmo um Job que executa a cada 1 minuto, e estas classes recebem anotações como: `@Controller, @RestController, @Service, @Repository ou @Component`.  A partir do momento que estas classes são configuradas como `beans` é possivel observar que não nos preocupamos em instancia-las, nossa unica preocupação é dizer em quais `beans` dependemos delas e o Spring  cuidará de disponibilizar para gente.

Quando um aplicação Spring é levantada, uma das primeiras tarefas que ela executa é a instancia de todos os `beans` e configuração de seus componentes, por exemplo efetuando a conexão com banco de dados, ou até mesmo com uma fila com seu broker preferido como Kafka ou rabbitMQ. Estas configurações formam o que chamamos de `ApplicationContext` ou Contexto de Aplicação em portugues.

O `ApplicationContext` nos permite utilizar de maneira eficiente seus `beans` para realizar tarefas que foram programadas no software, como por exemplo definir um comportamento para um verbo `HTTP`, ou realizar persistencia de dados com um `Repository`.

Quando construimos software de maneira responsavel e segura, optamos por elaborar uma boa bateria de testes, que se preocupam em verificar a corretude do que foi implementado e também revelar bugs antes que o software seja entegue ao ambiente de produção. Isto significa que é dever do desenvolvedor verificar se cada unidade trabalha como o esperado e que a junção das unidades quando integradas trabalham como o que foi determinado. Um bom exemplo para testes que visão avaliar se uma unidade trabalha conforme o esperado quando é integrada a outra, é quando testamos os nossos `controllers`.

Para criar bons testes em um **`controller`**, devemos verficicar alguns detalhes, como: 

1. O verbo `HTTP` que foi mapeado esta atendendo no caminho definido.
2. O efeito colateral deste **`controller`** , por exemplo, deve cadastrar no banco de dados.
3. O corpo da respónse é o que é esperado.

Olhando para estes pontos, somos capazes de perceber a importancia de que o `ApplicationContext` esteja disponivel, porquê é através deste que as funcionalidades do **`controller`**  se encontrem disponivel na Web, também é  através deste que temos acesso e uso simplificado do banco de dados através do `Repository`. 

É importante ressaltar que caso o `ApplicationContext` não esteja disponivel os testes serão regados de `Mocks` que são objetos que simuluam o comportamento de determinado objeto, ou seja, estaremos testando cada vez mais distante do funcionamento real do sistema, e provavelmente com maior probabilidade de não encontrar falhas.

Sabemos que a melhor estrategia para garantir o bom funcionamento do **`controller`** é testar de maneira integrada, e para que testes de qualidade possam ser criados o Spring disponibiliza `TestContext`, que é uma extensão do `ApplicationContext` que possibilita uso de um gerenciador de transações, injeção de dependencias, configuração para `profiles`, gerenciamento de contexto e configuração personalizada de beans para testes e diversas ferramentas para realizar de `asserts` como:

- JUnit 4
- JUnit 5 Jupiter
- AssertJ
- Hamcrest
- Mockito

Desta forma podems construir `testes de unidade e integração` da melhor maneira, extraindo maximo dos container de IoC do Spring.

Para configurar e disponibilizar o `ApplicationContext` e poder disfrutar da injeção de dependencias, devemos anotar a classe com **`@SpringBootTest`**, a partir deste momento estamos habilitados a realizar injeção de depência via atributo com anotação `@Autowired`, `@Inject` ou `@PersistenceContext` para o **`EntityManager`**. como no exemplo abaixo: 


```java
@SpringBootTest
class CadastrarUmUsuarioControllerTest {
    @Autowired
    private UsuarioRepository repository;
    
    @PersistenceContext
    private EntityManager manager;

}
```


OBS: A boa pratica sobre injeção de depêndencias em classes de codigo de produção é realizar a injeção via construtor, pois existem situações onde você poderá instanciar manualmente sua classe, e informas as dependencias via construtor publico. Jà quando falamos de classes de testes você nunca poderá instancia-las então obter um construtor ou metodo setter é desnecessário, e torna-se a boa pratica realizar a injeção via atributo através de anotações.


# Entendendo o funcionamento do Spring Test Context

A partir do momento que o TestContext disponibiliza o `ApplicationContext` para um testes, este contexto é armazenado em cahce e reutilizado para todos os testes subsequentes que fazer parte da mesma configuração. Isso quer dizer que se você utilizou apenas @SpringBootTest em todas suas classes sem personalizar alguma configuração, o contexto sera inicializado uma vez e todos os seus testes serão executados com o mesmo contexto.

Para maioria dos casos, um mesmo contexto pode abranger todos os testes, mas se for necessário substituir um bean ou construir um novo contexto para uma classe ou metodo podemos utilizar a anotação **[@DirtiesContext](https://docs.spring.io/spring-framework/docs/5.3.19/reference/html/testing.html#spring-testing-annotation-dirtiescontext)** indicando para o Spring que este contexto esta inadequado, para que seja limpo do cache e outro seja construido.

Veja um exemplo de quando todos os metodos de uma classe são executados em um novo contexto.


```java
@SpringBootTest
@DirtiesContext
class CadastrarUmUsuarioControllerTest {
    @Autowired
    private UsuarioRepository repository;


    @Test
    void deveCadastrarUmUsuario(){
        //logica para testar o cadastro
    }
    
     @Test
    void naoDeveCadastrarUmUsuarioComDadosInvalidos(){
        //logica para testar se o cadastro nao sera feito com dados invalidos
    }
}
```

Caso desejamos que apartir de um determinado metodo o contexto seja reconstruido podemos anotar a nivel de metodo, como no codigo abaixo.


```java
@SpringBootTest
class CadastrarUmUsuarioControllerTest {
    @Autowired
    private UsuarioRepository repository;


    @Test
    void deveCadastrarUmUsuario(){
        //logica para testar o cadastro
    }
    
    @Test
    void naoDeveCadastrarUmUsuarioComDadosInvalidos(){
        //logica para testar se o cadastro nao sera feito com dados invalidos
    }

    @Test
    @DirtiesContext
    void naoDeveCadastrarUmUsuarioComEmailInvalidos(){
        //logica para testar se o cadastro nao sera feito com email invalido
    }
}
```


## Ativando beans de perfils, ou configurações de perfil

É muito comum que um software possua diversos ambientes, onde as configurações são distintas. Por exemplo no ambiente de desenvolvimento você pode usar uma instância de banco diferente que a de produção por questões de infraestrutura. Ou determinar o comportamento de um modulo de segurança, nestes casos, podemos configurar o comportamento para cada perfil definindo as propriedades destes ambientes em arquivos de configuração.

A boa pratica é que seus testes executem em um ambiente onde não gere impacto no negocio, então ter uma instancia do mesmo SGBD separada para testes é uma otima pratica. A configuração do ambiente pode ser feita através de arquivos de configuração `yaml ou properties`. Para definir um arquivo de configuração com formato properties que atenda um perfil, podemos atribuir um hífen e o nome do perfil, como por exemplo:  **`application-test.properties`**

### Ativiando um perfil em uma classe de teste

Lembrando que podemos ter diversos perfils e ativar um para cada clase, através da anotação **`@ActiveProfiles`**, que recebe por argumento o nome do perfil que  será utilizada no momento de carregamento do `ApplicationContext`.

Para ativar o perfil que definimos acima, observe o seguinte codigo.


```java
@SpringBootTest
@ActiveProfiles("tests") 
class CadastrarUmUsuarioControllerTest {
    @Autowired
    private UsuarioRepository repository;


    @Test
    void deveCadastrarUmUsuario(){
        //logica para testar o cadastro
    }
}
```

Nos do time da Zup Edu recomendamos a leitura dos links de referência para compreensão mais profunda sobre Spring Test Context. O material apresentado neste artigo é sufuciente para quase todos casos de usos, porém, para alguns casos é necessário que o conhecimento de personalização e configuração de contexto seja amplificado.



## Referências

- [Para conhecer o Spring Test](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing)
- [Personalizando um Test Context](https://docs.spring.io/spring-framework/docs/5.3.19/reference/html/testing.html#testcontext-framework)