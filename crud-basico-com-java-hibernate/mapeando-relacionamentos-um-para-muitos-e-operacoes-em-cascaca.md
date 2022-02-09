# Mapeando relacionamentos um para muitos e operações em cascata

Agora, vamos aprender como mapear relacionamentos um para muitos com JPA e Hibernate através da anotação `@OneToMany` e também como tirar proveito do framework ORM para disparar operações em cascata simplificando assim o código escrito por nós.

## Mapeando um relacionamento `@OneToMany` com JPA e Hibernate

Imagine que precisamos modelar um sistema para emissão de notas fiscais. Para isso, precisamos modelar duas entidades importantes nesse domínio: **Nota Fiscal** e **Item de Nota Fiscal**. Pensando numa modelagem orientada a objetos, teríamos algo como abaixo:

```java
@Entity
class NotaFiscal {

    @Id
    @GeneratedValue
    private Long id;

    private String numero;
    private BigDecimal valor;

    // outros atributos
}

@Entity
class Item {

    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    private Produto produto;

    private Integer quantidade;

    // outros atributos

}
```

Agora, precisamos relaciona-las, afinal de contas, uma nota fiscal pode possuir um ou mais itens. Se pensarmos bem, em um modelo relacional teríamos um relacionamento **um para muitos** entre as entidades. Para expressar esse relacionamento, nós precisamos de um atributo que guarde todos os itens de uma nota fiscal. Uma maneira de expressá-lo seria através de uma coleção Java, como um `List`. O código ficaria:

```java
@Entity
class NotaFiscal {

    // demais atributos

    private List<Item> itens;
}
```

O nosso mapeamento está incompleto, pois a JPA não sabe como mapear nem tratar este tipo de objeto no banco. Assim como fizemos com o mapeamento de muitos para um usando a anotação `@ManyToOne`, podemos mapear a cardinalidade **um para muitos** através da anotação `@OneToMany`. O código da entidade `NotaFiscal` ficaria como a seguir:

```java
@Entity
class NotaFiscal {

    // demais atributos

    @OneToMany
    private List<Item> itens;
}
```

Com este mapeamento, sempre que carregarmos um objeto do tipo `NotaFiscal` poderemos também acessar todos os seus itens associados.

## Entendendo a convenção para relacionamentos `@OneToMany` da JPA

A convenção da JPA para relacionamentos `@OneToMany` se dá através de uma tabela de relacionamento (ou tabela de junção), como podemos ver no diagrama a seguir:

![Relacionamento @OneToMany via tabela de junção (@JoinTable)](imagens/MER-one-to-many-with-join-table.png "Relacionamento @OneToMany via tabela de junção (@JoinTable)")

Esta tabela de relacionamento possui as seguintes características:

- Nome da tabela será `<nome tabela pai>_<nome tabela filha>`;
- Esta tabela terá duas colunas FKs, uma para PK da tabela pai e outra para PK da tabela filha;
- O nome da coluna FK da tabela pai será o nome da tabela pai concatenado com "_" mais o nome da
coluna PK da tabela pai;
- Enquanto a coluna FK da tabela filha, terá o nome do atributo de relacionamento da entidade pai concatenado com "_" mais o nome da coluna PK da tabela filha;
- A coluna FK da tabela filha terá uma constraint de unicidade;

No fim, para o relacionamento entre `NotaFiscal` e `Item`, teríamos a tabela `NOTA_FISCAL_ITEM`, na qual possuiria duas colunas FKs: `NOTA_FISCAL_ID` e `ITENS_ID` (repare que o nome da coluna usa o nome da propriedade `itens`, ou seja, está no plural).

Para mudar a convenção, podemos usar a anotação `@JoinTable` e, através de seus atributos, definir o nome da tabela, os nomes das colunas FKs e muito mais. O código ficaria:

```java
@OneToMany
@JoinTable(name="ITENS_DE_NOTA",
    joinColumns=@JoinColumn(name="NOTA_ID"),
    inverseJoinColumns=@JoinColumn(name="ITEM_ID", unique=true)
)
private List<Item> itens;
```

Se não quisermos uma tabela de relacionamento, é possível mapearmos uma **coluna de relacionamento** (ou coluna de junção) na entidade filha. A entidade filha teria uma coluna FK para a PK da tabela pai. Para isso, precisamos usar a anotação `@JoinColumn`, como no exemplo abaixo:

