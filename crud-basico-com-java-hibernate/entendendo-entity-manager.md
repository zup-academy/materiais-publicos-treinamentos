# Desmistificando o  EntityManager

Ao utilizar a JPA/Hibernate o objetivo  do desenvolvedor é se distanciar da escrita de operações SQL, e começar a manipular objetos que fazem parte do modelo de dominio, ou seja, entidades. Cada entidade possui seus relacionamentos e através de mapeamentos conseguimos propagar operações para estes.

A `JPA` disponibiliza o `EntityManager` que é uma abstração que nós permite realizar operações junto ao Banco de Dados (BD) a nivel de objetos, para que as estas operações sejam propagadas ao BD, é necessário que a entidade esteja no estado gerenciado `(managed)`. Abaixo você encontrará a descrição do estados a qual uma entidade pode ocupar:

- `Transient`: É quando um objeto acaba de ser criado porém não foi associado ao `Contexto de Persistência`, isso quer dizer que ele não esta mapeado a nenhuma lima da tabela do BD, ou seja, ainda não possui chave primaria. 

- `Managed`: É quando a entidade esta associada a alguma linha da tabela ao banco de dados, ou seja, ela possui um ID. Quando a entidades esta neste estado ela é gerenciado pelo `Contexto de Persistência`, e qualquer alteração feita nesta entidade o proprio Hibernate identifica a operação SQL que deve ser disparada ao BD.

- `Detached`: É quando o `Contexto de Persistência` é encerrado, e as determinadas entidades não estão mais em sincronia com BD, isso quer dizer que qualquer alteração feita posteriormente não sera propagada ao BD.

- `Merge`: A operação de Junção copia o estado da entidade desanexada (origem) para uma instância da entidade (destino) gerenciada pelo `Contexto de Persistência`. Caso a entidade mergeada não estiver contida no `Contexto de Persistência` é realizada uma consulta ao BD.

- `Removed`: Dado que uma entidade é removida, é feito uma operação de `DELETE` no BD.

Para a entidade transite entre os estados, devemos utilizar os metodos do `EntityManager`, veja o diagrama abaixo: 

![Entity State](./imagens/stateLifeCicle.png)

## Entendendo as Operações do `EntityManager`

No decorrer da nossa experiência com JPA/Hibernate aprendemos utilizar o metodo `persist()` quando desejamos inserir as informações do objeto a uma linha na tabela do BD. Já o metodo  `merge() ` associamos a realizar alterações a determinada entidade, e por fim o metodo `remove()` quando desejamos deletar um registro de nossa base. Este conhecimento por si não esta errado, porém existe alguns detalhes sobre os metodos `persist` e `merge` que passam despercebidos e vamos explorar os mesmos agora.

## Como funciona o `EntityManager.persist()` 

A responsabilidade deste metodo é colocar uma instância da entidade no estado `Managed`, isso significa que dado que exista uma transação aberta, a JPA/Hibernate ira propagar o SQL necessário ao banco. 

Se observamos o contrato deste metodo ele recebe uma referência a entidade, e não possui retorno. Isto quer dizer que as informações que são alteradas dentro da transação são alteradas em memoria no objeto de referência.

Para enteder melhor o que acontece, imagine que existe uma funcionalidade para cadastrar uma entidade Livro, mapeada abaixo:

```java
@Entity
public class Livro{
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(nullable=false)
    private String titulo;
    
    public Livro(String titulo){
        this.titulo=titulo;
    }

    @Deprecated
    public Livro(){

    }

    public Long getId(){
        return this.id;
    }
}
```

Agora desejamos vincular uma instância deste objeto a uma linha do banco de dados, primeiro precisamos nos certificar que exista uma transação aberta. Dado que exista a transação chamaremos o metodo `persist` do `EntityManager`, veja o codigo abaixo.

```java
@Transactional
public void cadastrar(Livro livro){
    entityManager.persist(livro);
} 
```

Neste momento é disparado a seguinte operação SQL:

```sql
    INSERT INTO livro (id,titulo) VALUES (?,?) 
```

é dado um id a esta entidade, e por referência de memoria, o atributo id recebe o valor gerado no BD. Para verificar podemos utilizar o metodo `getId()`, que obteremos o valor. 
Ao utilizar o metodo `EntityManager.contains()` para verificar se a entidade esta presente ao `Contexto de Persistência`. Criamos o seguinte teste para validar.

