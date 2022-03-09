# Consultas em API REST com Spring Boot

Uma função de extrema utilidade e valor para dentro de um software são os relatorios, eles permitem ver  o progresso de uso dos clientes dentro do sistema. Imagine um contexto escolar onde todos os anos acontece o periodo de matricula e re-matricula para os alunos. A secretaria que é um usuario do sistema deve sempre cadastrar todas informações de um aluno que nunca fez parte da escola, e revalidar a matricula do aluno que já faz parte, poder realizar consulta dos dados deste aluno, e apenas incrementar a matricula do ano, certamente aumenta sua produtividade. E o aluno poder consultar todas as notas em determinada disciplina da para ele clareza para se preparar para proxima avaliação.

## Consultas em Sistemas e `APIs REST`

Realizar consultas em sistemas é algo que normalmente não exige muito esforço do desenvolvedor, dado que exista uma base de dados em memoria, arquivos de texto, consultas externas ou bancos de dados SQL ou noSQL. Um criterio que devemos sempre nos atentar ao determinar funcionalidades de consulta ou relatorios, é que o tempo de resposta não seja alto. Um alto de tempo de resposta pode levar que aos usuarios submetam novamente requisições para executar novamente a funcionalidade, e depêndendo da frequência podemos derrubar o servidor.

### Qual o designe utilizado em `Web Service`s e `APIs REST` que estão disponiveis no protocolo `HTTP`?

Sistemas Web tradicionais normalmente tem suas funcionalidades pautadas nos verbos `POST` e `GET`, as informações de leitura todas atreladas ao `GET`, e as demais ao `POST`. Já quando falamos de `APIs REST` onde cada verbo `HTTP` define uma sematica para uma operação, o verbo utilizado é o `GET` e por definição as requisições não possuem corpo `(body)` e as informações para definir paginação, e parâmetros são dadas através de parâmetros ou `Query String`. As princiais caracteristicas do verbo `GET` são: 
- É permitido que suas respostas sejam armazenadas em cache.
- As requisições não causam efeitos colaterais no servidor `(idempotente)`.

## Definindo endpoints para Consulta com Spring

O mapeamento para endpoint com `Spring` não é diferente dos demais verbos, podemos definir através de duas anotações:

- `@RequestMapping(method=RequestMethod.GET, value ="/pessoas")`
- `@GetMapping("/pessoas")`

Veja abaixo um exemplo de um controller para consulta de todos registros de pessoas.

```java
@RestController
@RequestMapping("/pessoas")
public class ConsultarPessoasController{
    private final PessoaRepository repository;

    public ConsultarPessoasController(PessoaRepository repository){
        this.repository=repository;
    }

    @GetMapping
    public ResponseEntity<?> consultar(){
        
        List<PessoaResponse> pessoas = repository.findAll().stream()
        .map(PessoaResponse::new)
        .collect(Collectors.toList());

        return ResponseEntity.ok(pessoas);
    }

}

```

## Links para aprofundamento

- Entendendo o HTTP GET
    - [rfc](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.1)
    - [mozzila developer](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Methods/GET)