```java
@OneToMany
@JoinColumn(name="NOTA_ID", nullable=false)
private List<Item> itens;
```

Ao gerar o schema, o Hibernate nos entregará um modelo mais simples para representar o relacionamento um para muitos utilizando uma coluna de junção:

![Relacionamento @OneToMany via coluna de junção (@JoinColumn)](imagens/MER-one-to-many-with-join-column.png "Relacionamento @OneToMany via coluna de junção (@JoinColumn)")

Repare que além de termos definido a relação via coluna de junção, nós também indicamos ao Hibernate que queremos que o nome da coluna na tabela `ITEM` seja `NOTA_ID` em vez da convenção da JPA, e que esta mesma coluna seja obrigatória (constraint `NOT NULL` no banco de dados).

É possível customizar ainda mais o mapeamento usando as anotações `@JoinTable` e `@JoinColumn`. Para saber mais detalhes sobre as duas anotações, consulte a documentação (Javadoc).

## Persistindo objetos envolvidos em relacionamentos `@OneToMany`

Precisamos adicionar um novo item associado com uma nota fiscal. Poderíamos criar um Repository no Spring Boot para entidade `Item` com todas as operações básicas de persistência, mas será que isto é necessário? A entidade
`Item` é um conceito relativamente fraco dentro do nosso domínio, pois um item só deve existir enquanto uma nota fiscal existir. Além disso, as chances são de que não tenhámos um cadastro de itens no sistema, mas sim uma funcionalidade que nos permita adicionar itens à uma nota fiscal já existente dentro do sistema, ou mesmo criar uma nota fiscal com todos seus itens inclusos.

O ideal é trabalharmos através da coleção de itens existente em `NotaFiscal`, seja para adicionar um novo item ou mesmo remover um já existente. Para isso, precisamos instanciar um novo `Item`, buscar uma `NotaFiscal` já cadastrada no banco e, por fim, adicionar este novo `Item` aos itens da nota fiscal que buscamos. O código seria similar a este:

```java
/**
 * (assuma que o código está dentro de um contexto transacional)
 */ 

// instancia novo item
Item novoItem = new Item(produto, 2);

// busca nota fiscal existente
NotaFiscal nota = entityManager.find(NotaFiscal.class, 42);

// adiciona item à lista de itens da nota
nota.getItens().add(novoItem);

// atualiza a nota fiscal
entityManager.merge(nota);
```

Ao atualizar a nota fiscal, o contexto de persistência da JPA se encarregaria de inserir o item já associado com a nota. Contudo, ao executarmos o trecho de código acima um erro ocorre: a exceção lançada é uma `IllegalStateException`, na qual tem como causa raiz uma `TransientObjectException`. Estamos adicionando uma instância transiente de `Item` em uma instância managed de `NotaFiscal`, e isso não é possível. Resumindo o problema, **não podemos relacionar entidades transientes com objetos managed**.

A solução aqui é tornar a instância de `Item` em managed antes de associa-la a instância managed de `NotaFiscal`, dessa forma ela existiria tanto no banco de dados quanto no contexto de persistência da JPA. Para isso, poderíamos simplesmente persistir o item antes de adiciona-lo a lista de itens:

```java
// instancia novo item
Item novoItem = new Item(produto, 2);
entityManager.persist(novoItem); // a entidade torna-se managed

// restante do código
```

Apesar de simples, isso pode se tornar trabalhoso quanto tivermos diversas coleções numa mesma entidade. Precisamos que nosso framework ORM, no caso a JPA, faça isso por nós. O ideal seria que, ao atualizarmos a instância de `NotaFiscal`, o contexto de persistência detectasse as instâncias transientes no atributo `itens` e as persistissem automaticamente em vez de lançar a exceção.

Para isso, precisamos utilizar operações em cascata em relacionamentos da JPA.

## Utilizando operações em cascata

A JPA nos fornece esse recurso através do atributo `cascade` existente nas anotações de cardinalidade dos relacionamentos. Através dele, podemos **propagar em cascata** as operações da `EntityManager` (como `persist`, `merge`, `remove` etc) para os relacionamentos de uma entidade. Para definir qual o tipo de propagação, nós usamos a `enum` **`CascadeType`**, na qual possui 6 tipos:

