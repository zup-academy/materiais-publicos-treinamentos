# Escrevendo testes de integração para API REST protegida por OAuth2

Nesse conteúdo veremos como podemos escrever testes de integração para uma API REST protegida por Spring Security e OAuth2 Resource Server. Aprenderemos como integrar o módulo Spring Boot Testing com o módulo Spring Security Testing e OAuth2.

## Nossa aplicação: Meus Contatos

Imagine que já exista uma aplicação Spring Boot responsável por gerenciar contatos de uma empresa chamada de **Meus Contatos**. A mesma está configurada como **OAuth2 Resource Server pelo Spring Security**. Para ter acesso as funcionalidades da aplicação, a mesma expõe uma API REST contendo os seguintes endpoints:
- Cadastrar novo contato
- Listar todos os contatos
- Detalhar um contato existente

Para este momento, vamos trabalhar em cima do endpoint **Cadastrar novo contato**. Para isso, esta funcionalidade possui o seguinte controller implementado:

```java
@RestController
public class NovoContatoController {

    @Autowired
    private ContatoRepository repository;

    @Transactional
    @PostMapping("/api/contatos")
    public ResponseEntity<?> cadastra(@RequestBody @Valid NovoContatoRequest request,
                                      UriComponentsBuilder uriBuilder,
                                      @AuthenticationPrincipal Jwt user) {

        String usuario = user.getClaim("preferred_username");
        if (usuario == null) {
            usuario = "anonymous";
        }

        Contato contato = request.toModel(usuario);
        repository.save(contato);

        URI location = uriBuilder
                .path("/api/contatos/{id}")
                .buildAndExpand(contato.getId()).toUri();

        return ResponseEntity
                .created(location).build();
    }
}
```

Trata-se de um controller comum com Spring Boot, não é mesmo? A única diferença que não estamos acostumados é a injeção do Access Token (JWT) via método através da anotação `@AuthenticationPrincipal`. Mas não se preocupe, logo chegaremos nela.

As regras de acesso do Spring Security estão configuradas desta maneira:

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResourceServerConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // @formatter:off
        http.cors()
            .and()
                .csrf().disable()
                .httpBasic().disable()
                .rememberMe().disable()
                .formLogin().disable()
                .logout().disable()
                .requestCache().disable()
                .headers().frameOptions().deny()
            .and()
                .sessionManagement()
                    .sessionCreationPolicy(STATELESS)
            .and()
                .authorizeRequests()
                    .antMatchers(HttpMethod.GET, "/api/contatos").hasAuthority("SCOPE_contatos:read")
                    .antMatchers(HttpMethod.GET, "/api/contatos/**").hasAuthority("SCOPE_contatos:read")
                    .antMatchers(HttpMethod.POST, "/api/contatos").hasAuthority("SCOPE_contatos:write")
                .anyRequest()
                    .authenticated()
            .and()
                .oauth2ResourceServer()
                    .jwt()
        ;
        // @formatter:on
    }
}
```

Nosso objetivo é cobrir este endpoint com testes de integração do Spring Boot Testing integrado ao módulo Spring Security Testing, dessa forma poderemos enviar um Access Token no formato JWT com seus respectivos scopes e claims.

## Cobrindo o controler com testes

### 1. Escreva um teste de integração e veja-o quebrar com `401-Unauthorized`

A primera coisa que precisamos fazer é escrever um teste de integração para este controller com jUnit e Spring Boot Testing.

> **Não sabe escrever testes de integração com Spring Boot?** <br/>
> Neste módulo assumimos que você já domina a escrita de testes de integração com Spring Boot, por esse motivo não entraremos em detalhes. Caso tenha dificuldades ou esteja enferrujado(a) nesse momento, você pode se inscrever no treinamento de [Testes de Integração com Spring Boot e Spring Testing](https://handora.zup.com.br/treinamentos/7) na plataforma da Handora.


Para isso, basta criarmos a classe `NovoContatoControllerTest`, injetarmos as dependências necessárias para manipularmos as requisições e para limpeza do banco de dados, e sem seguida implementarmos um método de teste para o cenário feliz (happy-path):

```java
@SpringBootTest
@ActiveProfiles("test")
@AutoConfigureMockMvc(printOnlyOnFailure = false)
class NovoContatoControllerTest {

    @Autowired
    protected MockMvc mockMvc;
    @Autowired
    private ObjectMapper mapper;
    @Autowired
    private ContatoRepository repository;

    @BeforeEach
    public void setUp() {
        repository.deleteAll();
    }

