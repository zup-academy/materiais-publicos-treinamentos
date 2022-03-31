# Evoluindo o modelo de dominio do negocio para lidar com operações concorrentes

Em sistemas distribuidos os maiores gargalos estão em operações de entrada e saida (I/O), seja a leitura ou escrita de uma informação no Banco de Dados (BD), ou uma integração em rede com outros sistemas.

O modelo de dominio de uma aplicação deve ser definido com base em informações que ditam, a necessidade de gravação, e padrões de leitura dos dados. Uma boa modelagem proporcionará uma melhor aplicação de estrategias de concorrência, e melhor performance em consultas.

### Mas porque estamos falando de modelagem de dominio?

Suponha que estamos em um ambiente de alta concorrência, onde estamos sujeitos a necessidade de fazer diversas atualizações a um determinado registro em atributos distintos? Dado que a necessidade de escalabilidade é alta, a alternativa que temos é o `Bloqueio Otimista`. 

`Bloqueio Otimista` detecta o conflito no momento da atualização, atráves da comparação de estado de versionamento, mesmo que atributos não sobrepostos sejam atualizados, o mecânismo de bloqueio otimista implicito abortaria todas transações com versões direntes a atual.

Para compreender melhor veja a entidade referente a uma Postagem em um blog mapeada abaixo. 


```java
@Entity
public class Postagem{
    @Id
    private Long id;
    private String titulo;
    private String corpo;
    private long curtidas;
    private long visualizacoes;
    
    @Version
    private int versao;
    //construtores, getters e setters omitidos
}
```

Imagine agora que existe duas transações desejando atualizar o registro representado abaixo em JSON.
```json
{
    "id"            : 1,
    "titulo"        : "Aprendendo Sobre JPA",
    "corpo"         : "JPA é uma especificacao java",
    "curtidas"      : 1,
    "visualizacoes" : 10,
    "versao"       : 11 
}

```

 A primeira transação atualizará o titulo do blog, enquanto a outra atualizará as curtidas.

1. Atualizando as curtidas
    ```java
        @Transactional
        public void atualizarCurtida(Long idPostagem){
            Postagem post = entityManager.find(Postagem.class, idPostagem);

            post.incrementarCurtida();// aqui é somado mais um a quantidade de curtidas
        }
    ```
    observe o SQL gerado pelo Hibernate:

    ```sql
    SELECT p.*
      FROM postagem p
     WHERE p.id = 1

     UPDATE postagem 
        SET curtidas = 11,  versao = 12
      WHERE id = 1 
        AND versao = 11   
    ```

    Neste momento a versão é incrementada de 11 para 12, e a transação paralela que contem a versão 11, ira atualizar o titulo e não deveria conflitar a esta transação.
    
    porém, verifique o que acontecesse com  segunda transação.

2. Atualizando o titulo
   ```java
        @Transactional
        public void atualizarTitulo(Long idPostagem, String titulo){
            Postagem post = entityManager.find(Postagem.class, idPostagem);

            post.setTitulo(titulo);
        }
    ```
    observe o SQL gerado pelo Hibernate:

    ```sql
    SELECT p.*
      FROM postagem p
     WHERE p.id = 1

     UPDATE postagem 
        SET titulo = 'Aprenda sobre JPA',  versao = 12
      WHERE id = 1 
        AND versao = 11   
    ``` 

    Neste momento é lançado uma `OptimisticLockException`, porque a versão da entidade agora é 12 e não 11.

## Como podemos permitir que atualizações a atributos não sobre postos não sejam abortadas?

Situações como estas exigem uma evolução no modelo de dominio, já que as informações como titulo, visualizações e curtidas devem ser atualizadas independentemente. 

Para este caso criaremos duas novas entidades, uma referente as  visualizações, e a outra as curtidas. As novas entidades conteram um relacionamento bidireicional de `Um para Um`, veja o mapeamento abaixo.

```java
@Entity
public class CurtidaEmPostagem{
    @Id
    @GeneretedValue
    private Long id;

    private long curtidas; 

    @OneToOne
    private Postagem postagem;

    @Version
    private int versao; 
}
```
```java
@Entity
public class VisualizacoesEmPostagem{
    @Id
    @GeneretedValue
    private Long id;

    private long visualizacoes; 

    @OneToOne
    private Postagem postagem;

    @Version
    private int versao;
}
```
Por fim a classe Postagem.

```java
@Entity
public class Postagem{
    @Id
    private Long id;
    private String titulo;
    private String corpo;

    @OneToOne(mappedBy="postagem")
    private CurtidaEmPostagem curtidas;

    @OneToOne(mappedBy="postagem")
    private VisualizacoesEmPostagem visualizacoes;
    
    @Version
    private int versao;
    //construtores, getters e setters omitidos
}
```
Agora as entidades referentes a Curtidas e Visualizacoes possuem seus proprios identificadores de versão, então quando os atributo titulo ou corpo forem atualizados, as versões dos atributos de curtidas e visualizações não são incrementados. 

Veja como funcionaria a dinâmica agora.

1. Atualizando as curtidas
    ```java
        @Transactional
        public void atualizarCurtida(Long idPostagem){
            Postagem post = entityManager.find(Postagem.class, idPostagem);

            post.incrementarCurtida();// aqui é somado mais um a quantidade de curtidas
        }
    ```
    observe o SQL gerado pelo Hibernate:

    ```sql
        SELECT p.*, cp.*
          FROM postagem p 
    INNER JOIN curtida_em_postagem cp
            ON p.curtida_em_postagem_id = cp.id    
     WHERE p.id = 1

     UPDATE curtida_em_postagem 
        SET curtidas = ?,  versao = ?
      WHERE id = ? 
        AND versao = ?   
    ```
    observamos que apenas o atributo de versão da entidade CurtidaEmPostagem é incrementado, desta forma não existirá conflito na atualização de versão da entidade Postagem e VisualizacoesEmPostagem.

