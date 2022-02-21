# Implementando API REST para deleção de registros

Nesse conteúdo veremos como podemos implementar uma API REST responsável por deletar um registro do sistema. Vamos partir da construção da API desde a escrita do controller via módulo Spring Web até a persistência com Spring Data, JPA e Hibernate.

## Deletando um livro do sistema

Por se tratar de uma deleção, é muito comum que boa parte do código e estrutura já exista no sistema. Portanto, imagine que tenhamos um sistema de livraria virtual, na qual possui uma classe que representa um livro dentro do sistema. O código da classe `Livro` mapeado com anotações da JPA pode ser visto abaixo:

```java
@Entity
class Livro {

    @Id
    @GeneratedValue
    private Long id;

    private String titulo;
    private String descricao;

    // outros atributos, construtores, getters e setters
}
```

E claro, se vamos deletar uma entidade é porque há grandes chances de já existe um Repository para ela, afinal só podemos deletar algo do banco de dados se este tiver sido inserido antes. O código da interface `LivroRepository` seria em simples com graças ao Spring Data JPA:

```java
@Repository
public class LivroRepository extends JPARepository<Livro, Long> {

}
```



## Implementando um controller para deleção

Quando falamos de deleção de algum recurso numa API REST estamos falando do uso do verbo HTTP `DELETE`. Além do verbo ou método HTTP, também precisamos de uma URI que identifique o recurso de forma única na qual queremos deletar dentro do sistema, que geralmente segue o formato `/nome-do-recurso-no-plural/id`. No caso de um livro, poderíamos ter uma URI como `/api/livros/12345` por exemplo. 

Outro detalhe é que não necessidade de enviar um payload na requisição, o que torna a implementação ainda mais simples. Portanto, para representar esta API REST com Spring Boot, basta implementarmos uma classe de controller como abaixo:

```java
@RestController
public class RemoveLivroController {

    @DeleteMapping("/api/livros/{id}")
    public ResponseEntity<?> remove(@PathVariable("id") Long livroId) {

        return null;
    }
}
```

Repare que usamos a anotação `@DeleteMapping` para indicar que se trata do verbo HTTP `DELETE`; ainda na anotação, indicamos a URI `/api/livros/{id}` na qual denota o endereço único deste recurso, porém estamos declarando que o `{id}` dessa URI será dinâmico e que deve ser extraído pelo Spring Boot. Para extrair esse `{id}` basta declarar um parâmetro no método do controller e anotá-lo com `@PathVariable("id")`, onde "id" é o nome da variável entre chaves (`{varName}`) na URI.

O próximo passo é implementar a lógica para deletar a entidade do banco de dados usando o `id` indicado na URI e extraido para parâmetro `livroId` do método. Para isso, primeiramente, precisamos injetar o repository no controller e em seguida carregar a entidade do banco de dados:

```java
@RestController
public class RemoveLivroController {

    @Autowired
    private LivroRepository repository;

    @DeleteMapping("/api/livros/{id}")
    public ResponseEntity<?> remove(@PathVariable("id") Long livroId) {

        Livro livro = repository.findById(livroId).orElseThrow(() -> {
            return throw new ResponseStatusException(HttpStatus.NOT_FOUND, "livro não encontrado")
        });

        // ...

        return null;
    }
}
```

Repare que além de carregar o livro do banco de dados nós também já verificamos sua existência: caso ela não exista nós lançamos a exceção `ResponseStatusException` com status HTTP `NOT_FOUND` indicando para o cliente da nossa API REST que o recurso (`id` indicado na URI) não foi encontrado. 

Dado que a entidade exista no banco de dados, basta agora invocar o método `delete` do repository passando a instância de `Livro` encontrada:

```java
@RestController
public class RemoveLivroController {

    @Autowired
    private LivroRepository repository;

    @DeleteMapping("/api/livros/{id}")
    public ResponseEntity<?> remove(@PathVariable("id") Long livroId) {

        Livro livro = repository.findById(livroId).orElseThrow(() -> {
            return throw new ResponseStatusException(HttpStatus.NOT_FOUND, "livro não encontrado")
        });

        repository.delete(livro);

        return null;
    }
}
```