    @Test
    public void deveCadastrarNovoAutor() throws Exception {
        // cenário
        NovoContatoRequest novoContato = new NovoContatoRequest("Alberto", "Zup", List.of(
                new TelefoneRequest("casa", "+5511988888888"),
                new TelefoneRequest("celular", "+5511999999999")
        ));

        // ação
        String json = mapper.writeValueAsString(novoContato);

        mockMvc.perform(post("/api/contatos")
                            .contentType(APPLICATION_JSON)
                            .content(json))
                .andExpect(status().isCreated())
                .andExpect(redirectedUrlPattern("**/api/contatos/*"))
        ;

        // validação
        assertEquals(1, repository.count(), "total de contatos");
    }
}
```

Se rodarmos este teste de integração receberemos o seguinte erro no console da nossa IDE:

```log
java.lang.AssertionError: Status expected:<201> but was:<401>
Expected :201
Actual   :401
```

O teste esperava um Status HTTP `201 (Created)` mas recebeu um `401 (Unauthorized)`, ou seja, o teste não conseguiu cadastrar nosso contato no banco de dados.

Embora o teste esteja correto, o mesmo quebrou pois nossa API REST está protegida pelo Spring Security com OAuth 2.0. A partir de agora, para podermos consumir esta API REST precisamos obter um Access Token válido do Keycloak e em seguida envia-lo como cabeçalho HTTP `Authorization` da requisição do `MockMvc`.

Mas será que faz sentido rodar um Keycloak para rodar para nosso testes? Na nossa opinião, **com toda certeza não**.

### 2. Gere e envie o Token na requisição

Apesar dos testes serem integrados, rodar um servidor Keycloak apenas para testar nossa API REST seria demais, poderia não só complicar nossas vidas com também ainda tornaria os testes mais lentos. O que precisamos é apenas de um Access Token válido durante a execução dos testes, sem se importar muito com quem de fato gera este token.

Para isso, podemos usar o módulo **Spring Security Testing**, na qual resolve este problema de geração e envio de tokens para nós. Para isso, vamos adicionar a dependência `spring-security-test` no `pom.xml` da aplicação:

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
</dependency>
```

Só o fato de adicionar essa biblioteca no projeto o Spring Boot Testing de imediato já detecta e configura ela em tempo de execução ao rodar nossa bateria de testes. Mesmo assim, ainda precisamos alterar nossos testes para enviar o token em cada requisição explicitamente.

Com a dependência configurada, o próximo passo é fazermos o `import` da classe `SecurityMockMvcRequestPostProcessors` com seu método estático `jwt` na nossa nossa classe de testes:

```java
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.jwt;
```

A idéia basica é invocar o método `SecurityMockMvcRequestPostProcessors.jwt()` registrando seu `RequestPostProcessor` na requisição (`MockHttpServletRequestBuilder`) do `MockMvc`.

