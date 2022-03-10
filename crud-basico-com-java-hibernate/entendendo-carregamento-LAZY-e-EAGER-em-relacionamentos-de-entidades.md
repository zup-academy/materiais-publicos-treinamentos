# Entendendo carregamento LAZY e EAGER em relacionamentos de entidades

Vamos entender como a JPA/Hibernate executa o carregamento dos relacionamentos de uma entidade, dessa forma poderemos decidir onde e quando tais relacionamentos serã carregados ao buscarmos uma entidade do banco de dados com JPA e Hibernate.

## Entendendo o comportamento LAZY nos relacionamentos

Antes de mais nada, assuma as entidades abaixo que representam nosso modelo de domíno para um site de noticias:

```java
@Entity
class Artigo {

    @Id
    @GeneratedValue
    private Long id;

    private String titulo;
    private String descricao;

    @OneToMany
    private List<Comentario> comentarios;

    // métodos getters e setters
}

@Entity
class Comentario {

    @Id
    @GeneratedValue
    private Long id;

    private String conteudo;
    private LocalDateTime criadoEm;

    // métodos getters e setters
}
```

Imagine que tenhámos que buscar um artigo no banco de dados e exibir todos os seus comentários para o usuário. Não seria difícil buscarmos um artigo e logo em seguida invocarmos o seu método `getComentarios`. Para isso, bastaria termos um código semelhante a este:

```java
EntityManager manager = // obtem a EntityManager

Artigo artigo = manager.find(Artigo.class, 24);

for (Comentario comentario : artigo.getComentarios()) {
    System.out.println(comentario.getConteudo());
}
```

Se olharmos o console da IDE, notaremos que duas consultas foram feitas no banco de dados, a primeira para simplesmente carregar o artigo pelo `id` e, a segunda, no momento em que invocamos o método `getComentarios` do artigo dentro do `for` (loop) para carregar todos os comentários do artigo. Os dois `SELECT`'s gerados pelo Hibernate seriam semelhantes estes:

```sql
-- busca artigo
SELECT a.*
  FROM artigo a
 WHERE a.id = ?

-- busca comentarios do artigo
SELECT c.*
  FROM artigo_comentarios ac
 INNER JOIN
    comentario c
        ON ac.comentario_id = c.id
 WHERE 
    ac.artigo_id = ?
```

Para tentar otimizar ao máximo a performance da aplicação, o Hibernate só executa as consultas quando elas são de fato necessárias, evitando assim idas desnecessárias ao banco de dados. Esse comportamento é conhecido como **Lazy Loading**, ou do português, **carregamento preguiçoso**.

O Hibernate, normalmente, só irá disparar a consulta de um relacionamento *Lazy* quando trabalhamos em cima do atributo da entidade (normalmente ao invocar o método getter ou iterarmos sobre a coleção). Neste caso, o segundo `SELECT` só foi disparado porque iteramos a coleção de `Comentario`’s retornada pelo método `getComentarios` da entidade `Artigo`.

Também podemos declarar de forma explícita um relacionamento como Lazy através do atributo `fetch` na anotação de relacionamento, neste caso na anotação `@OneToMany`, como abaixo:

```java
@OneToMany(fecth = FetchType.LAZY)
private List<Comentario> comentarios;
```

Além do `FetchType` do tipo `LAZY`, também podemos utilizar o tipo `EAGER` que tem comportamento oposto ao carregamento preguiçoso.

## Entendendo a estratégia de fetching padrão da JPA e Hibernate

Como vimos, por padrão, o Hibernate usa como estratégia de fetching o carregamento preguiçoso (lazy loading) para relacionamentos `@OneToMany`, mas podemos mudar este comportamento para um **carregamento ansioso**, ou simplesmente **Eager**. Dessa forma, a JPA carregaria os comentários sempre que carregássemos um artigo do banco. Esse comportamento é o oposto ao carregamento Lazy.

Por exemplo, para indicarmos ao Hibernate que a coleção de comentários da entidade `Artigo` será carregada juntamente com seu artigo, basta configurarmos o atributo `fetch` da anotação `@OneToMany` para `EAGER`, como abaixo:

```java
@OneToMany(fecth = FetchType.EAGER)
private List<Comentario> comentarios;
```

A partir desse momento, sempre que carregarmos um artigo sua coleção de comentários será carregado juntamente pelo Hibernate. Portanto, ao executarmos o mesmo trecho de código anterior veríamos no console da IDE um `SELECT` gerado pelo Hibernate semelhante a este:

```sql
-- carrega artigo juntamente com seus comentarios
SELECT a.*,
       c.*
  FROM artigo a
  LEFT OUTER JOIN 
    artigo_comentarios ac
        ON ac.artigo_id = a.id
  LEFT OUTER JOIN 
    comentario c
        ON ac.comentario_id = c.id
 WHERE 
    a.id = ?
```

Repare que o Hibernate gerou um único `SELECT` com dois `LEFT OUTER JOIN`'s para carregar todos os dados do banco em vez de dois `SELECT`'s. Apesar do framework tentar fazer o seu melhor, nem sempre isso será possível por causa de configuração e complexidade do mapeamento. Dito isso, não se assuste se você ver duas ou mais queries no console da IDE ao carregar uma entidade.