O código acima seria suficiente para remover a entidade do banco de dados, mas ainda precisamos informar ao cliente que a entidade foi removida com sucesso através de um status HTTP. No caso do método `DELETE`, temos [alguns possíveis status de sucesso](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/DELETE):

- `OK (200)`: a entidade foi deletada e temos corpo na resposta: algum payload ou representação descrevendo o status;
- `No Content (204)`: a entidade foi deletada e não temos corpo na resposta;
- `Accepted (202)`: a entidade não foi deletada imediatamente mas será deletada em breve (geralmente processamento assincrono);

No nosso caso, não pretendemos retornar um payload de resposta para o cliente. Nós poderíamos até retornar um status HTTP `OK` sem um payload na resposta, não estaria errado, mas semanticamente retornar um `No Content` seria o mais apropriado para o protocolo HTTP. Portanto, vamos alterar o método do nosso controller como a seguir:

```java
@RestController
public class RemoveLivroController {

    @Autowired
    private LivroRepository repository;

    @DeleteMapping("/api/livros/{id}")
    public ResponseEntity<?> remove(@PathVariable("id") Long livroId) {

        Livro livro = repository.findById(livroId).orElseThrow(() -> {
            return throw new ResponseStatusException(HttpStatus.NOT_FOUND, "livro não encontrado")
        });

        repository.delete(livro);

        return ResponseEntity
                    .noContent().build(); // HTTP 204
    }
}
```

O código acima está pronto e pode ser utilizado em produção. Mas é sempre bom ter cuidado com a transação do banco de dados.

## Não esqueça de delimitar a transação do banco

Não menos importante, por estarmos lidando com comandos de escrita do banco de dados, precisamos **delimitar onde começa e termina uma transação com o banco de dados**: na maioria dos casos a transação do banco está alinhada com a transação de negócio, não à toa podemos encara-las como a mesma. Dessa forma, dado que a nossa lógica de negócio é a implementação do método `remove` do controller, vamos anotá-lo com a anotação `@Transactional` do Spring como a abaixo:


```java
@RestController
public class RemoveLivroController {

    @Autowired
    private LivroRepository repository;

    @Transactional // delimita transação do banco
    @DeleteMapping("/api/livros/{id}")
    public ResponseEntity<?> remove(@PathVariable("id") Long livroId) {

        Livro livro = repository.findById(livroId).orElseThrow(() -> {
            return throw new ResponseStatusException(HttpStatus.NOT_FOUND, "livro não encontrado")
        });

        repository.delete(livro);

        return ResponseEntity
                    .noContent().build(); // HTTP 204
    }
}
```

Uma transação será iniciada na invocação do método. Se o método terminar sem erro o Spring se encarrega de comitar a transação e devolver a conexão para o pool de conexões; caso contrário, se alguma exceção for lançada o Spring efetua um rollback e libera a conexão para o pool.

## Consumindo a API REST

Por fim, para consumir a API REST podemos usar algum cliente HTTP que nós já conhecemos, como POSTman, Insomnia ou mesmo cURL. Por exemplo, para remover um livro do sistema usando o bom e velho cURL via linha da comando, podemos fazer algo como:

```shell
curl -i -XDELETE http://localhost:8080/api/livros/4

HTTP/1.1 204
Date: Mon, 21 Feb 2022 14:05:27 GMT
```

Como podem ver, recebemos uma resposta HTTP com status `No Content (204)`. Caso tentemos deletar uma entidade que não existe ou que foi deletada anteriormente teremos uma resposta diferente:
```shell
curl -i -XDELETE http://localhost:8080/api/livros/999999

HTTP/1.1 404
Content-Type: application/json
Transfer-Encoding: chunked
Date: Mon, 21 Feb 2022 14:05:20 GMT

{ 
    "timestamp": "2022-02-21T14:05:20.797+00:00",
    "status": 404,
    "error": "Not Found",
    "message": "livro não encontrado",
    "path":"/api/livro/999999"
}
```

Pronto! Acabamos de implementar nossa API REST para deleção de uma entidade no sistema. Não foi tão dificil assim, não é mesmo?

Como podemos ver, geralmente implementar a deleção de uma entidade será mais simples do que o cadastro ou atualização de uma entidade, pois toda a estrutura de classes e mapeamento estará pronta, restando para o desenvolvedor(a) somente a implementação da API REST e algumas possíveis validações.
