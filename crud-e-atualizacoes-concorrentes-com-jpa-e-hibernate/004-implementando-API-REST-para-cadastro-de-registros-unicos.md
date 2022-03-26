# Implementando API REST para cadastro de registros únicos

Nesse conteúdo veremos como podemos implementar uma API REST responsável por cadastrar um registro único no sistema. Vamos partir da construção da API desde a escrita do controller via módulo Spring Web até a persistência com Spring Data, JPA e Hibernate.

## Cadastrando alunos únicos na universidade

Precisamos implementar um cadastro de alunos em uma universidade. Esse cadastro ocorre num sistema interno da universidade, portanto precisamos criar uma API REST para expor essa funcionalidade. Um ponto importante, o sistema **não** deve permitir o cadastro de alunos duplicados, e, para isso, deve-se usar o email do aluno como chave de negócio para garantir a unicidade no sistema.

Para cadastrar um novo aluno no sistema de uma universidade precisamos dos dados básicos deste aluno, como nome, email e ID do curso. Portanto, teríamos a seguinte representação de aluno em uma classe `Aluno` mapeado com anotações da JPA:

```java
@Entity
class Aluno {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;
    private String email;
    private Long cursoId;

    // outros atributos, construtores, getters e setters
}
```

E claro, como vamos gravar esta entidade no banco de dados nós precisamos de uma classe Repository do Spring Data. O código da interface `AlunoRepository` seria bem simples graças ao Spring Data JPA:

```java
@Repository
public class AlunoRepository extends JPARepository<Aluno, Long> {

}
```

## Implementando um controller para cadastro de alunos

Agora, precisamos implementar uma API REST para efetuar o cadastro de alunos. Para isso vamos criar um controller `NovoAlunoController` com um método que recebe uma requisição do tipo `POST` na URI `/api/alunos`. Além disso, os dados do payload JSON deserializados na nossa classe de DTO `NovoAlunoRequest` e em seguida validados através das anotações da Bean Validation:

```java
@RestController
class NovoAlunoController {

    @Autowired
    private AlunoRepository repository;

    @Transactional
    @PostMapping("/api/alunos")
    public ResponseEntity<?> cadastra(@RequestBody @Valid NovoAlunoRequest request, UriComponentsBuilder  uriBuilder) {

        Aluno aluno = request.toModel();
        repository.save(aluno);

        URI location = uriBuilder.path("/api/alunos/{id}")
                    .buildAndExpand(aluno.getId())
                    .toUri();

        return ResponseEntity
                .created(location).build();
    }
}

class NovoAlunoRequest {

    @NotBlank
    private String nome;

    @Email
    @NotBlank
    private String email;

    @NotNull
    private Long cursoId;

    // construtor e getters
}
```

Pronto! Com essa API REST podemos cadastrar um novo aluno no sistema corretamente, mas ainda falta um ponto: ela não previne o cadastro de alunos duplicados no sistema, ou seja, podemos ter 2 alunos com o mesmo email no sistema.

## Evitando alunos duplicados

Para implementar a lógica de unicidade de alunos, basta adicionar uma lógica de negócio que verifica se determinado aluno existe no banco de dados. Para isso, usaremos o email do aluno como filtro da consulta no banco de dados. Portanto, o código do controller será alterado como abaixo:

```java
@Transactional
@PostMapping("/api/alunos")
public ResponseEntity<?> cadastra(@RequestBody @Valid NovoAlunoRequest request, UriComponentsBuilder  uriBuilder) {

    if (repository.existsByEmail(request.getEmail())) {
        throw new ResponseStatusException(HttpStatus.UNPROCESSABLE_ENTITY, "aluno existente no sistema");
    }

    // restante do código
}
```

Repare que utilizamos o método `existsByEmail` do `AlunoRepository` para verificar a existência de um aluno por seu atributo email, e caso ele já exista no banco de dados nós retornamos um erro com Status HTTP `422 (Unprocessable Entity)`. Além disso, por se tratar de uma consulta especifica para satisfazer a regra de negócio, precisamos adicionar este novo método, também conhecido como **Derived Query**, na classe `AlunoRepository`, :

