# Introducão a Bloqueio Otimista

Ao lidar com ambiêntes concorrêntes não podemos ficar refém de uma unica estratégia. Inserir bloqueios exclusivos em todas ocasiões pode se tornar custoso, não é sempre que queremos que a quantidade de threads executadas seja reduzida.

O bloqueio pessimista é a estrategia apropriada para quando estamos lidando com prevenção de conflitos, ou seja, dado uma situação nós precavemos e evitamos atualizações concorrentes em um unico registro. O custo desta estrátegia é que podemos gerar a anomalia de atualização perdida.


## Anomalia de Atualização Perdida

Para entender a anomalia de atualização perdida, imagine que estamos trabalhando em um sistema de estoque, onde diversos funcionarios concorrem a um determinado insumo. 

Suponha que o insumo com id igual a um, é uma camiseta estampada e conta com apenas uma quantidade. A entidade pode ser representada pelo seguinte Json.

```json
    {
        "id":1,
        "nome":"camiseta estampada nike",
        "quantidade":1
    }
```

Agora imagine que o `vendedor 1` e `vendedor 2` estão atendendo clientes diferentes no estabelecimento e estes clientes se interessaram na camiseta. Então ambos os vendedores iniciam a transação de compra, e o `vendedor 1` finaliza a compra, atualizando a quantidade para zero.

```json
    {
        "id":1,
        "nome":"camiseta estampada nike",
        "quantidade":0
    }
```


 O `vendedor 2` não ciente que a quantidade foi alterada para zero, finaliza a compra, e o resultado é que o registro que representa a camiseta fica da seguinte forma. 

```json
    {
        "id":1,
        "nome":"camiseta estampada nike",
        "quantidade":-1
    }
```

A anomalia de atualização perdida foi dada quando o `vendedor 2` acha que a camiseta ainda esta disponivel, porém não esta.

A solução para esta problematica pode ser feita através de um mecanismo de detecção de conflitos. Este mecanismo por si, tera que verificar se o estado da entidade no momento de atualização é o mesmo que no memomento de leitura. Esta estratégia é o que chamamos de Bloqueio Otimista.

## Bloqueio Otimista com JPA

O mecanismo de detecção de conflitos, analisa o estado da entidade, este estado é observado apartir de um criterio de versão. A entidade Insumo teria um atributo com a responsabilidade de armazenar a versão desta entidade. A cada operação de escrita (atualização) esta versão deve ser incrementa. 

É muito comum que a versão de uma entidade seja dada por um campo de data e hora (TimeStamp), este campo deve ser previamente anotado com `@Version`. Veja abaixo como a entidade Insumo ficaria com mecanismo de bloqueio otimista.


```java
    @Entity
    public class Insumo{
        @Id
        @GeneratedValue
        private Long id;
        
        @Column(nullable=false)
        private String nome;

        @Column(nullable=false)
        private Integer quantidade;

        @Version
        private TimeStamp versao;
    }
```

Esta abordagem funciona, porém carregam as seguintes desvantagens:

1. Hora de um sistema não é incrementada sequencialmente, existem casos onde pode retroceder devido a sincronização de hora na rede devido ao Network Time Protocol (NTP).
2. Diferentes sistemas de banco de dados utilizam precisões em periodos diferentes, alguns em nanosegundos, outros em microssegundos.

A melhor estratégia para favorecer o versionamento de entidades em sistemas distribuidos é a utilização de campos inteiros.

Refatoramos nossa entidade para:

```java
    @Entity
    public class Insumo{
        @Id
        @GeneratedValue
        private Long id;
        
        @Column(nullable=false)
        private String nome;

        @Column(nullable=false)
        private Integer quantidade;

        @Version
        private int versao;

        public Insumo(String nome, Integer quantidade){
            this.nome=nome;
            this.quantidade=quantidade;
        }

        @Deprecated
        public Insumo(){}

        public void abaterEstoque(){
            this.quantidade--;
        }
    }
```

OBS: O campo versão nunca deve conter um metodo setter, a responsabilidade de atualizar este atributo é exclusiva do Hibernate.

### Entendendo a dinâmica do mecanismo de Bloqueio Otimista

No momento de realizar uma inserção da entidade Insumo, o atributo `versao` é inicializado com zero. 


```java
    @Transaction
    public void cadastrar(Insumo insumo){
        entityManager.persist(insumo);
    }      
```
Veja abaixo o SQL gerado após a chamada do metodo persist do `EntityManager`.

```sql 
INSERT INTO insumo (nome,quantidade, version, id) VALUES ('Camiseta estampada', 1, 0,1)
```

Deste momento para frente toda atualização neste registro verificara se a versão do registro é igual a versão no momento da consulta.  Quando um insumo é comprado, deve ser abatido a quantidade no estoque.

```java
    @Transaction
    public void comprar(Long idInsumo){
        Insumo insumo=entityManager.find(Insumo.class,idInsumo);

        insumo.abaterEstoque();
        entityManager.merge(insumo);
    }  
```
 Observe o SQL gerado pelo Hibernate:

 ```sql
    SELECT 
        i.nome as nome,
        i.quantidade as quantidade,
        i.versao as versao
        i.id as id
    FROM
        insumo i
    WHERE 
        i.id=?


    UPDATE insumo SET quantidade=0, versao=1
    WHERE id=1 AND versao=0
 ```

Notamos que no momento de fazer a atualização, a condição é que o valor da  versão seja igual a zero, que é o valor retornado no momento da consulta.

Sempre que ocorrer uma atualização, o Hibernate irá filtrar o registro do banco de dados de  acordo com a versão da entidade esperada. Caso a versão tenha mudado, a contagem de linhas atualizadas sera 0 e uma `OptimisticLockException` será lançada.


## Links para Aprofundamento

- [Documentação @Version](https://javaee.github.io/javaee-spec/javadocs/javax/persistence/Version.html)
- [Bloqueio Otimista com JPA e Hibernate com Vlad Mihalcea](https://vladmihalcea.com/optimistic-locking-version-property-jpa-hibernate/)
- [Bloqueio Otimista com JPA e Hibernate com Baeldung](https://www.baeldung.com/jpa-optimistic-locking)