```java
@SpringBootTest
public class TestandoEntityManagerMetodos{
    @Autowired
    private EntityManager manager;


    @Transactional
    @Test
    public void devePersistirUmLivro(){
        Livro livro = new Livro("Clen Code");
        manager.persist(livro);

        assertNotNull(livro.getId());
        assertTrue(manager.contains(livro));
    }
}

```


## Como funciona o `EntityManager.merge()` 

O comportamento deste é metodo é juntar o estado de dois objetos que fazem referência uma entidade. Dado que existe um objeto em memoria que é solicitado a operação de `merge`, é feito uma verficação se este existe no `Contexto de Persistencia`, caso não exista é feita uma operação SQL de `INSERT`, caso exista, ele procura o Objeto de destino no `Contexto de Persistência` e atualiza suas as informações com objeto em memoria, e retorna a instância existente no `Contexto de Persistência`. 


Para entender melhor observe o teste para o metodo merge.

```java
@SpringBootTest
public class TestandoEntityManagerMetodos{
    @Autowired
    private EntityManager manager;


    @Transactional
    @Test
    public void devePersistirUmLivro(){
        Livro livro = new Livro("Clen Code");
        Livro livroManaged = manager.merge(livro);

        assertNull(livro.getId());
        assertFalse(manager.contains(livro));
        assertNotNull(livroManaged.getId());
        assertTrue(manager.contains(livroManaged));
    }
}
```

Podemos observar que o objeto livro não esta presente no `Contexto de Persistência` e nem que seu id esta preenchido. Já o livroManaged contém um id e esta presente no contexto.


## Como funciona a propagação em Cascasta dos Metodos Persistence e Merge.

A JPA/Hibernate oferece atráves de mapeamento a possibilidade de propagar em cascata as operações do `EntityManager` para os relacionamentos de uma entidade.
Para tirar proveito deste recurso em um determinado relacionamento devemos junto a anotação de cardinalidade, definir as operações no argumento cascade. Veja um exemplo abaixo.


```java
@Entity
public class Livro{
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(nullable=false)
    private String titulo;


    @OneToMany(mappedBy = "livro", cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    private List<Capitulo> capitulos = new ArrayList<>();

    public Livro(String titulo){
        this.titulo=titulo;
    }

    @Deprecated
    public Livro(){

    }

    public void adicionar(Capitulo capitulo){
        capitulo.setLivro(this);
        this.capitulos.add(capitulo);
    }

    public Long getId(){
        return this.id;
    }
}
```

Para entender o comportamento das propagações, vamos inserir um livro com um capitulo. e aplicar os metodos `persist` e `merge`.
A entidade Capitulo esta mapeada abaixo.


### Propagando operações em relacionamentos utilizando o `EntityManager.merge()`

```java
@Entity
public class Capitulo{
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(nullable = false)
    private String nome;

    @ManyToOne(optional = false)
    private Livro livro;

    
    public Capitulo(String nome){
        this.nome=nome;
    }

    @Deprecated
    public Capitulo(){

    }

    public Long getId(){
        return this.id;
    }

    public void setLivro(Livro livro){
        this.livro=livro;
    }

}
```

Para verficar o comportamento do metodo `merge`, observamos o caso de teste abaixo, dado que existe um Livro no estado `Managed`,  o que acontece ao adicionar um Capitulo que esta no estado `Transient` a ele.

```java
@Test
@Transactional
public void devePropagarOperacaoMerge(){

    Livro livro = manager.find(Livro.class, 1L);

    Capitulo capitulo = new Capitulo("Dependency Inversion");

    livro.adicionar(capitulo);

    manager.merge(livro);


    assertNull(capitulo.getId());
    assertNotNull(livro.getCapitulos().get(0).getId());
    assertNotSame(capitulo,livro.getCapitulos().get(0));
    assertTrue(manager.contains(livro.getCapitulos().get(0)));
    assertFalse(manager.contains(capitulo));

}
```

Observamos que o objeto capitulo não tem sua referência atualizada, porém se percorremos os capitulos de um livro, encontramos a referência atualizada, porque no relacionamento foi preenchido com a instancia que foi anexada ao `Contexto de Persistência`.
Também observamos que as instâncias dos capitulos não são iguais, e o objeto capitulo não presente no `Contexto de Persistência`.

### Propagando operações em relacionamentos utilizando o `EntityManager.persist()`


```java
@Test
@Transactional
public void devePropagarOperacaoPersist(){

    Livro livro = manager.find(Livro.class, 1L);

    Capitulo capitulo = new Capitulo("Dependency Inversion");

    livro.adicionar(capitulo);

    manager.persist(livro);


    assertNotNull(capitulo.getId());
    assertNotNull(livro.getCapitulos().get(0).getId());
    assertSame(capitulo,livro.getCapitulos().get(0));
    assertTrue(manager.contains(livro.getCapitulos().get(0)));
    assertTrue(manager.contains(capitulo));

}
```

Já quando aplicamos o metodo `persist` observamos que o objeto livro tem um id atributo a ele, e que sua instancia é a mesma que esta presente na coleção, e que agora ela é gerenciada pelo `Contexto de Persistência`.

## Porém se desejamos que a JPA/Hibernate identifique qual operação usar para sincronizar com banco o que acontece?
Quando estamos realizando mudanças em uma entidade dentro de um escopo Transacional e queremos que a JPA/Hibernate, identifique as mudanças e imediatamente propague as mudanças ao BD, podemos utilizar o metodo `flush()` do EntityManager. Este metodo disparará um mecanismo de checkagem de operações e por si decidirá se propagara o metodo `merge()` ou `persist()`.

```java
@Test
@Transactional
public void deveSincronizarComBanco(){

    Livro livro = manager.find(Livro.class, 1L);

    Capitulo capitulo = new Capitulo("Dependency Inversion");

    livro.adicionar(capitulo);

    manager.flush();

    assertNotNull(capitulo.getId());
    assertNotNull(livro.getCapitulos().get(0).getId());
    assertSame(capitulo,livro.getCapitulos().get(0));
    assertTrue(manager.contains(livro.getCapitulos().get(0)));
    assertTrue(manager.contains(capitulo));

}
```
No caso acima a JPA/Hibernate identifica que a operação de `persist` deve ser propagada, e dado a isso, temos nossa referência atualizada. Isso quer dizer que o objeto capitulo agora é gerenciado, e que tem o id atribuito.

## E quando falamos de Spring Data JPA, como funciona os metodos `save()` e `saveAndFlush()`

Entender o comportamento dos metodos `save` e `saveAndFlush` nos permite extrair o maximo deles em nosso dia a dia. Abaixo veremos como cada um funciona.

  
- `save()`: primeiro é feito uma verificação se a entidade esta no estado `Transient`, caso esteja é feito uma chamada do metodo `entityManager.persist()`. Caso contrario é feito uma chamada do metodo `entityManager.merge()`.

- `saveAndFlush()`: primeiramente é chamado o metodo save, e após é feito a chamada do metodo `entityManager.flush()` que esta encapsulado no metodo `flush()` do repository.


E se uma entidade filha em estado **`Transient`**  é adicionada a uma entidade pai que esta no estado **`Managed`**, onde tem-se uma propagação em cascata de **`PERSIST`** e/ou **`MERGE`**, e a entidade pai recebe a chamada do metodo `save()` ou `saveAndFlush()`, a JPA/Hibernate identifica que a operação merge deve ser propagada, e quando isso é feito a referência em memoria do objeto da entidade filha não é atualizado. veja o exemplo abaixo.

```java
@Transactional
public void adicionarCapituloALivro(Long idLivro){
    Livro livro = repository.findById(idLivro)
                        .orElseThrow(() -> new LivroNaoExistenteException("Livro não cadastrado"));

    Capitulo capitulo = new Capitulo("Segregação de Interface");

    livro.adicionar(capitulo);

    repository.saveAndFlush(livro);

    system.out.println("id capitulo: "+ capitulo.getId());
    
} 
```

recebemos no console a seguinte mensagem.

```
id capitulo: null
```

E para que a gente consiga obter este id, devemos solicitar que o mecanismo de checagem da JPA/Hibernate decida qual operação propagar quando pedimos para que sincronize ao banco as mudanças. Então substituimos a chamada do metodo  `saveAndFlush()` pelo `flush()`.


```java
@Transactional
public void adicionarCapituloALivro(Long idLivro){
    Livro livro = repository.findById(idLivro)
                        .orElseThrow(() -> new LivroNaoExistenteException("Livro não cadastrado"));

    Capitulo capitulo = new Capitulo("Segregação de Interface");

    livro.adicionar(capitulo);

    repository.flush();

    system.out.println("id capitulo: "+ capitulo.getId());
    
} 
```

recebemos no console a seguinte mensagem.

```
id capitulo: 1
```

Podemos concluir que para casos onde precisamos que uma persistência seja feita antes do termino da transação é mais indicado que deixamos qua a JPA/Hibernate identifiquem qual operação seja propagada.