```java
@Repository
public class AlunoRepository extends JPARepository<Aluno, Long> {

    public boolean existsByEmail(String email);

}
```

Dessa forma, sempre que o usuário tentar cadastrar um aluno pela segunda vez o sistema não permitirá, que é exatamente o que queremos. Mas será que esta lógica de negócio, esse simples `if` no controller, é suficiente?

## Definindo constraints de unicidade no banco de dados

Embora o código acima esteja correto no ponto de vista de negócio, ele é falho no ponto de vista sistêmico. Não podemos esquecer que um sistema web lida com usuários simultâneos, ou seja, usuários concorrentes. Por esse motivo, em ambientes concorrentes, pode acontecer de dois usuários tentarem cadastrar o mesmo aluno no mesmo instante, o que pode levar ao cadastro duplicado do aluno no banco de dados.

Uma possível solução seria adicionar alguma estratégia de locking para garantir que somente um usuário por vez consiga executar a lógica de negócio; no mundo Java, podemos conseguir isto adicionando o modificador `synchronized` no método do controller, dessa forma a JVM se encarregaria de serializar (enfileirar) as requisições dos usuários, o que evitaria o problema de duplicidade:

```java
public synchronized ResponseEntity<?> cadastra(...) {

    // ...
}
```

O problema dessa solução é que ela funcionaria somente em uma única máquina (JVM), mas falharia em um ambiente clusterizado (duas ou mais máquinas), afinal uma JVM não enxergaria o lock (bloqueio) do `synchronized` existente na outra JVM. E, no mundo elástico de cloud, geralmente nós desenvolvedores(as) acabamos não tendo controle sobre o número de máquinas rodando.

Para resolver este problema precisamos que esta validação de unicidade esteja centralizada de tal forma que, independente do número de máquinas, ela seja segura e confiável. Por estarmos trabalhando com um banco de dados relacional e, por ele ser o responsável por manter a integridade e consistência dos dados, nada mais justo do que delegar esta regra de validação para ele, afinal ele já faz isso muito bem há mais de 45 anos.

Para isso, precisamos definir uma restrição de unicidade (ou **UNIQUE constraint**) para coluna `email` na tabela `ALUNO` no banco de dados. Conseguimos isto habilitando o atributo `unique` da anotação `@Column` na entidade:

```java
@Entity
class Aluno {

    @Column(unique = true) // define UNIQUE constraint
    private String email;

    // restante do código
}
```

Deste modo, o Hibernate vai gerar a constraint de unicidade semelhante ao comando DDL abaixo:

```sql
ALTER TABLE aluno
  ADD CONSTRAINT UK_rrtn7wgfxo0jfwkhby23f72cn UNIQUE (email);
```

A partir de agora, quem garante a unicidade dos alunos no sistema é o banco de dados, na qual é nossa borda mais interna do sistema: o banco é nossa **última barreira de proteção dos dados**. O que estou querendo dizer, é que nossa aplicação pode até falhar ou mesmo afrouxar as regras por qualquer motivo, e isso em alguns casos é aceitável, mas não o banco de dados, pois não só ele é a última proteção do nosso sistema como também é a nossa melhor alternativa.

Nesse momento, a pergunta que fica é: quando essa constraint é violada, qual erro é lançado na aplicação?

## Trantando erros de constraints de unicidade: `org.hibernate.exception.ConstraintViolationException`

Adicionar constraints no banco, próximas dos dados, sem dúvida é a melhor estratégia que podemos adotar para maioria gritante dos casos, mas isso também quer dizer que quando o banco detecta alguma violação dessas constraints ele precisa informar a aplicação sobre essa violação, e a forma como ele faz isso é lançando um erro da camada de persistência.

Se não capturarmos e tratarmos esse erro lançado pelo banco de dados ele vai subir todas as camadas da aplicação e estourar na cara do usuário com um Status de erro HTTP `500 (Internal Server Error)`, semelhante a este:

