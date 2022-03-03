# Atualização de Recursos de uma API REST com Spring 

Não é de hoje que definir um designe de um `endpoint` para atualização de uma API REST é assunto para duvidas. Tarefas como definir o verbo HTTP, Status HTTP de retorno e até como definir as resposabilidades do codigo estão altamente relacionadas a interpretação das documentações oficiais de cada Metodo HTTP, e até descisões de designer de codigo entre equipes. Dado que cada pessoa pode interpretar determinada informações de diferentes formas, torna-se complexo manter um padrão universal.

## Atualização Completa Vs Atualização Parcial

Hoje no protocolo HTTP existem verbos especificos para atualiazações que contemplem toda representação de um recurso ou parte dela. Quando falamos de API REST os tipos de atualização são diferenciados pelos verbos `PUT` e `PATCH`. `PUT` quando vamos alterar todos campos de um recurso, e  `PATCH` para quando iremos alterar alguns campos. 

Abaixo tem-se a representação de um recurso Pessoa,a representação esta em JSON.


```json
{
    "id": 1,
    "nome":"Matheus Souza Alegre",
    "email": "matheus.alegre@email.com"
}
```

Para melhor entendimento das atualizações, veja abaixo o que tange cada uma delas.

### Atualização Completa (PUT)

Ao direcionar nossa energia para implementar um endpoint de atualização, o primeiro passo a se observar é a quais informações desejamos atualizar em determinado recurso. Quando definimos que iremos atualizar todas as informações é o que chamamos de atualização completa, estas são representas pelo verbo PUT. Veja abaixo o exemplo de uma atualização.


Aqui temos um recurso Pessoa, que representa as informações de id, nome e email.

```json
{
    "id":1,
    "nome":"Matheus Souza Alegre",
    "email": "matheus.alegre@email.com"
}
```

Quando iremos enviamos uma solicitação para alteração do nome e email para `Matheus Alegre` e `m.alegre@email.com` respectivamente
, estamos alterando toda a representação deste recurso. Para fazer essa solicitação utilizaremos o verbo `PUT`  para o seguinte endereco "/pessoas/1", veja como seria o corpo da requisição.

```json

{

    "id":1,
    "nome":"Matheus Alegre",
    "email": "m.alegre@email.com"
}
```

### Atualização Parcial (PATCH)

Como o proprio nome já diz, é feito uma alteração em uma fração de uma representação de um recurso. Olhando para o exemplo anterior, onde existe um recurso pessoa que contém um id, nome e um email, caso seja alterado apenas o nome ou email, estaremos atualizando parcialmente determinada representação. Veja o exemplo abaixo.

Aqui temos a representação do nosso recurso.

```json
{
    "id": 2,
    "nome":"Adriano Ferreira Moraes",
    "email": "adriano.moraes@email.com"
}
```

Para submeter uma atualização do email, seria feito uma requisição do tipo `PATCH` para o endereco "/pessoas/2" com o corpo abaixo.



```json
{
    "email": "adriano.ferreira@email.com"
}
```

Após a alteração do email, para `adriano.ferreira@email.com` obtemos a seguinte representação.

```json
{
    "id": 2,
    "nome":"Adriano Ferreira Moraes",
    "email":"adriano.ferreira@email.com"
}
```

## PUT vs PATCH

Após entender quais tipos de atualizações podemos implementar, devemos nos atentar a semântica de cada verbo, ou seja, dado aos verbos `PUT` e `PATCH` HTTP qual tipo de atualização cada um representa?

- `PUT`: É um método que solicita que atráves de uma requisição a determinado recurso, suas informações sejam substituidas caso este exista, caso contrario uma instância pode ser definida como novo rescurso. 

- `PATCH`: É um método que solicita que um conjunto de alterações descritas seja incorporado a uma representação de um recurso já existente.

A principal diferença entre estes metodos é a forma que o servidor processa as alterações. Quando uma solicitação `PUT` sobreescreve todas informações que de uma entidade, o metodo `PATCH` altera apenas um conjuto de informações. 

Existem algumas nuances que ambos os verbos compartilham como, quando as respostas possuem representação do recurso no corpo, o HTTP Status 200 deve ser retornado, ou quando não possue o HTTP Status 204 deve ser informado. 

