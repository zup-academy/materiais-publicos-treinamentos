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

Trata-se de um controller comum com Spring Boot, não é mesmo? A única diferença que não estamos acostumados é a injeção do Access Token via método através da anotação `@AuthenticationPrincipal`. Mas não se preocupe, logo chegaremos nela.

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

A primera coisa que precisamos fazer é escrever um teste de integração para este controller com jUnit e Spring Boot Testing. Para isso, basta criarmos a classe `NovoContatoControllerTest`, injetarmos as dependências necessárias para manipularmos as requisições e para limpeza do banco de dados, e sem seguida implementarmos um método de teste para o cenário feliz (happy-path):

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

Só o fato de adicionar essa biblioteca no projeto o Spring Boot de imediato já detecta e configura ela em tempo de execução ao rodar nossa bateria de testes. Mas ainda precisamos alterar nossos testes para enviar o token em cada requisição.

Com a dependência configurada, o próximo passo é fazermos o `import` da classe `SecurityMockMvcRequestPostProcessors` com seu método estático `jwt` na nossa nossa classe de testes:

```java
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.jwt;
```

E agora, vamos alterar a requisição (`MockHttpServletRequestBuilder`) para configurar o Post-Processor (`RequestPostProcessor`) do Spring Security responsável por adicionar o Access Token no formato JWT nas requisições do `MockMvc`:

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

Ao fazer isto, o Spring Security mocka um token JWT para nós, ou seja, ele gera e adiciona um token JWT "fake" válido na requisição do `MockMvc` contendo os seguintes dados:

```json
{
  "headers" : { "alg" : "none" },
  "claims" : {
    "sub" : "user",
    "scope" : "read"
  }
}
```

Apesar de ser um token mockado, o Spring MVC entende e aceita por causa de sua integração com Spring Security Testing. O que é suficiente para nosso ambiente de testes!

Perceba que, se rodarmos o teste novamente, nós não receberemos mais o erro `401 (Unauthorized)`, mas sim um `403 (Forbidden)`, como podemos ver abaixo:

```log
java.lang.AssertionError: Status expected:<201> but was:<403>
Expected :201
Actual   :403
```

Mas por que isso acontece se estamos enviando o token na requisição?

### 3. Configure o Scope no token

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

Poderíamos parar por aqui e ir para o próximo endpoint, mas ainda falta informar o `username` do Resource Owner no token JWT, lembra?

### 4. Configure claims no token

xxx