```json
HTTP/1.1 500
Content-Type: application/json
Transfer-Encoding: chunked
Date: Mon, 25 Mar 2022 16:35:05 GMT

{
	"timestamp": "2022-03-25T16:35:05.968+00:00",
	"status": 500,
	"error": "Internal Server Error",
	"message": "could not execute statement; SQL [n/a]; constraint [uk_rrtn7wgfxo0jfwkhby23f72cn]; nested exception is org.hibernate.exception.ConstraintViolationException: could not execute statement",
	"path": "/api/contatos"
}
```

Repare que a exceção lançada foi uma `org.hibernate.exception.ConstraintViolationException` (atenção: não é o pacote da Bean Validation: `javax.validation.*`) com uma mensagem de erro especifica. Essa exceção `ConstraintViolationException` é do Hibernate, ela **encapsula a exceção original lançada pelo banco de dados**. Tanto é, que se olharmos mais de perto os logs de erro no console da aplicação, veremos algo parecido com isso:

```log
org.postgresql.util.PSQLException: ERROR: duplicate key value violates unique constraint "uk_rrtn7wgfxo0jfwkhby23f72cn"
  Detail: Key (nome)=(Yuri Matheus) already exists.
	at org.postgresql.core.v3.QueryExecutorImpl.receiveErrorResponse(QueryExecutorImpl.java:2674) ~[postgresql-42.3.1.jar:42.3.1]
	at org.postgresql.core.v3.QueryExecutorImpl.processResults(QueryExecutorImpl.java:2364) ~[postgresql-42.3.1.jar:42.3.1]
	at org.postgresql.core.v3.QueryExecutorImpl.execute(QueryExecutorImpl.java:354) ~[postgresql-42.3.1.jar:42.3.1]
	at org.postgresql.jdbc.PgStatement.executeInternal(PgStatement.java:484) ~[postgresql-42.3.1.jar:42.3.1]
	at org.postgresql.jdbc.PgStatement.execute(PgStatement.java:404) ~[postgresql-42.3.1.jar:42.3.1]
    ...
```

Como estamos usando o PostgreSQL como banco de dados, a exceção lançada pelo driver JDBC foi `org.postgresql.util.PSQLException`. É uma exceção muito especifica, por esse motivo o Hibernate a encapsula em uma exceção mais genérica para que facilite a captura e tratamento de erros. Dessa forma, se quisermos captura-la devemos alterar nosso controller como abaixo:

```java
@Transactional
@PostMapping("/api/alunos")
public ResponseEntity<?> cadastra(...) {

    if (repository.existsByEmail(request.getEmail())) {
        throw new ResponseStatusException(HttpStatus.UNPROCESSABLE_ENTITY, "aluno existente no sistema");
    }

    Aluno aluno = request.toModel();
    try {
        repository.save(aluno);
    } catch (ConstraintViolationException e) { // org.hibernate.exception.ConstraintViolationException
        throw new ResponseStatusException(HttpStatus.UNPROCESSABLE_ENTITY, "aluno existente no sistema (unique constraint)");
    }
    
    // restante do código
}
```

O código acima é suficiente para tratar o erro, mas sabemos que o Spring Boot nos fornece alternativas mais elegantes de tratamento de erros, não é mesmo? Nós podemos implementar um exception handler (controller advice) para capturar esta exceção, deixando assim nosso código no controller mais simples sem a necessidade do bloco `try-catch`. Portanto, basta implementar uma classe de exception handler como abaixo:

```java
import org.hibernate.exception.ConstraintViolationException;
// outros imports

@RestControllerAdvice
public class CustomExceptionHandler {

    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<?> handleDatabaseConstraintErrors(ConstraintViolationException ex, WebRequest request) {

        Map<String, Object> body = Map.of(
                "status", 422,
                "error", "Unprocessable Entity",
                "path", request.getDescription(false).replace("uri=", ""),
                "timestamp", LocalDateTime.now(),
                "message", "aluno existente no sistema (unique constraint)"
        );

        return ResponseEntity
                .unprocessableEntity().body(body); // HTTP 422
    }

}
```

