# Coesão em Controllers

## Controllers sem coesão

Resumindo bem, Controllers são conversores do mundo 'web' para o mundo das regras de negócios e banco de dados.
Eles são responsáveis por receber as requisições HTTP, converter os dados recebidos para o formato desejado e/ou a
realização de alguma ação.

Olhando pela via mais subjetiva, coesão em controllers se refere à medida em que um controller é focado em uma tarefa
específica.
Mas é comum encontrar longos códigos escritos em controladores de aplicações MVC. Vemos muitas regras de negócio soltas,
muitas responsabilidades e isso leva a nenhuma coesão, dificuldade de manutenção e testes complexos.
Veja o código a seguir:

```java

@RestController
@RequestMapping("/api/v1/usuarios")
public class UsuarioController {

    @Autowired
    private UsuarioRepository usuarioRepository;
    @Autowired
    private ServicoExternoRemoto servicoExternoRemoto;
    @Autowired
    private ServicoNotificacaoExclusao servicoNotificacaoExclusao;

    @PostMapping
    public ResponseEntity<Usuario> cadastrar(@RequestBody Usuario usuario, @RequestBody Endereco endereco) {
        if (usuario.ehDeMinasGerais()) {
            // aplica alguma regra de negócio
        }
        if (usuarioRepository.existsByEmail(usuario.getEmail())) {
            // aplica alguma regra de negócio
        }
        if (usuario.possuiNecessidadesEspecificas()) {
            // aplica alguma regra de negócio
        }
        usuarioRepository.save(usuario);
        servicoExternoRemoto.enviaParaUmServicoExterno(usuario);
        return ResponseEntity.ok(usuario);
    }

    @GetMapping
    public ResponseEntity<List<Usuario>> listar() {
        return ResponseEntity.ok(usuarioRepository.findAll());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletar(@PathVariable Long id) {
        if (usuario.ehInvalido()) {
            // aplica alguma regra de negócio
        }
        if (usuario.possuiEndereco()) {
            // aplica alguma regra de negócio
        }
        servicoNotificacaoExclusao.notifica(usuario);
        usuarioRepository.deleteById(id);
        return ResponseEntity.ok().build();
    }

}
```

Repare que o controller acima possui muitas responsabilidades, como por exemplo, a validação de regras de negócio, todas
soltas nos métodos. A comunicação com serviços externos, a comunicação com o banco de dados e a comunicação com
outros serviços da aplicação.
Esse é um exemplo de CRUD muito encontrado em aplicações Java, mas que não segue o princípio da responsabilidade única
nem olhando pela via subjetiva e nem pela maneira lógica.

A primeira coisa que podemos fazer para melhorar a coesão do controller é separar as responsabilidades em classes, veja
a seguir:

```java

@RestController
@RequestMapping("/api/v1/usuarios")
public class UsuarioController {

    @Autowired
    private UsuarioRepository usuarioRepository;
    @Autowired
    private ServicoExternoRemoto servicoExternoRemoto;
    @Autowired
    private ServicoNotificacaoExclusao servicoNotificacaoExclusao;

    @PostMapping
    public ResponseEntity<Usuario> cadastrar(@RequestBody Usuario usuario, @RequestBody Endereco endereco) {
        var regras = new RegrasDeCadastro(usuarioRepository, servicoExternoRemoto, () -> acaoParaQuandoNaoEstiverValido());
        regras.aplica(usuario);
        usuarioRepository.save(usuario);
        servicoExternoRemoto.enviaParaUmServicoExterno(usuario);
        return ResponseEntity.ok(usuario);
    }

    @GetMapping
    public ResponseEntity<List<Usuario>> listar() {
        return ResponseEntity.ok(usuarioRepository.findAll());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletar(@PathVariable Long id) {
        RegrasExcusao regras = new RegrasExcusao(usuarioRepository, servicoNotificacaoExclusao);
        regras.aplica(usuario);
        servicoNotificacaoExclusao.notifica(usuario);
        usuarioRepository.deleteById(id);
        return ResponseEntity.ok().build();
    }

}
 ```

Melhoramos um pouco ao separar as regras em classes específicas, mas ainda temos uma classe complexa, com algumas
responsabilidades e dependências que não deixam a classe ser coesa.
Isso não quer dizer que não podemos ter regras de negócio nos métodos dos controllers, mas que devemos ter cuidado com
a responsabilidade de cada método e com a quantidade de regras que ele possui. E ainda temos que verificar a coesão das
classes de Regras e dividir em classes menores se necessário.

## Controllers 100% coesos

Talvez a melhor forma de deixar um Controller coeso, seria sempre focar em ser 100% coeso realmente. Isso significa que
cada método do controller deve ter uma única responsabilidade, e que essa responsabilidade deve ser bem definida e cada
método deve utilizar todos os atributos da classe. Fazendo que por qualquer medida que você queira analisar, ele estará
coeso.
Veja como poderíamos refatorar o Controller anterior:

```java

@RestController
@RequestMapping("/api/v1/usuarios")
public class CriaUsuarioController

        @Autowired
        private UsuarioRepository usuarioRepository;
        @Autowired
        private ServicoExternoRemoto servicoExternoRemoto;

        @PostMapping
        public ResponseEntity<Usuario> cadastrar(@RequestBody Usuario usuario, @RequestBody Endereco endereco) {
            var regras = new RegrasDeCadastro(usuarioRepository, servicoExternoRemoto, () -> acaoParaQuandoNaoEstiverValido());
            regras.aplica(usuario);
            usuarioRepository.save(usuario);
            servicoExternoRemoto.enviaParaUmServicoExterno(usuario);
            return ResponseEntity.ok(usuario);
        }
}
```

```java

@RestController
@RequestMapping("/api/v1/usuarios")
public class BuscaUsuarioController {

    @Autowired
    private UsuarioRepository usuarioRepository;

    @GetMapping
    public ResponseEntity<List<Usuario>> listar() {
        return ResponseEntity.ok(usuarioRepository.findAll());
    }
}
```

```java

@RestController
@RequestMapping("/api/v1/usuarios")
public class DeletaUsuarioController {

    @Autowired
    private UsuarioRepository usuarioRepository;
    @Autowired
    private ServicoNotificacaoExclusao servicoNotificacaoExclusao;

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletar(@PathVariable Long id) {
        RegrasExcusao regras = new RegrasExcusao(usuarioRepository, servicoNotificacaoExclusao);
        regras.aplica(usuario);
        servicoNotificacaoExclusao.notifica(usuario);
        usuarioRepository.deleteById(id);
        return ResponseEntity.ok().build();
    }
}
```

Dessa forma temos uma classe para cada responsabilidade, e cada método dessas classes utilizam todas as dependências
dela.
Isso faz com que cada classe seja coesa, garantindo uma manutenção mais fácil e uma melhor compreensão do código, além
de ser bem melhor de testar.

Veja mais [aqui](https://youtu.be/bylZdwE8gKM) sobre Controllers 100% coesos.

