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

    List<Album> all = repository.findAll();
    int size = all.size();

    assertEquals(1, size);
}
```

Se observamos o resultado deste testes no **console**

<pre>
<mark>2022-04-26 15:28:39.959  INFO 16248 --- [           main] o.s.t.c.transaction.TransactionContext   : Began transaction (1) for test context  transaction manager [org.springframework.orm.jpa.JpaTransactionManager@3f6a9ba0]; rollback [true]</mark>

<span style="background-color: lightyellow; color:black"> HIBERNATE </span>
Hibernate: 
    insert 
    into
        album
        (id, nome) 
    values
        (default, ?)
Hibernate: 
    select
        album0_.id as id1_0_,
        album0_.nome as nome2_0_ 
    from
        album album0_

<mark><b>2022-04-26 15:28:40.309  INFO 16248 --- [           main] o.s.t.c.transaction.TransactionContext   : Rolled back transaction for test: </b></mark>
</pre>

 que ao termino do metodo é **`revertida (rollback)`**, ou seja o desenvolvedor não tem que se preocupar de limpar o banco.  Metodos anotados com `@BeforeEach` ou ` @AfterEach`, também são executados em transações gereciadas pelo `TestContext`.

OBS: caso deseje definir de maneira explicita uma transação a um metodo, basta anota-lo com `@Trasactional`. Quando uma classe é anotada com  `@Trasactional`, cada metodo é executado em uma transação gerenciada pelo `TextContext`.


Para entender melhor, temos um teste de inserção de uma entidade com Repository da JPA, observe o codigo abaixo.


```java
@SpringBootTest
class CadastrarAlbumControllerTest { 
    @Autowired
    private AlbumRepository repository;

    @Test
    void deveCadastrarUmAlbum(){
        Album novoAlbum = new Album("Imagens Feriado Dia de Ação de Graças");
        
        repository.save(novoAlbum);

        assertEquals(1,repository.findAll().size());
    }
}
```

Ao final do metodo `deveCadastrarUmAlbum` todas as alterações são revertidas, ou seja, o comando de `insert` não é confirmado, por tanto o estado do banco é preservado.  Este comportamento é perigoso ao utilizar um framework de persistencia, como Hibernate, pois as exceções disparadas dado as erros de constraints, só acontece junto ao `commit`, que por padrão não é realizado, então podemos ficar reféns de falsos positivos.

Para estes casos é possível definir a nivel de classe ou metodo quando uma transação sera confirmada ou revertida de maneira declarativa, através das anotações `@Commit`  e `@Rollback`. 

Veja o exemplo abaixo: 

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