Agora, quando um usuário tentar cadastrar um novo aluno com email existente e, essa requisição bater no banco de dados, a constraint de unicidade vai garantir a integridade dos dados e lançar o erro, e nesse momento a exceção `ConstraintViolationException` será lançada e capturada pelo nosso exception handler; por fim devolvendo a resposta para o usuário como esta aqui:

```json
HTTP/1.1 422
Content-Type: application/json
Transfer-Encoding: chunked
Date: Mon, 25 Mar 2022 14:25:27 GMT

{
    "status": 422,
    "error": "Unprocessable Entity",
    "message": "aluno existente no sistema (unique constraint)",
    "timestamp": "2022-03-25T14:25:27.7656076",
    "path": "/api/contatos"
}
```

Pronto! Estamos retornando uma mensagem de erro mais amigável para o usuário juntamente com um Status HTTP apropriado. Deixar a exceção `ConstraintViolationException` estourar na cara do usuário não seria nada elegante além de possivelmente vazer informações internas do sistema.


## Dicas do especialista

### Validações redundantes não são um problema

Com a constraint `UNIQUE` definida na tabela a lógica de validação no controller pode parecer desnecessária e/ou redundante, e de fato ela é, mas está tudo bem! 

Embora a validação na aplicação não seja de fato 100% eficaz pois ela pode falhar em momentos de alta concorrência, ela nos permite validar os dados antes de bater no banco de dados, ou antes de executar algum procedimento mais custoso na aplicação, como também nos permite logar o erro e lançar uma mensagem mais amigável para o usuário. Tê-la na borda mais externa do sistema permite que a aplicação falhe rapidamente, liberando assim recursos do servidor, como threads, memória, conexões abertas com o banco de dados e outros serviços etc, ou seja, é a implementação de uma boa prática conhecida como **Fail Fast**.

A verdade, é que em aplicações web é muito comum que lógicas de validação de entrada de usuários estejam duplicadas em várias camadas da arquitetura ou aplicação. Para citar algumas: no browser, no dispositivo mobile, no controller da aplicação, nas regras de negócio, nas entidades da JPA e banco de dados.

### Definindo o nome da constraint de unicidade no banco de dados

Se você reparou com atenção, ao definir a nossa constraint de unicidade via anotação `@Column(unique = true)` na entidade `Aluno`, o Hibernate gerou uma constraint com um nome auto-gerado **aleatório**: `uk_rrtn7wgfxo0jfwkhby23f72cn`. E as chances são de que isso seja um problema em alguns times ou projetos de software onde há uma padronização de nomenclatura dos objetos criados no banco de dados, incluindo tabelas, colunas, indexes e contraints. Por esse motivo, para termos controle sobre os nomes das constraints de unicidade gerado pelo Hibernate precisamos utilizar outra anotação da JPA especifica para `UNIQUE` constraints, a anotação `@UniqueConstraint`.

Diferentemente da anotação `@Column`, essa anotação não é utilizada no atributo da entidade, mas sim na classe, via anotação `@Table`, como abaixo:

```java
@Table(uniqueConstraints = { 
    @UniqueConstraint(name = "UK_ALUNO_EMAIL", columnNames = { "email" }) 
})
@Entity
class Aluno {
    // atributos e métodos
}
```

Agora, ao iniciar a aplicação o Hibernate vai gerar nossa `UNIQUE` constraint para coluna `EMAIL` com o nome definido por nós no atributo `name` da anotação `@UniqueConstraint` usando um comando DDL semelhante a este aqui:

```sql
ALTER TABLE aluno
  ADD CONSTRAINT UK_ALUNO_EMAIL UNIQUE (email);
```

É justamente via anotação `@UniqueConstraint` que podemos definir um nome para nossa constraint, quais colunas participam da constraint, além múltiplas constraints diferentes para outras colunas ou mesmo chaves compostas.

### Cuidados ao implementar um exception handler para constraints do banco

Um detalhe é que a exceção `ConstraintViolationException` do Hibernate encapsula todos os erros de constraints lançados pelo banco de dados, sejam estes erros de constraints do tipo `UNIQUE`, `NOT NULL`, `PRIMARY KEY`, `FOREIGN KEY`, entre outras. O que o Hibernate não conseguir validar antecipadamente a nível de aplicação sobra para banco de dados fazer. 

Dessa forma, uma boa implementação de exception handler deve levar isso em consideração olhando para nome da constraint lançada pelo banco e informando a mensagem de erro adequada:

```java
ConstraintViolationException e = // captura exception...

String constraintName = e.getConstraintName(); // extrai nome da constraint
if (constraintName.equals("UK_ingresso_codigo")) {
    throw new ResponseStatusException(UNPROCESSABLE_ENTITY, "codigo do ingresso existente no sistema");
}

if (constraintName.equals("FK_outra_constraint_qualquer")) {
    throw new ResponseStatusException(UNPROCESSABLE_ENTITY, "nota fiscal aponta para um produto inválido");
}

// constraint desconhecida? erro genérico..
logger.error("erro inesperado de constraint no banco: " + constraintName);
throw new ResponseStatusException(BAD_REQUEST, "alguns dos dados submetidos estão incorretos");
```

Aqui a criatividade do desenvolvedor(a) é quem manda! Implemente seu exception handler de acordo com as necessidade do negócio e especificações do time de qualidade.

### Erros de constraints podem ocorrer somente no commit da transação

Vimos que para capturar a exceção `ConstraintViolationException` nós utilizamos um bloco `try-catch` ao invocar o método `save` do `AlunoRepository`, e embora esteja correto do ponto de vista de tratamento de erros ele pode não funcionar como esperado em alguns casos. Para que a constraint entre em ação se faz necessário que o Hibernate envie o comando SQL `INSERT` para o banco de dados, mas o detalhe aqui é que o Hibernate pode enviar este comando tardiamente, por exemplo no final da transação, ou seja, somente no momento do `COMMIT`:

```java
try {
    entityManager.persist(aluno); // não gera INSERT, logo não dispara erros de constraints
    // ...
    // outras lógicas de negócio...
    // ...
    entityManager.getTransaction().commit(); // gera INSERT somente no ultimo momento
} catch (ConstraintViolationException e) {
    // trata erro de constraints
}
```

Não é fácil prever quando o Hibernate poderá enviar os comandos SQL para o banco de dados, mas no caso de persistir uma nova entidade, é comum que esta decisão esteja atrelada ao mapeamento da entidade de alguma forma, por exemplo se usamos uma chave auto-incremento (`IDENTITY`) ou sequence (`SEQUENCE`) no banco. Outros casos tem a ver com uso de operações em cascata, se há uma consulta JPQL ou SQL no meio das operações de escrita etc. Por esse motivo, sempre olhe o SQL gerado nos logs da aplicação, ela é a sua fonte da verdade.

De qualquer forma, se você precisar forçar o envio dos comandos SQL para o banco você pode sempre recorrer ao **flushing manual** do contexto de persistência da JPA. Por exemplo, invocando os métodos `saveAndFlush()` ou simplesmente `flush()`, ambos da interface da `JpaRepository` do Spring Data JPA.

### Artigos que valem a pena a leitura

- [Defining Unique Constraints in JPA](https://www.baeldung.com/jpa-unique-constraints)
- [PostgreSQL - CONSTRAINTS](https://www.tutorialspoint.com/postgresql/postgresql_constraints.htm)
- [JavaDoc: @UniqueConstraint](https://docs.oracle.com/javaee/7/api/javax/persistence/UniqueConstraint.html)
- [A beginner’s guide to flush strategies in JPA and Hibernate](https://vladmihalcea.com/a-beginners-guide-to-jpahibernate-flush-strategies/)
- [How do JPA and Hibernate define the AUTO flush mode](https://vladmihalcea.com/how-do-jpa-and-hibernate-define-the-auto-flush-mode/)