Por padrão, todos os relacionamentos `@*ToMany` são considerados Lazy pela JPA, ou seja, os relacionamentos  mapeados com `@OneToMany` e `@ManyToMany` são carregados de maneira preguiçosa por padrão. Enquanto os mapeamentos `@*ToOne`, como `@OneToOne` e `@ManyToOne`, são considerados como Eager por padrão.

Uma curiosidade, embora a especificação JPA tenha este padrão, o Hibernate em si segue uma estratégia de fetching diferente. Para o Hibernate, o padrão de todos os relacionamentos é Lazy, e é exatamente assim que deveria ser. Esta decisão não foi à toa, o time de especialistas e mantenedores do Hibernate entendem que esta convenção é a melhor estratégia para aplicações que almejam alta performance e escalabilidade. Ou seja, em um mundo sem JPA onde trabalhamos diretamente com o Hibernate, teríamos todos os relacionamentos configurados como `LAZY` por default.

## Entendendo o famigerado `LazyInitializationException`

Quando trabalhamos com relacionamentos `LAZY`, é muito importante que os relacionamentos sejam acessados enquanto o contexto de persistência da JPA esteja aberto, caso contrário o Hibernate não conseguirá carregar o relacionamento e ainda lançará um erro informando que o relacionamento não foi inicializado apropriadamente pelo desenvolvedor.

Por exemplo, no exemplo abaixo a `EntityManager` é fechada antes de inicializar (carregar) a coleção de comentários do artigo:

```java
EntityManager manager = // ...

Artigo artigo = manager.find(Artigo.class, 24);

manager.close(); // fecha o contexto de persistência

for (Comentario comentario : artigo.getComentarios()) {
    System.out.println(comentario); // lança LazyInitializationException
}
```

Ao executarmos o código acima, o Hibernate vai buscar o artigo de `id` 24 para nós e, ao invocar o método `getComentarios` da instância encontrada, ele tentará disparar uma nova consulta para o banco de dados. Nada demais, é o comportamento esperado de um relacionamento lazy.

O problema é que, ao invocar o método `getComentarios`, a entidade `Artigo` está *detached*, pois o **contexto de persistência da JPA já foi fechado** antes da coleção ter sido inicializada. Lembre-se, a JPA implementa o conceito de contexto de persistência através da `EntityManager`. Como a entidade não está mais gerenciada (*managed*), fica impossível para o Hibernate realizar a segunda consulta.

Consequentemente, logo que ocorre a invocação do método, o Hibernate lança um erro informando que não foi possível realizar o lazy loading para este relacionamento. Neste caso, a exceção que representa este erro é a famigerada **LazyInitializationException**.

De forma simplista, para contornar este problema, podemos inicializar os relacionamentos Lazy somente dentro de um contexto transacional do Spring, **pois sua duração é equivalente ao tempo de vida do contexto de persistência da JPA**. Portanto, basta garantir que seus métodos de negócio estejam anotados com a anotação `@Transactional`, como a seguir:

```java
@Transactional // contexto inicia e termina com o método
public FileOutputStream imprime(Long artigoId) {

    Artigo artigo = repository.findById(artigoId).orElseThrow(() -> {
        return new ArtigoNaoEncontratoException("artigo não encontrado");
    });

    for (Comentario comentario : artigo.getComentarios()) {
        processaComentario(comentario);
    }

    // restante da lógica de negócio
}
```

A exceção `LazyInitializationException` ocorre frequentemente em aplicações Web quando trabalhamos com relacionamentos Lazy. Existem algumas formas de resolver este problema, mas não se preocupe agora pois veremos com detalhes cada uma delas futuramente.

## Dicas do especialista

### Favoreça relacionamentos Lazy

Quando falamos de aplicações de alta-performance falamos de carregar somente os dados necessários ao buscar entidades no banco de dados, e isso inclui principalmente seus relacionamentos. Infelizmente, neste tipo de aplicação, a estragégia `EAGER` passa a ser um bad smell, o ideal seria que todos os relacionamentos fossem `LAZY` e escrevessemos consultas planejadas via JPQL e Criteria para cada caso de uso. Mas nesse momento da sua jornada, entendemos que a estratégia de fetching padrão da JPA é boa o suficiente para maioria dos casos e, portanto, podemos abraça-la sem medo.

Contudo, se por algum motivo você desconfiar que um relacionamento `*ToOne`, que por padrão é `EAGER`, não será carregado em algum cenário na sua aplicação, favoreça configurá-lo como `LAZY`; da mesma forma, se você tiver 100% de certeza que um relacionamento `@*ToMany` precisa sempre ser carregado juntamente com sua entidade pai, vale refletir se não é melhor configurá-lo como `EAGER`. Lembre-se, quem determina a estratégia escolhida são os casos de uso da sua aplicação.

### Artigos que valem a pena a leitura

- [A beginner’s guide to Hibernate fetching strategies](https://vladmihalcea.com/hibernate-facts-the-importance-of-fetch-strategy/)
- [JPA Default Fetch Plan](https://vladmihalcea.com/jpa-default-fetch-plan/)
- [JPA and Hibernate FetchType EAGER is a code smell](https://vladmihalcea.com/eager-fetching-is-a-code-smell/)