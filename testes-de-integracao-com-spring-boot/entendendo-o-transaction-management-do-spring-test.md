#  Entendendo o Transaction Management do Spring Test

Aplicar testes automatizados é uma prática que visa entregar uma garantia que o codigo implementado funciona como o esperado, e que esteja seguro para ir para um ambiente de produção. 

No momento de construção do conjuntos de testes do sistema, olhamos para cada unidade e verificamos o seu comportamento, e isso é o que chamamos de testes de unidade.  Os testes de unidades cobrem muitos dos casos, porém, existem algumas funcionalidades dos sistemas onde o comportamento de uma unidade depende da realização de operações de `escrita` e `leitura` do banco de dados. E nestes casos temos a complexidade da integração com o sistema de banco de dados e também do **`controle transacional`**.

Relembramos que a responsabilidade das `transações (Transactions)` é garantir que as informações sejam atomicas, consistentes, integras e duraveis. Isso signfica que se no momento de operações de escrita ocorrer a violação de `constraints` a transação deve ser revertida, ou se existir um problema de concorrencia a integridade das informações sejam preservadas.

Toda essa complexidade deve ser levada em consideração no momento da criação dos testes das funcionalidades que fazem acesso ao banco de dados. Então devemos delimitar o inicio da transação e o final, e para cada caso definir quando será feito o `commit` ou `rollback`.  Definir um componente para que gerencie o contexto transacional em testes pode ser contra produtivo, dado que a maior parte do tempo do desenvolvedor não estará em criar codigo que gerer valor ao cliente.

O `Spring Test` disponibiliza o `TestContext` que oferece diversos recursos entre eles `Transaction Management`, que é responsavel por gerenciamento de transações em classes e metodos de testes.

Quando criamos uma classe de Teste, as vezes desejamos que nossos testes participem da mesma transação que o nosso **``controller``**, para estes casos o `Spring Test` disponibiliza através de sua `API` maneiras para você inicie e encerre transações gerenciadas pelo scopo de testes. 

## Habilitando e Desabilitando Transações.

Por padrão o `Spring Test` não habilita o controle transacional para suas classes ou metodos de teste. Para que um metodo anotado com `@Test` seja executado  em uma transação, devemos anotar `@Transactional`.

Uma forma de verificar se o metodo de teste, realmente possue uma transação ativa é propagar a persistência de um objeto, através do metodo `entityManager.persist()` que exige que uma transação esteja aberta. Veja o codigo abaixo.


```java
@Test
@Transactional
void deveCadastrarUmAlbum() {
    Album album = new Album("Imagens Feriado Dia de Ação de Graças");

    manager.persist(album);

    

    assertNotNull(album.getId());
}
```

Se observamos o resultado deste testes no **console**, concluimos que foi gerado o **SQL** referente a inserção no banco de dados, porém a mudança não foi propagada no banco de dados, porque a transação foi revertida. 

<pre>
<mark><b>2022-04-26 15:28:39.959  INFO 16248 --- [           main] o.s.t.c.transaction.TransactionContext   : Began transaction (1) for test context  transaction manager [org.springframework.orm.jpa.JpaTransactionManager@3f6a9ba0]; rollback [true]</b></mark>
Hibernate: 
    insert 
    into
        album
        (id, nome) 
    values
        (default, ?)
<mark><b>2022-04-26 15:28:40.309  INFO 16248 --- [           main] o.s.t.c.transaction.TransactionContext   : Rolled back transaction for test: </b></mark>
</pre>

A persistencia do album não é confirmada, porque quando qualquer operação **SQL** é executada em uma transação gerenciada por testes, no termino do metodo a transação é **`revertida (rollback)`**. Este comportamento é dado para que  o desenvolvedor não tem que se preocupar em efetuar a limpeza dos dados no banco.

## Como desabilitamos uma transação a unico metodo ou para todos metodos da classe?

Para definir de maneira explicita que um metodo ou todos os metodos de uma classe de testes, não perteceram a transações, utilizamos a anotação `@Trasactional(Transactional.TxType.NEVER)`. 