Para casos de erros de valição no corpo do documento é cabivel um HTTP Status 400. 

Outro fator em comum aos verbos e que ambos acessam um URI de um resource, ou seja, um endereco indentificador unico. Como por exemplo: `"/pessoas/1"`.

Existe também diferenças quando as respostos em casos de erros. o `PATCH` indica que o servidor pode recusar uma modificação que invalide o recurso, neste caso é necessário que a resposta contenha um HTTP Status 422




| VERBO | Status ao Criar um Resource | Status Sucesso | Status Falha |
|-----|-------| -----| ----|
|PUT | 201 | 200 ou 204| 400 |
|PATCH| não implementavel |200 ou 204|400 ou 422|

## Definindo um `endpoint PUT` com Spring

Suponha que nossa funcionalidade não crie uma representação para um recurso inexistente, então devemos retornar na resposta um Status coerente, para este caso informaremos um HTTP Status 404. 

Como é descrito que o verbo `PUT` acessa um URI, é notavel que receberemos uma variavel para obter o identificador que esta composto ao caminho da requisição. Também é notavel que receberemos todas as informações de um recurso no corpo. Então é importante que exista uma classe que represente esta entrada de dados.

Então chega de conversa e bora para o codigo. Neste exemplo sera construido uma api para atualização completa de uma entidade Pessoa. Veja abaixo.

```java
@RestController
public class AtualizaPessoaController{
    private final PessoaRepository repository;

    public AtualizaPessoaController(PessoaRepository repository){
        this.repository=repository;
    }

    @PutMapping("/pessoas/{id}")
    public ResponseEntity<Void> atualizar(@RequestBody @Valid AtualizaPessoaRequest request, @PathVariable Long id){
       Pessoa pessoa = repository.findById(id).orElseThrow(()-> new ResponseStatusException(HttpStatus.NOT_FOUND));

       pessoa.atualiza(request.getNome(),request.getEmail());

       repository.save(pessoa);

       return ResponseEntity.noContent().build();
    }
}
```

```java
    public class AtualizaPessoaRequest{
        @NotBlank
        private String nome;

        @NotBlank
        private String email;


        public String getNome(){
            return this.nome;
        }

        public String getEmail(){
            return this.email;
        }

    
    }
```

```java
    @Entity
    public class Pessoa{
        @Id
        @GeneratedValue(strategy=IDENTITY)
        private Long id;
        private String nome;
        private String email;


        public void atualiza(String nome, String email){
            this.nome=nome;
            this.email=email;
        }

    
    }
```
```java
    public interface PessoaRepository extends JpaRepository<Pessoa,Long>{

    }
```


## Implementando um `endpoint PATCH` com Spring

caso queiramos atualizar apenas o email de uma pessoa, escreveriamos um codigo como o abaixo.

```java
@RestController
public class AtualizaEmailPessoaController{
    private final PessoaRepository repository;

    public AtualizaEmailPessoaController(PessoaRepository repository){
        this.repository=repository;
    }

    @PatchMapping("/pessoas/{id}")
    public ResponseEntity<Void> atualizar(@RequestBody @Valid AtualizaEmailPessoaRequest request, @PathVariable Long id){
       Pessoa pessoa = repository.findById(id).orElseThrow(()-> new ResponseStatusException(HttpStatus.NOT_FOUND));

       pessoa.setEmail(request.getEmail())

       repository.save(pessoa);

       return ResponseEntity.noContent().build();
    }
}
```

```java
    public class AtualizaEmailPessoaRequest{
       
        @NotBlank
        private String email;

        public String getEmail(){
            return this.email;
        }

    
    }
```

## Link para aprofundamento

Como existem fontes complexas e simples recomendamos que use a que fizer mais sentido para você. 



|VERBO|RFC|MOZZILA DEV DOC|
|-|-|-|
|PUT|[RFC](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.3)|[Mozzila Doc](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Methods/PUT)|
|PATCH|[RFC](https://datatracker.ietf.org/doc/html/rfc5789)|[Mozzila Doc](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Methods/PATCH)|

- [Diferenças entre HTTP PUT e HTTP PATCH com Spring](https://www.baeldung.com/http-put-patch-difference-spring)