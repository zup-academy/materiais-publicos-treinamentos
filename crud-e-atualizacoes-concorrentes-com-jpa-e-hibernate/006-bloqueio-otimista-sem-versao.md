# Bloqueio Otimista Para Sistemas Legados

Trabalhar com sistemas legados é sempre um grande desafio, tarefas como evoluir e dar manutenção exigem um alto nivel de atenção e esforço. Estes sistemas podem ser de dificil compreensão das regras negocio, dado a má estruturação dos componentes e módulos de codigo.

Outra desvantagem de sistemas legados, é o alto custo para realizar mudanças no schema do Banco de Dados (BD), as vezes pela quantidade de registro ou por risco de impactar outras funcionalidades. 

Caso o ambiente que um sistema com as caracteristicas ditas acima opera, tem um  aumento de requisições, e diversas threads competem para atualizar  registros, é normal que a integridade dos dados fique ameaçada, e neste momento se torna necessário implementar mecânismos de controle de concorrência. 

Por questões de escalabilidade do sistema nem sempre é possível utilizar uma estrátegia de `Bloqueio Pessimista`, então outra alternativa é utilizar o `Bloqueio Otimista`. Recordando a estrátegia do bloqueio otimista trabalha com a detecção de conflitos, no momento da atualização do registro é verificado se o estado que esta em memoria é o mesmo que é presente no BD, caso não seja é lançado uma `OptimisticLockException` e a transação é abortada.

A verificação de igualdade do estado de um registro ou entidade, é realizada da seguinte maneira, existe um campo incremental que é incrementado a cada atualização do registro. No momento da atualização é executada uma comparação verficiando se o valor que esta na memoria da aplicação é igual ao valor que esta no BD.

### Para configurar o Bloqueio Otimista em uma entidade já existente é necessário a inserção de um novo campo no schema, dado a restrição do sistema legado, não é possível utilizar a estratégia?

Para casos onde não é possível inserir um novo campo para controle de versionamento, o `Hibernate` oferece um mecanismo de controle sem versão, é o que chamamos de ` Versionless Optmistic Locking`. 

## Versionless Optmistic Locking

Para habilitar o mecanismo de Bloqueio Otimista sem versão, iremos utilizar a anotação @OptimisticLocking a nivel de entidade. A anotação recebe um argumento responsavel por identificar a estrátegia de bloqueio otimista.

### OptimisticLockType - A estrátegia de Bloqueio Otimista.

O atributo OptimisticLockType carrega 4 estrátegias de bloqueio, dentre elas estão:

1. `NONE`: Ausencia de mecanismo de bloqueio otimista.
2. `VERSION`: O mecanismo de bloqueio otimista implicito (padrão) utiliza um carimbo de data e hora ou atributo inteiro numerico para versão.
3. `DIRTY`: O mecanismo de bloqueio otimista utiliza apenas os atributos modificados para verficação de estado.
4. `ALL`: o mecanismo de bloqueio otimista utilizara todos atributos da entidade, para verficação de estado.

As estrátegias `ALL` e `DIRTY` requerem uma anotação `@DynamicUpdate`, para que a instrução da operação de atualização sejam escritas contendo na clausula `WHERE` todos os atributous ou apenas os que foram atualizados.  É importante ressaltar que o estado da entidade utilizado é o mesmo que foi armazenado ao `Contexto de Persistencia` durante a primeira consulta.

## Entendendo o `OptimisticLockType.ALL`

Dado a entidade pessoa mapeada abaixo, observer como funciona a operação de atualização.

```java
@Entity
@DynamicUpdate
@OptimisticLocking(type = OptimisticLockType.ALL)
public class Pessoa{
    @Id
    private Long id;
    private String nome;
    private String cpf;
    //getters, setters e outros metodos omitidos
}
```
 Dado que será atualizado apenas o nome da pessoa, observe como o mecânismo de bloqueio otimista verificara o estado.

 ```java
 @Transacional
 public void  atualizarNome(Long id, String novoNome){
     Pessoa pessoa = entityManager.find(Pesoa.class, id);
     pessoa.setNome(novoNome);
 }
 ```

 Saida gerada pelo Hibernate: 

 ```sql
    SELECT p.*
    FROM produto p
    where id = 1


    UPDATE pessoa 
       SET nome = 'Jordi Henrique Silva'
     WHERE id   = 1 
       AND cpf  = '12345678990' 
       AND nome = 'Jordi Henrique Marques da Silva'
 ```

 Notamos que todos os campos foram utilizados nas comparações na clasula `WHERE`, caso contivesse mais atributos, todos eles seriam utilizados.

 A estratégia de verificação de conflitos `OptimisticLockType.ALL` é muito util quando a tabela não pode inserir um campo para versionamento. O mecanismo quando utiliza esta estrategia, unifica os atributos da entidade em atributo global de versão, o risco desta estrategia é que se transações simultaneas estão atualizando conjunto de atributos não sobrepostos conflitos não poderam ser indentificados.

 ### `OptimisticLockType.DIRTY`

 Já a estrategia `DIRTY` permite que transações simultaneas que atualizam atributos distintos ocorram sem conflitos. Isto so é possível por que o mecanismo de `Bloqueio Otimista` utiliza os atributos que foram atualizados como identificador de estado, então sempre que as transações utilizem atributos distintos sera possivel identificar o estado da entidade.

Para entender seu funcionamento iremos imaginar a seguinte dinâmica. Existe uma entidade Produto que esta mapeada abaixo:


 ```java
@Entity
@DynamicUpdate
@OptimisticLocking(type = OptimisticLockType.DIRTY)
 public class Produto{
    @Id 
    private Lond id;
    private String titulo;
    private BigDecimal preco;

    //getters,setters e demais metodos omitidos 

 }
 ```

 Dentre os registros desta entidade, temos o registro que representa um cubo magico tradicional, seu registro pode ser representado pelo seguinte `JSON`:

```json
 {
     "id":1,
     "titulo": "Cubo Magico 3x3x3",
     "preco": 38.9
 }
```
 Agora imagine que existem duas transações que estão responsaveis por atualizar atributos diferentes, uma para o preço e outra para titulo. 

 ### Transação para atualizar o preço do registro de um produto.

 ```java

 @Transaction
 public class AtualizarPreçoProduto(Long id, BigDecimal preco){
    Produto produto = entityManager.find(Produto.class, id);

    produto.setPreco(preco);

 }
 ```

 Observe como é gerado o SQL pelo Hibernate:
```sql
    SELECT p.*
    FROM produto p
    where id = 1

    UPDATE produto
       SET preco = 35.6
     WHERE id    = 1
       AND preco = 38.9

```

verificamos que o atributo preco que foi utilizado para verificar o estado da entidade. 

 ### Transação para atualizar o titulo do registro de um produto.


 ```java
 @Transaction
 public class AtualizarTituloProduto(Long id, String titulo){
    Produto produto = entityManager.find(Produto.class, id);

    produto.setTitulo(titulo);

 }
 ```

Observe como é gerado o SQL pelo Hibernate:
```sql
    SELECT p.*
    FROM produto p
    where id = 1

    UPDATE produto
       SET titulo = 'Cubo Magico Elpluze 3x3x3'
     WHERE id     = 1
       AND titulo = 'Cubo Magico 3x3x3'

```

Como ambas as transações modificam atributos distintos o mecanismo de bloqueio otimista, é capaz de permitir que a atualização funcione sem conflitos.
  


## Links para aprofundamento

- [Lock Otimista sem versao Vlad Mihalcea](https://vladmihalcea.com/how-to-prevent-optimisticlockexception-using-hibernate-versionless-optimistic-locking/)