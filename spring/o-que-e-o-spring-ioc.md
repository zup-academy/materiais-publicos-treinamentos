# Entendendo o que é o Spring IOC

O Spring IOC é o core do Spring.
Antes do Spring pensar em como implementar a Inversão de Controle para aplicações com Java, era muito diferente como era feito o gerenciamento de intâncias.
Essa forma com a qual o Spring resolveu trabalhar também é conhecida como Injeção de dependência.

Resumidamente é o processo onde um objeto depende de outros objetos, e neste caso estamos falando do objeto quanto uma instância na memória da JVM.

O Spring construiu um container que gerencia e cria essas instâncias além de injetálas conforme suas dependências.

A Inversão de controle se dá como o processo de injeção acontece , diferentemente de um objeto que acaba recebendo suas dependências pelo seu construtor, ele não recebe a dependência dessa forma.

Os pacotes org.springframework.beans e org.springframework.context são principais em relação a parte que faz esse processo.

O Spring utiliza de Programação Orientada a Aspecto mais conhecido como AOP para fazer as ações do IOC.

## Mas o que é um Bean?

Um Bean é todo objeto que é gerenciado pelo Spring, resumidamente que o Spring que instancia o objeto e suas dependências.

### Escopos de um Bean - @Scope

- singleton : todo bean é por pdrão Singleton, ma única instância de objeto para cada contêiner Spring IoC
- prototype : toda vez que o container for solicitado ele criará uma nova instância do Bean
- request : para o ciclo de vida de uma única solicitação HTTP. Ou seja, cada solicitação HTTP tem sua própria instância de um bean criado a partir de uma única definição de bean, podemos utiliza a anotação @RequestScope
- session : para o ciclo de vida de um HTTP Session, podemos utilizar @SessionScope
- application : para o ciclo de vida de um arquivo ServletContext
- websocket : para o ciclo de vida de um arquivo WebSocket

### Definição de um Bean

* class
* name - todo bean deve ter nome exclusivo, não é possível ter mais de um bean com o mesmo nome
* scope - o ciclo de vida e criação do bean
* constructor arguments
* properties
* autowiring mode
* lazy-initialization mode
* initialization method
  * Nesse Caso não é recomendado utilizar a interface InitializingBean, pois ela acopla o código ao Spring, o que o próprio Spring recomenda é o uso da anotação do Java EE @PostConstruct
Exemplo 1:
```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```
Exemplo 2:
```java
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```
Exemplo 3:
```java
@Component
public class DbInit {

    @Autowired
    private UserRepository userRepository;

    @PostConstruct
    private void postConstruct() {
        User admin = new User("admin", "admin password");
        User normalUser = new User("user", "user password");
        userRepository.save(admin, normalUser);
    }
}
```

* Destruction method 
  * Assim como initialization method, não é recomendado utilizar a interface DisposableBean e sim a anotação do Java EE @PreDestroy.
Exemplo 1:
```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```
Exemplo 2:
```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```
Exemplo 3:
```java
@Component
public class UserRepository {

    private DbConnection dbConnection;
    @PreDestroy
    public void preDestroy() {
        dbConnection.close();
    }
}
```

### Ciclo de vida de um bean

Envolve o momento que o bean é criado e o momento que ele é destruído.


### Annotations

- @Bean - configurar uma nova instância para o Spring gerenciar
- @Configuration - Classes com essa anotação permitem criação e configuração de beans
- @Component - sua implementação é de nível de classe , quando esta sendo feita a varredura de classes o Spring considera classes anotadas com esse tipo de anotação.
- @Controller - Essa anotação prevê tratamento especifico para criação de camada de controlador para estrutura mvc de aplicações.
- @RestController - é uma especialização de @Controller porém com tratativas especificas para API REST
- @Service - Essa anotação é utilizada em camadas onde há lódica de negócio.
- @Repository - Todas classes com acesso a banco de dados precisam ser anotadas com @Repository, segue a mesma regra de @Component porém prevê utilização de transações com banco de dados e tratativa de exceções de persistência de dados.
- @ComponentScan - usando o parametro da anotaao basePackages configuramos quais pacotes procurar por classes com configuração de anotação, ou podemos utilizar o basePackageClasses ele irá olhar os demais pacotes em que essa classe esta.
- @Qualifier - Permite configurarmos um bean destinando um nome especifico que queremos injetar 
- @Lazy - Utilizado quando queremos que a instancia seja criada somente quando o objeto será utilizado.
-  Java EE - @PostConstruct - O Spring entende que essa anotação que é aplicada a nível de método é acionada após a criação do bean
-  Java EE - @PreDestroy - Ele é semelhante ao @PostConstruct porém é executado um pouco antes de o Spring remover o objeto do contexto.


#### Link de referência 

https://docs.spring.io/spring-framework/docs/current/reference/html/core.html

https://www.baeldung.com/spring-bean-scopes

https://www.baeldung.com/spring-bean-annotations