de limpar o banco.  Metodos anotados com `@BeforeEach` ou ` @AfterEach`, também são executados em transações gereciadas pelo `TestContext`.

OBS: Quando uma classe é anotada com  `@Trasactional(Transactional.TxType.NEVER)`, significa que todos os metodo não serão executados em transações gerenciada pelo `TextContext`.


Para entender melhor, observe o mesmo testes de inserção de um album, utilizando o `EntityManager`.


```java
@Test
@Transactional(Transactional.TxType.NEVER)
void deveCadastrarUmAlbum() {
    Album album = new Album("Imagens Feriado Dia de Ação de Graças");

    manager.persist(album);

    

    assertNotNull(album.getId());
}
```

Observe o que é exibido no console ao executar este testes. 


```java
javax.persistence.TransactionRequiredException: No EntityManager with actual transaction available for current thread - cannot reliably process 'persist' call
```
É dado uma exceção pois a operação  `entityManager.persist()` exige que uma transação seja aberta!


## Confirmando ou Revertendo Transações de maneira explicita com Spring Test

Quando anotamos nossa classe ou metodos com `@Transactional` sabemos que por padrão a transação é revertida, porém, existem casos e estratégias que se torna necessário que o **`commit`** seja feito, e para estes casos podemos utilizar as anotações `@Commit` e `@Rollback`, que definem de maneira explicita quando sera feito o ``commit`` ou ``rollback``. 

OBS: As anotações `@Commit`  e `@Rollback` podem ser utilizadas a nivel de classe ou metodo.

Veja o exemplo abaixo, de como utilizar as anotações em seus testes. 

```java
@SpringBootTest
class CadastrarAlbumControllerTest { 
    @Autowired
    private AlbumRepository repository;

    @Test
    @Commit
    void deveCadastrarUmAlbum(){
        Album novoAlbum = new Album("Imagens Feriado Dia de Ação de Graças");
        
        repository.save(novoAlbum);

        assertEquals(1,repository.findAll().size());
    }

    @Test
    @Rollback
    void deveAtualizarONomeDoAlbum(){
        Album album = repository.findByNome("Imagens Feriado Dia de Ação de Graças");

        album.setNome("Feriado Ação de Graças");
        
        repository.save(novoAlbum);

        assertTrue(
            repository.findByNome("Feriado Ação de Graças")
        .isPresent()
        );
    }
}
```

Existem casos onde é necessário executar uma confirmação de estado do Banco de Dados, antes ou depois que uma transação seja executada, nestes casos o  Spring Test oferece  as anotações `@BeforeTransaction e @AfterTransaction`.

 Abaixo estará um codigo com todas as anotações referentes a transações com Spring Test.


```java 

@Transactional
@SpringBootTest
class CadastrarAlbumControllerTest { 
    @Autowired
    private AlbumRepository repository;

    @BeforeEach
    void setUp(){
        //logica que deve ser executada antes dos testes em uma transação
    }

    @BeforeTransaction
    void setUpTransaction(){
        //logica que deve ser executada antes dos testes,  antes de uma transação
    }
    
    @AfterTransaction
    void tearDownTransaction(){
        //logica que deve ser executada apos dos testes,  apos  uma transação
    }

    @AfterEach
    void tearDown(){
        //logica que deve ser executada após os testes em uma transação
    }



    @Test
    @Commit
    void deveCadastrarUmAlbum(){
        Album novoAlbum = new Album("Imagens Feriado Dia de Ação de Graças");
        
        repository.save(novoAlbum);

        assertEquals(1,repository.findAll().size());
    }

    @Test
    @Rollback
    void deveAtualizarONomeDoAlbum(){
        Album album = repository.findByNome("Imagens Feriado Dia de Ação de Graças");

        album.setNome("Feriado Ação de Graças");
        
        repository.save(novoAlbum);

        assertTrue(
            repository.findByNome("Feriado Ação de Graças")
        .isPresent()
        );
    }
}
```

## Para aprender mais sobre o assunto, recomendamos os seguintes links: 

- [Transaction Management](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-tx)