- `CascadeType`.**PERSIST**: Propaga a operação `persist` para os relacionamentos da entidade. Ao persistir a
entidade pai, as entidades filhas também serão persistidas;
- `CascadeType`.**MERGE**: Propaga a operação `merge` para os relacionamentos da entidade. Ao atualizar a
entidade pai, as entidades filhas também serão atualizadas;
- `CascadeType`.**REMOVE**: Propaga a operação `remove` para os relacionamentos da entidade. Ao remover a
entidade pai, as entidades filhas também serão removidas;
- `CascadeType`.**REFRESH**: Propaga a operação `refresh` para os relacionamentos da entidade. Ao recarregar
do banco o estado da entidade pai, as entidades filhas também terão seus estados recarregados;
- `CascadeType`.**DETACH**: Propaga a operação `detach` para os relacionamentos da entidade. Ao desvincular
a entidade pai, as entidades filhas também serão desvinculadas do contexto de persistência;
- `CascadeType`.**ALL**: Propaga todas as operações acima para os relacionamentos da entidade;

Agora, podemos propagar a atualização da nota fiscal para seus itens. Para isso, poderíamos definir o tipo de propagação para `CascadeType.MERGE`, **pois a operação que está acontecendo sobre a instância de `NotaFiscal` é um `merge`**. O código do mapeamento do relacionamento entre `NotaFiscal` e `Item` ficaria como a seguir:

```java
@Entity
class NotaFiscal {

    // demais atributos

    @OneToMany(cascade=CascadeType.MERGE)
    private List<Item> itens;
}
```

Apenas o `CascadeType.MERGE` seria suficiente para resolver o problema de `TransientObjectException`, mas podemos pensar um pouco melhor sobre nosso domínio. De acordo com o nosso modelo, a entidade `Item` depende exclusivamente da existência de `NotaFiscal`, ou seja, ao persistirmos uma nova nota fiscal, é importante que os itens desta nova nota fiscal também sejam persistidos em cascata; da mesma forma, se removermos uma nota fiscal do banco, queremos que seus itens também sejam removidos em cascata, pois não faz sentido a existência deles sem a nota fiscal.

Tendo em vista que nosso modelo requer esta dependência estreita entre `NotaFiscal` e `Item`, então, podemos definir o `cascade` como `CascadeType.ALL` e deixar com que a JPA propague as demais operações para o relacionamento quando necessário. Sendo, nosso código ficaria melhor mapeado da seguinte maneira:

```java
@Entity
class NotaFiscal {

    // demais atributos

    @OneToMany(cascade=CascadeType.ALL)
    private List<Item> itens;
}
```

Com a operação em cascata configurada, não precisamos mais persistir a instância de `Item` antes de associa-la a sua nota fiscal.

## Removendo objetos em um relacionamento `@OneToMany`

Da mesma forma que um usuário adiciona um item em uma nota fiscal, ele também pode remover este mesmo item posteriormente, se assim desejar. Semelhante ao que vimos no trecho de código anterior, para remover um item, nós também devemos trabalhar em cima da coleção de itens existente na nota fiscal. O código ficará similar a este:

```java
/**
 * (assuma que o código está dentro de um contexto transacional)
 */ 

Item item = new Item();
item.setId(2); // usar um ID que exista no banco

NotaFiscal nota = entityManager.find(NotaFiscal.class, 42);

boolean removido = nota.getItens().remove(item);
if (removido) {
    System.out.println(">> Item REMOVIDO!");
}

entityManager.merge(nota);
```

Ao executar o código acima nenhum erro ocorre, mas o item associado a nota fiscal **não** é removido do banco de dados. Como posso saber? Basta verificar os logs da aplicação e perceber que:
1. não foi impresso a linha `">> Item REMOVIDO!"`, ou seja, não entrou no `if`;
2. o Hibernate não gerou nenhum comando SQL de `DELETE` para remover a linha da tabela;

Por que isso aconteceu se a lógica do código parece estar correta?

## Definindo a identidade de uma entidade

Quando mapeamos uma entidade, a JPA nos obriga a mapear o identificador único (a chave primária da tabela) da nossa entidade com a anotação `@Id`, pois a JPA precisa ter conhecimento de como diferenciar uma entidade de outra dentro do contexto de persistência e no banco de dados. E, muitas vezes, esse mesmo identificador único é o que difere as entidades dentro do nosso modelo de domínio, ele é a **identidade** da nossa entidade.

Uma regra importante ao trabalhar no mundo orientado a objetos é que, **toda entidade tem uma identidade**. Essa identidade não necessariamente precisa ser a PK da tabela no banco, mas é comum que ela seja nas entidades mais simples. Por exemplo, uma entidade `Cliente` poderia ter o atributo `cpf` ou mesmo `email` como identidade e, ainda assim, ter o atributo `id` como chave primária.

A JPA tem ciência que o atributo `id` representa a identidade da nossa entidade `Item` através da anotação `@Id`, mas o nosso modelo orientado a objetos não sabe disso. E é exatamente neste trecho de código que podemos perceber que o problema não é da JPA, mas sim do nosso modelo:

```java
boolean removido = nota.getItens().remove(item);
if (removido) {
    System.out.println(">> Item REMOVIDO!");
}
```

O método `remove` da coleção não encontrará a entidade `item` com `id` 2 e retornará `false`, e consequentemente, acabará não removendo a entidade do banco de dados, pois a **coleção não foi modificada**.

Para que seja possível remover uma instância de `Item` da coleção, precisamos primeiramente definir sua **identidade de negócio** (_business key_), que no nosso caso, é o próprio `id` da entidade. Após esta definição estar clara para o desenvolvedor(a), precisamos agora sobrescrever dois métodos importantes de um objeto Java, os métodos `equals` e `hashCode`, como no código abaixo:

```java
@Entity
public class Item {

    // ...

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + ((id == null) ? 0 : id.hashCode());
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        Item other = (Item) obj;
        if (id == null) {
            if (other.id != null)
                return false;
        } else if (!id.equals(other.id))
            return false;
        return true;
    }
}
```

Estes dois métodos são bastante utilizados pela API de Collections do Java, em operações para adicionar, remover ou buscar itens da coleção. E é exatamente esta API (`List`) que estamos usando para representar o relacionamento entre `NotaFiscal` e `Item`.

Eu sei que o código acima assusta um pouco, mas não se preocupe, você não precisa implementa-lo na mão. No dia a dia, estes métodos são gerados pela sua IDE com apenas alguns cliques. Mas não se engane, seu papel como desenvolvedor(a) é definir corretamente quais atributos da entidade representam sua identidade, e isso muitas vezes significa conversar com os especialistas de negócio.

## Removendo registros orfãos do banco de dados

Após remover um item da nota fiscal, o relacionamento entre eles foi desfeito. Contudo, o registro na tabela de itens ainda existe, ele não foi removido do banco de dados. Por que isto acontece se estamos usando `CascadeType.ALL`?

A verdade é que, mesmo com o uso do `CascadeType.ALL` (que contém `CascadeType.REMOVE`), a JPA não pode assumir que ao desfazer um relacionamento entre duas entidades ela deve deletar a entidade filha. Por isso, o que de fato é deletado do banco de dados é apenas o relacionamento entre elas. No fim, as entidades filhas que não foram deletadas do banco e não possuem mais referência para a entidade pai se tornam entidades **órfãos**.

Na JPA temos a possibilidade de usar o atributo `orphanRemoval` em relacionamentos `@OneToOne` e `@OneToMany` para remover entidades órfãos. Dessa forma, ao removermos um item da coleção da nota fiscal, a JPA se asseguraria de deletar todos os itens órfãos da tabela. Nosso mapeamento ficaria assim:

```java
@Entity
class NotaFiscal {

    // demais atributos

    @OneToMany(
        cascade=CascadeType.ALL, 
        orphanRemoval=true // habilita deleção de registro orfãos
    )
    private List<Item> itens;
}
```

É muito comum a confusão entre `CascadeType.REMOVE` e `orphanRemoval=true`, pois ambas podem deletar
as entidades órfãos. A primeira só é disparada quando ocorre uma operação `remove` (da `EntityManager`) na entidade pai, enquanto a `orphanRemoval`, além de atuar como a `CascadeType.REMOVE`, ainda atua em deleções feitas diretamente nos relacionamentos. Se usarmos a `orphanRemoval=true`, não precisamos declarar a `CascadeType.REMOVE`, pois sua configuração já está implícita.

É importante entender que definir `orphanRemoval` como `false` (que é o default) indica ao Hibernate que ao remover um item da coleção de uma entidade **somente a relação entre elas é desfeita no banco de dados**. E para essa relação ser desfeita no banco de dados o Hibernate pode gerar tanto um `DELETE` quanto um `UPDATE` para desfazer o vinculo entre entidade filha de pai.

Para entender melhor o que estou querendo dizer, imagine que tenhamos um mapeamento com `orphanRemoval` setado para `true` e usando uma coluna de junção (`@JoinColumn`) como abaixo:

```java
@JoinColumn // definimos uma coluna de junção
@OneToMany(
    cascade=CascadeType.ALL, 
    orphanRemoval=true // remove orfãos
) 
private List<Item> itens;
```

Agora, imagine que tenhámos essa lógica de negócio para remover o primeiro item de uma nota fiscal. Para isso, basta carregarmos a entidade `NotaFiscal` do banco de dados e remover seu item:

```java
NotaFiscal nota = entityManager.find(NotaFiscal.class, 42);
nota.getItens.remove(0); // remove primeiro item
```

Ao executar o código acima o Hibernate vai gerar alguns comandos SQL, incluindo um `DELETE` para o item, semelhantes a estes aqui:

```sql
SELECT n.*
  FROM nota_fiscal n
 WHERE n.id = ?

DELETE FROM item WHERE id = ?
```

Se por acaso definirmos o `orphanRemoval` como `false` e executarmos o mesmo trecho de código anterior, o Hibernate irá gerar o seguinte SQL:

```sql
SELECT n.*
  FROM nota_fiscal n
 WHERE n.id = ?

UPDATE item 
   SET nota_fiscal_id = NULL, -- desfaz a relação
       produto_id     = ?,
       quantidade     = ?
 WHERE id = ?
```

Repare que o Hibernate gerou um comando `UPDATE` em vez de um `DELETE`, pois não há necessidade de deletar a entidade filha, mas somente sua relação com a entidade pai. Tudo vai depender de como definimos o mapeamento do relacionamento, ou seja, se foi mapeado como `@JoinTable` ou `@JoinColumn`, se o atributo `orphanRemoval` está habilitado ou não, o tipo de `cascade` configurado etc.

## Dicas do especialista

### Favoreça coluna de junção em vez de tabela de junção

Por padrão, em relacionamentos `@OneToMany` a JPA gera a relação no banco de dados utilizando uma tabela de junção (`@JoinTable`), mas entendemos que esta não é a melhor abordagem para este tipo de relacionamento em um modelo relacional na maioria dos casos.

Dessa forma, sempre que possível, favoreça usar a anotação `@JoinColum` para indicar a JPA que o modelo gerado no banco de dados deve ser feita via uma coluna de junção em vez de uma tabela de junção. Isso facilitará não só a escrita de consultas nativas em SQL, como também pode colaborar para o desempenho destas consultas ao evitar JOINs entre tabelas, juntamente com o menor uso de storage e menor consumo de memória para indexação.

### Boas práticas ao implementar `equals` e `hashCode` com Hibernate

Implementar adequadamente os métodos `equals` e `hashCode` nem sempre será uma tarefa fácil, pois algumas vezes não será tão simples satisfazer o modelo de domínio e o Hibernate ao mesmo tempo.

Por isso, indicamos os artigos da JBoss e do blog do Vlad Mihalcea abaixo na qual descrevem algumas estratégias, boas práticas e problemas comuns ao implementar estes dois métodos:

- [JBoss Wiki: Equals and HashCode](https://community.jboss.org/wiki/EqualsandHashCode)
- [The best way to implement equals, hashCode, and toString with JPA and Hibernate](https://vladmihalcea.com/the-best-way-to-implement-equals-hashcode-and-tostring-with-jpa-and-hibernate/)

### Artigos que valem a pena a leitura

- [The best way to map a @OneToMany relationship with JPA and Hibernate](https://vladmihalcea.com/the-best-way-to-map-a-onetomany-association-with-jpa-and-hibernate/) 
- [How does orphanRemoval work with JPA and Hibernate](https://vladmihalcea.com/orphanremoval-jpa-hibernate/)
- [The best way to implement equals, hashCode, and toString with JPA and Hibernate](https://vladmihalcea.com/the-best-way-to-implement-equals-hashcode-and-tostring-with-jpa-and-hibernate/)
- [Toda entidade tem uma identidade](http://blog.triadworks.com.br/toda-entidade-tem-uma-identidade)