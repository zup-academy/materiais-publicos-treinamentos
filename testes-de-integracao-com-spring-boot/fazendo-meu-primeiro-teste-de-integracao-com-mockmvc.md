# Construindo Testes de Integração com Mockmvc do Spring Test

Nos como desenvolvedores buscamos cada vez mais construir software se maneira mais produtiva e segura, durante este processo nos esbarramos em diversas metodologias, como o uso de frameworks como Spring e Spring Boot que oferecem diversos modulos e configurações que permitem que o desenvolverdor se restrinja a codificar o que gere mais valor ao cliente. Já pelo lado da segurança do que foi implementado é possivél aplicar testes que revelem bugs e que garantam que a funcionalidade é executada de acordo com o projetado.

Ao definir um conjuto de testes para determinado sistema, o desenvolvedor olha sempre para duas dimensões, os testes de unidade, que focam em verificar que uma unidade de codigo, funciona corretamente apartir de determinado cenario. E os Testes de Integração que tem como responsabilidade garantir que as unidades trabalham corretamente quando executam juntas.

Olhando para um Sistema Web no padrão MVC ou uma API REST na hora de produzir testes de integração devemos nos atentar em garantir que não apenas as classes escritas funcionem como esperado, mas que também o serviço esteja diponivel em determinado endereço junto a semântica de um verbo `HTTP`. E que se seja feito alguma operação junto a um Banco de Dados (BD), a mesma funcioe de acordo com o esperado.

## Testando um Serviço de Cadastro de Pessoas

Abaixo você encontrará uma API REST que é responsavel por cadastrar informações de uma Pessoa.


```java
public class PessoaRequest{
    @NotBlank
    private String nome;

    @NotBlank
    @CPF(message="CPF deve estar no formato valido")
    private String cpf;

    //construtor, getters e setters otimitos
}




@RestController
@RequestMapping("/api/pessoas")
public class CadastrarPessoaController{
    private final PessoaRepository repository;

    public CadastrarPessoaController(PessoaRepository repository){
        this.repository=repository;
    }

    @PostMapping
    public ResponseEntity<?> cadastrar(@RequestBody @Valid PessoaRequest request, UriComponentsBuilder uriComponentsBuilder){
        Pessoa pessoa = request.paraPessoa();

        repository.save(pessoa);

        URI location = uriComponentsBuilder.path("/api/pessoas/{id}")
            .buildExpand(pessoa.getId())
            .toUri();

        return ResponseEntity.created(location).build();
    }
}
```

Olhando para esta classe, podemos inferir o seguinte: 

1. A funcionalidade de cadastrar esta disponivel no endereco: `/api/pessoas`, através do verbo `POST HTTP`.

2. A funcionalidade de cadastrar retorna um header HTTP chamado location com a URI do recurso que foi criado, e a resposta contem um verbo `HTTP Created`.

3. Quando um recurso é criado é inserido um registro no banco de dados com as informações do mesmo.

4. Caso aconteça erro de validações, teremos mensagens personalizadas, e por padrão um verbo `HTTP Bad Request`.


Com as informações apresentadas acima podemos decidir que um teste de integração é o cenario correto a se aplicar, e para que a gente consiga fazer acesso a camada Web precisaremos de um Cliente HTTP e que o `Application Context` esteja de pé para fazer uso do servidor.

//falar sobre o que é o mock mvc como configuralo e como criar um teste de integracao com spring test.