> **O que é um `RequestPostProcessor`?** <br/>
> É uma interface funcional do Spring Boot Testing que funciona como **ponto de extensão** para aplicações ou bibliotecas terceiras que almejam modificar uma request do `MockMvc` após sua criação, neste caso a classe `MockHttpServletRequest`.
>
> Por exemplo, podemos alterar a request adicionando, removendo ou alterando suas informações como cabeçalhos, parâmetros, encoding, body, URL, tokens etc, até mudanças mais rebuscadas como configurar o usuário logado, suas credenciais e permissões.
>
> Praticamente, todos os [mecanismos de segurança do Spring Security integrados ao Spring Boot Testing](https://docs.spring.io/spring-security/reference/5.8/servlet/test/mockmvc/request-post-processors.html) são implementados através de `RequestPostProcessor`.

Agora, vamos alterar a requisição `POST` do nosso teste passando o token "fake" via método `jwt()` do  `SecurityMockMvcRequestPostProcessors`, pois ele será o responsável por gerar e adicionar o access token no formato JWT na requisição:

```java
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.jwt;

// código do método de testes

mockMvc.perform(
            post("/api/contatos")
                .contentType(APPLICATION_JSON)
                .content(json)
                .with(jwt()) // adiciona token JWT
            )
        .andExpect(status().isCreated())
        .andExpect(redirectedUrlPattern("**/api/contatos/*"))
;
```

Ao fazer isto, o Spring Security **mocka** um token JWT para nós, ou seja, ele gera e adiciona um token JWT "fake" porém válido na requisição do `MockMvc` contendo por default os seguintes dados:

```json
{
  "headers" : { "alg" : "none" },
  "claims" : {
    "sub" : "user",
    "scope" : "read"
  }
}
```

Apesar de ser um token mockado, o Spring Boot Testing entende e aceita este token por causa de sua integração com Spring Security Testing. O que é suficiente para nosso ambiente de testes!

Perceba que, se rodarmos o teste novamente, nós não receberemos mais o erro `401 (Unauthorized)`, mas sim outro erro, um `403 (Forbidden)`, como podemos ver abaixo:

```log
java.lang.AssertionError: Status expected:<201> but was:<403>
Expected :201
Actual   :403
```

Mas por que isso acontece se estamos enviando o token na requisição?

### 3. Configure os Scopes no token

Isso acontece pois o token enviado apesar de válido ele não possui os scopes necessários para consumir nosso endpoint.

Para corrigir isso, precisamos adicionar o scope `contatos:write` no nosso token JWT, e a maneira mais simples de fazer isso é adicionando um ou mais `SimpleGrantedAuthority` diretamente no token, como no seguinte código:

```java
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.jwt;

// código do método de testes

mockMvc.perform(
            post("/api/contatos")
                .contentType(APPLICATION_JSON)
                .content(json)
                .with(jwt()
                    .authorities(new SimpleGrantedAuthority("SCOPE_contatos:write"))
                )
            )
        .andExpect(status().isCreated())
        .andExpect(redirectedUrlPattern("**/api/contatos/*"))
;
```

Se rodarmos o teste novamente desta vez ele passa com sucesso! 🥳

> **Customize ainda mais seu token JWT** <br/>
> Você pode especificar seu token JWT de forma completa para cenários de testes mais complexos, como abaixo:
> ```java
> Jwt jwt = Jwt.withTokenValue("token")
>    .header("alg", "none")
>    .claim("sub", "user")
>    .claim("scope", "read")
>    .build();
>
> mvc
>    .perform(get("/endpoint")
>        .with(jwt().jwt(jwt)));
> ```
> Para mais informações, não esqueça de consultar a [documentação oficial do Spring Security Testing](https://docs.spring.io/spring-security/reference/5.7.0/servlet/test/mockmvc/oauth2.html#_jwt_requestpostprocessor).
>

Poderíamos parar por aqui e ir para o próximo endpoint, mas ainda falta informar o usuário (Resource Owner) no token JWT, lembra?

### 4. Configure claims no token

Se você olhou com atenção a implementação do controller `NovoContatoController`, você percebeu que o "usuário logado" (neste caso, uma instância de `JWT`) está sendo injetada via parâmetro de método com o auxílio da anotação `@AuthenticationPrincipal`:

```java
@PostMapping(...)
public ResponseEntity<?> cadastra(..., @AuthenticationPrincipal Jwt user) {

    String usuario = user.getClaim("preferred_username");
    if (usuario == null) {
        usuario = "anonymous";
    }

    // restante do código
}
```

Repare também que a claim `preferred_username` é extraída do token para executar alguma lógica de negócio, neste caso apenas por questões de auditoria. Infelizmente nossa implementação do teste não leva isso em consideração, ela não passa esta claim na requisição, o que acaba gravando o usuário `anonymous` no banco de dados.

> **O que são Claims?** <br/>
> Podemos entender as claims como informações sobre o usuário (Resource Owner) e, também, informações sobre o token em si. Claims comuns em um token JWT são name, email, first name, last name, username entre outros.
>
> Lembre-se, alguns Authorization Servers podem adicionar claims especificas cutomizadas ou mesmo padrões nos tokens, no exemplo acima a claim `preferred_username` apesar de padrão no OpenID Connect, ela é uma claim adicionada ao Access Token pelo Keycloak, ou seja, talvez outros vendors não a adicionem por padrão no token.

Vamos alterar nosso teste para adicionar a claim `preferred_username` no token antes de exercitar o endpoint. Para isso, precisamos manipular diretamente a instância do token `jwt`, como abaixo:

```java
mockMvc.perform(
            post("/api/contatos")
                .contentType(APPLICATION_JSON)
                .content(json)
                .with(jwt()
                    .jwt(jwt -> {
                        jwt.claim("preferred_username", "rponte");
                    })
                    .authorities(new SimpleGrantedAuthority("SCOPE_contatos:write"))
                )
            )
        .andExpect(status().isCreated())
        .andExpect(redirectedUrlPattern("**/api/contatos/*"))
;

// validação
assertEquals(1, repository.count(), "total de contatos");
assertEquals("rponte", repository.findAll().get(0).getCriadoPor(), "criado por"); // verifica efeito colateral
```

Repare também que adicionamos uma validação para ter certeza que o usuário do token (`preferred_username`) foi gravado como esperado no banco de dados. Em alguns cenários esse tipo de verificação pode ser muito importante.

Ao fazer isso, o teste continuará passando, mas desta vez o usuário `rponte` existente no token foi extraído e gravado corretamente no banco de dados 🥳 


### 5. Não esqueça os cenários para `401-Unauthorized` e `403-Forbidden`

O que aconteceria se tentassemos enviar uma requisição sem um token? Ou se enviarmos uma requisição com um token sem os scopes apropriados? Neste momento você sabe que o teste quebraria com os erros HTTP `401-Unauthorized` e `403-Forbidden`, afinal foi o que aconteceu ao rodar nosso teste.

Mas e se amanhã alguma regra de acessa mudar e um endpoint que deveria estar protegido é liberado ou "afrouxado" sem querer por algum desenvolvedor(a)? Nesse cenário nosso teste continuaria passando, afinal não haveria mais verificação do token ou de seus scopes. O que estou querendo dizer é que se **desligarmos o Spring Security** no ambiente de testes há grandes de chances de todos os testes (ou a maioria deles) continuarem executando com sucessso, desta forma mudanças críticas no código não seriam detectadas por nossa bateria de testes.

Por esse motivo entendemos que precisamos ter uma bateria de testes que detecte com precisão estes tipos de mudanças. Para isso, uma boa prática é garantir que as regras de acesso estejam de fato habilitadas para cada endpoint, e alcançamos isto através de 2 cenários simples:

1. O que acontece quando a requisição não possui token?
2. O que acontece quando a requisição possui um token mas não possui o Scope esperado?

Enquanto o primeiro cenário trata do Status HTTP `401-Unauthorized`, o segundo cenário espera o Status `403-Forbidden`. Para implementa-los basta termos os 2 métodos de testes na nossa classe de testes que esperem esses Status de erro:

```java
@Test
public void naoDeveCadastrarNovoAutor_quandoTokenNaoEnviado() throws Exception {
    // cenário
    NovoContatoRequest novoContato = new NovoContatoRequest("Alberto", "Zup", List.of(
            new TelefoneRequest("casa", "+5511988888888"),
            new TelefoneRequest("celular", "+5511999999999")
    ));

    // ação
    String json = mapper.writeValueAsString(novoContato);

    mockMvc.perform(post("/api/contatos")
                        .contentType(APPLICATION_JSON)
                        .content(json)) // sem token
            .andExpect(status().isUnauthorized()) // esperamos um 401
    ;
}

@Test
public void naoDeveCadastrarNovoAutor_quandoTokenNaoPossuiEscopoApropriado() throws Exception {
    // cenário
    NovoContatoRequest novoContato = new NovoContatoRequest("Alberto", "Zup", List.of(
            new TelefoneRequest("casa", "+5511988888888"),
            new TelefoneRequest("celular", "+5511999999999")
    ));

    // ação
    String json = mapper.writeValueAsString(novoContato);

    mockMvc.perform(
                post("/api/contatos")
                    .contentType(APPLICATION_JSON)
                    .content(json)
                    .with(jwt()) // token sem scope(s)
                )
            .andExpect(status().isForbidden()) // esperamos um 403
    ;
}
```

Idealmente o cenário é o mesmo do happy-path, pois assim saberemos que a requisição foi negada pelo Spring Security e não por alguma regra de negócio no código ou problemas no nosso payload.

Pronto! Mais uma vez ao rodarmos a classe de testes todos os 3 cenários de testes passam com sucesso 🥳

## Links e referências

- [Spring Security Testing OAuth 2.0](https://docs.spring.io/spring-security/reference/5.7.0/servlet/test/mockmvc/oauth2.html)
- [Spring Security Testing](https://docs.spring.io/spring-security/reference/5.7.0/servlet/test/index.html)
- [Spring Security Testing: SecurityMockMvcRequestPostProcessors](https://docs.spring.io/spring-security/reference/5.8/servlet/test/mockmvc/request-post-processors.html)
- [Spring Security Testing: Testing JWT Authentication](https://docs.spring.io/spring-security/reference/5.7.0/servlet/test/mockmvc/oauth2.html#testing-jwt)
- [OpenID Connect Scopes](https://auth0.com/docs/get-started/apis/scopes/openid-connect-scopes)
- [OpenID Connect Standard Claims](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims)