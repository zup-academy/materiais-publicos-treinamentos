# Definindo constraint de unicidade para múltiplos atributos com JPA e Hibernate

Nesse conteúdo veremos como podemos definir constraints de unicidade para múltiplos atributos de uma entidade. Vamos conhecer a anotação `@UniqueConstraint` da JPA e aprender como utilizá-la para declarar constraints `UNIQUE` em duas ou mais colunas de uma tabela.

## Definindo constraints de unicidade simples

Imagine que em uma plataforma de cursos online precisamos cadastrar novos usuários para consumir os diversos treinamentos, para isso o desenvolvedor(a) modelou a entidade que representa o usuário do sistema dessa forma:

```java
@Entity
class Usuario {

    @Id
    @GeneratedId
    private Long id;

    private String nome;
    private String cpf;
    private String email;

    // demais atributos, construtores e getters
}
```

Um dos requisitos da plataforma é que uma pessoa física (CPF) possa ter múltiplos usuários desde que seu email não se repita, ou seja, não é permitido cadastrar usuários duplicados no banco de dados tendo como chave de unicidade seus atributos `cpf` e `email`. A regra parece um pouco confusa, por esse motivo, um desenvolvedor(a) mais desatento(a) poderia acabar mapeando os atributos da entidade com `@Column` como abaixo:

```java
@Entity
class Usuario {

    @Id
    @GeneratedId
    private Long id;

    private String nome;

    @Column(unique = true)
    private String cpf;

    @Column(unique = true)
    private String email;

    // demais atributos, construtores e getters
}
```

Este mapeamento até faria sentido se não houve uma relação de unicidade dos atributos, mas de acordo com o requisito de negócio eles tem uma relação: lembre-se, uma pessoa física pode ter múltiplos usuários desde que o email seja único. Se olharmos para o que o Hibernate vai gerar no banco, você veria dois comandos DDL semelhantes a estes aqui:

```sql
ALTER TABLE usuario
  ADD CONSTRAINT UK_rrtn7wgfxo7c98d8c9d08c09d UNIQUE (cpf);

ALTER TABLE usuario
  ADD CONSTRAINT UK_jjlcnk33jkljfwkhby23fj2ck UNIQUE (email);
```

Ou seja, teríamos 2 constraints de unicidade: uma para a coluna `CPF` e outra para coluna `EMAIL`; o que quer dizer que o banco não permitiria dois usuários com o mesmo CPF **ou** com o mesmo email na tabela. E não é isso que o requisito de negócio deseja! Mas como mapear uma única constraint para essas 2 colunas?

## Definindo uma constraint composta para entidade `Usuario`

Como vimos, a regra de negócio esperada é que uma pessoa física possa ter múltiplos usuários (registros na tabela), ou seja, seu único CPF vai se repetir, porém seu email precisa ser único por CPF. Para isso, precisamos definir uma constraint de unicidade composta (ou **Composite Unique Key**), e com a JPA fazemos isso através da anotação `@UniqueConstraint`. E, diferentemente da anotação `@Column`, essa anotação não é utilizada no atributo da entidade, mas sim na classe (a nível de tabela), via anotação `@Table`, como abaixo::

```java
@Table(uniqueConstraints = { 
    @UniqueConstraint(name = "Unique_usuario_cpf_email", columnNames = { "cpf", "email" }) 
})
@Entity
class Usuario {

    @Id
    @GeneratedId
    private Long id;

    private String nome;
    private String cpf;
    private String email;

    // demais atributos, construtores e getters
}
```

Repare no uso da anotação `@UniqueConstraint`: não só declaramos quais colunas participam da constraint composta via atributo `columnNames`, como também definimos o nome da constraint, `Unique_usuario_cpf_email`, que o Hibernate vai gerar no banco por causa do atributo `name` da anotação - o que pode ser bem útil para ter controle sobre os objetos criados no schema do banco de dados.

Pronto! Após declararmos nossa constraint de unicidade composta, o Hibernate vai gerar o seguinte comando DDL:

```sql
ALTER TABLE usuario
  ADD CONSTRAINT Unique_usuario_cpf_email UNIQUE (cpf, email);
```

Mas e se **também** quisermos que o email de um usuário seja único para todo o sistema e não somente para uma pessoa (CPF)? Certamente precisaríamos de outra constraint de unicidade na tabela.

## Definindo múltiplas constraints de unicidade para uma entidade

Embora não exista um requisito de negócio para que o email de um usuário seja único para todo o sistema, não é incomum que esse tipo de requisito exista no mundo real.

Portanto, nós poderíamos declarar esta nova constraint ainda via o uso da anotação `@UniqueConstraint`:

```java
@Table(uniqueConstraints = { 
    @UniqueConstraint(name = "Unique_usuario_cpf_email", columnNames = { "cpf", "email" }),
    @UniqueConstraint(name = "UK_email", columnNames = { "email" }) // nova constraint
})
@Entity
class Usuario {

    // atributos, construtores e getters
}
```

Na anotação `@Table`, o atributo `uniqueConstraints` recebe um array de `@UniqueConstraint`'s, por esse motivo conseguimos declarar **múltiplas constraints simples e compostas** diretamente na entidade. Ao reiniciar a aplicação o Hibernate vai gerar os seguintes comandos DDL:

```sql
ALTER TABLE usuario
  ADD CONSTRAINT Unique_usuario_cpf_email UNIQUE (cpf, email);

ALTER TABLE usuario
  ADD CONSTRAINT UK_email UNIQUE (email);
```

Simples, não é?

Graças ao atributo `uniqueConstraints` na anotação `@Table` nós conseguimos declarar múltilplas constraints de unicidade na entidade, sejam elas simples ou compostas.

## Dicas do especialista

### Evite mesclar `@UniqueConstraint` e `@Column(unique=true)` nas entidades

Nós também poderíamos mesclar as declarações de constraints de unicidade via as duas anotações: `@UniqueConstraint` e `@Column`. A verdade é que há grandes chances de você encontrar código dessa forma, e está tudo bem, contudo entendemos que essa abordagem pode dificultar a vida dos desenvolvedores(as).

Embora pareça tentador mesclar as duas abordagens, nós acreditamos que para o caso de múltiplas constraints, faz mais sentido favorecer o uso da anotação `@UniqueConstraint` mantendo todas as constraints no mesmo local, facilitando a leitura e manutenção futura do código para outros desenvolvedores(as).

### Artigos que valem a pena a leitura

- [JavaDoc: @UniqueConstraint](https://docs.oracle.com/javaee/7/api/javax/persistence/UniqueConstraint.html)
- [Defining Unique Constraints in JPA](https://www.baeldung.com/jpa-unique-constraints)
