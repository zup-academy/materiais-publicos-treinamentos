# Consumindo uma API REST protegida por OAuth2 via OpenFeign

Nesse conteúdo veremos como podemos consumir uma API REST protegida por OAuth 2.0 de um sistema externo utilizando o cliente HTTP **Spring Cloud OpenFeign** para Spring Boot.

## Nosso contexto: filtrando os contatos de uma empresa

Nossa missão é construir um microsserviço que expões uma API REST para listar os contatos de uma determinada empresa. Nossa API REST precisa receber como entrada o nome da empresa e usa-lo para filtrar apenas os contatos que pertencem a esta empresa.

Para isso, não vamos buscar estes contatos no nosso banco de dados como de costume, mas sim consumiremos a API REST do sistema Meus Contatos. Esse sistema expõe diversos endpoints, mas nesse momento estamos interessados no seu endpoint de listagem de contatos:

```json
GET /api/contatos
[
	{
		"id": 1,
		"nome": "Yuri Matheus",
		"empresa": "Zup"
	},
	{
		"id": 9,
		"nome": "Steve Jobs",
		"empresa": "Apple"
	},
    {
        ...
    }
]
```

Por questões de segurança, todos os endpoints do sistema Meus Contatos estão protegidos pelo procoloto OAuth 2.0, e para acessa-los precisamos obter um Access Token do [Authorization Server (Keycloak)](/seguranca-com-spring-security-e-oauth2/004-Instalando-Keycloak-via-Docker-Compose.md). Perceba que aqui o papel do sistema Meus Contatos dentro do fluxo OAuth 2.0 é justamente o de Resource Server.

Por se tratar de uma integração M2M (Machine-to-Machine), ou seja, não há necessidade da interação do usuário para informar credenciais ou logar no nosso sistema, precisaremos obter este token via o fluxo OAuth2 conhecido como **Client Credentials Flow**.

> **Como configuro o Client Credentials Flow no Keycloak?** <br/>
> Como vimos, o fluxo Client Credentils foi desenhado para comunicação Machine-to-Machine (M2M), e para configura-lo no Keycloak você pode ler nosso material teórico [Configurando Client Credentials Flow no Keycloack](/seguranca-com-spring-security-e-oauth2/017-Configurando-Client-Credentials-Flow.md).

Nesse momento, assuma que o Client, Scopes e este fluxo já estejam devidamente configurados no Keycloak e que precisaremos apenas obter um Access Token antes de consumir a API REST do Meus Contatos.

Usaremos toda a stack do Spring Boot que já conhecemos para construir nosso endpoint, incluindo o módulo Spring Security OAuth2 Client.

### 1. Adicione a dependência do Spring Security no `pom.xml`

A primeira coisa que temos que fazer é configurar nosso projeto Spring Boot com a dependência do Maven, neste caso estamos falando do **Spring Boot Starter OAuth2 Client**. Para isso, basta adicionar a dependência abaixo no seu `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

O artefato `spring-boot-starter-oauth2-client` inclui todas as dependências necessárias por nos dar suporte ao modo OAuth 2.0 Client. Esta dependência também inclui as bibliotecas core do Spring Security, por esse motivo, podemos remover a dependencia `spring-boot-starter-security` do nosso `pom.xml` sem qualquer problema. 

> **Aplicação nova? Vá de Spring Initializr!** <br/>
> Nós adicionamos a dependência no `pom.xml` explicitamente pois estamos partindo de um projeto existente, porém se você está criando um novo projeto Spring Boot através do [Spring Initializr](https://start.spring.io/) e, de antemão sabe que precisará consumir outra aplicação ou serviço protegido por OAuth 2.0, aproveite para adicionar a dependência **OAuth2 Client** no momento do setup da sua aplicação Spring Boot.

Se estiver numa IDE, lembre-se de recarregar as dependências do Maven.

### 3. Configure `application.yml` com informações do Authorization Server

Precisamos configurar as informações do Client que registramos no Authorization Server (Keycloak). Essa é a forma como estabelecemos uma relação de confiança entre Client e Authorization Server, ou seja, precisamos desse passo para que nossa aplicação (OAuth2 Client) consiga solicitar um Access Token para o Authorization Server.

Para que esta comunicação ocorra, basicamente precisamos configurar as seguintes informações na aplicação:

1. **Informações do Client**: são as mesmas informações do Client que foi registado no Authorization Server;
2. **Informações do Service Provider**: são as informações necessárias para acessar o Authorization Server;

Vamos entender como configurar cada um destas informações para satisfazer o módulo Spring Security OAuth 2.0 Client.

#### Informações do Client

Dado que no Keycloak temos o Realm `meus-contatos` possuindo um Client chamado `meus-contatos-client` configurado com suporte a Client Credentials Flow e seus respectivos Scopes associados, a configuração no `application.yml` seria semelhante a esta:

```yml
spring:
  security:
    oauth2:
      client:
        registration:
          meus-contatos:
            authorization-grant-type: client_credentials
            client-id: meus-contatos-client
            client-secret: IUGoaJlRlczbg4d40sTDf3Lwj6SSKW3L
            scope: contatos:read, contatos:write
```

Repare que usamos a namespace `spring.security.oauth2.client.registration`, que é a namespace raiz para configurar qualquer Client. Ou seja, podemos registrar múltiplos Clients na nossa aplicação.

No caso acima, configuramos um Client chamado `meus-contatos` (poderia ser qualquer outro nome) de acordo com o Client registrado no Authorization Server (Keycloak), com suas respectivas propriedades `client-id` e `client-secret`, além do `scope` e `authorization-grant-type` a ser utilizado.

#### Informações do Service Provider

Após registrar nosso Client na aplicação, precisamos **definir o Service Provider** que basicamente significa configurar as informações do Authorization Server. Para isso, usamos a namespace `spring.security.oauth2.client.provider` que recebe o mesmo nome `meus-contatos`, como abaixo:


```yml
spring:
  security:
    oauth2:
      client:
        registration:
          meus-contatos:
            # ...
        provider:
          meus-contatos:
            token-uri: http://localhost:18080/auth/realms/meus-contatos/protocol/openid-connect/token
```

Perceba que configuramos a propriedade `token-uri` com a URI do Keycloak responsável por gerar nosso Access Token.

### 4. Crie seu OpenFeign client

Precisamos de um simples HTTP client para consumir a API REST do Resource Server (sistema Meus Contatos). Mas em vez de usarmos uma biblioteca de HTTP client qualquer, utilizaremos o HTTP client declarativo **OpenFeign**, que tem sua integração com Spring Boot feita através do projeto **Spring Cloud OpenFeign**. Se você é novo com OpenFeign, [este artigo introdutório](https://www.baeldung.com/spring-cloud-openfeign) pode te ajudar a dar os primeiros passos.

A vantagem de usar o OpenFeign aqui é que eles no permite implementar integração HTTP com APIs REST de forma totalmente **declarativa**. Basicamente precisamos declarar uma interface com métodos e anota-los, a partir disto toda a mágica do Feign acontece por debaixo dos panos. Não à toa ela tem se tornada uma das principais alternativas quando desenvolvedores(as) precisam se integrar com serviços externos.

Para isso, precisamos adicionar a dependência do Spring Boot Starter do OpenFeign no nosso `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

Além disso, também precisamos adicionar as dependências do projeto Spring Cloud no `pom.xml`, como abaixo:

```xml
<!-- adicione uma propriedade com a versão do Spring Cloud -->
<properties>
    <spring-cloud.version>2021.0.2</spring-cloud.version>
</properties>

<!-- adicione as dependências do Spring Cloud -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Nós podemos encontrar as últimas versões do [`spring-cloud-starter-openfeign`](https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-starter-openfeign) e [`spring-cloud-dependencies`](https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-dependencies) no repositório central do Maven. Caso precise verificar a compatibilidade com a versão do Spring Boot usada na sua aplicação, não deixe de ler a tabela de compatibilidade na [documentação oficial do Spring Cloud](https://spring.io/projects/spring-cloud/).

Agora, precisamos habilitar o suporte a clientes declarativos do OpenFeign na nossa aplicação para que o Spring Boot consiga encontrar todos os clientes declarados no classpath:

```java
@EnableFeignClients // habilita OpenFeign na aplicação
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

A partir de agora podemos declarar nossos clientes de maneira declarativa. Por exemplo, para consumir o endpoint de listagem de contatos do sistema Meus Contatos basta declararmos uma interface com um método que mapeia este endpoint, como abaixo:

```java
@FeignClient(
    name = "meusContatos",
    url = "http://localhost:8080/meus-contatos"
)
public interface MeusContatosClient {

    @GetMapping("/api/contatos")
    public List<ContatoResponse> lista();

}

/**
 * Mapeado com base no payload de resposta
 */
public class ContatoResponse {

    private Long id;
    private String nome;
    private String empresa;

    // construtor e getters

}
```

Perceba que anotamos a interface `MeusContatosClient` com a anotação `@FeignClient` para indicar de que se trata de uma cliente HTTP do Feign, também definimos um `name` e uma URL base (`url`) que aponta para API REST do sistema Meus Contatos. No caso do método `lista()`, nós o anotamos com a anotação `@GetMapping` indicando seu endpoint, justamente as anotações do Spring MVC que já estamos acostumados 😉

Para utilizar esta cliente basta injeta-lo no controller ou em qualquer outro bean gerenciado pelo Spring e em seguida invocar seus métodos. A implementação de fato desta interface ocorre em runtime, nós não precisamos nos preocupar com isso!

Nosso HTTP client está configurado e pronto para uso, porém ele ainda não está completo, pois se tentarmos utiliza-lo para acessar endpoint do sistema Meus Contatos nós receberemos um erro com Status HTTP `401 (Unauthorized)`, afinal não estamos repassando nenhum Access Token na requisição, não é mesmo?

Portanto, vamos dar um jeito de repassar este token em cada uma das requisições 😉

### 5. Configure o OpenFeign para trabalhar com fluxo OAuth 2.0

Para que nosso OpenFeign consiga se comunicar de maneira autenticada/autorizada com o Resource Server, precisamos configurá-lo para que ele leve o fluxo OAuth 2.0 em consideração.

**A idéia aqui é obter um Access Token válido do Authorization Server e em seguida adiciona-lo como cabeçalho HTTP (`Authorization: Bearer <token>`) em cada requisição enviada para o Resource Server**. Para não termos que implementar esse workflow manualmente, precisamos registrar um interceptor no OpenFeign que cuidará exatamente destes passos para nós.

Antes de tudo, isso significa criar e configurar uma instância de `OAuth2AuthorizedClientManager`, que é o objeto responsável por gerenciar todo o workflow de solicitação de tokens para o Authorization Server assim como gerenciar o ciclo de vida destes tokens.

Para isso, crie a classe de configurações `ClientSecurityConfig` e declare um novo método de fábrica para criar nosso `OAuth2AuthorizedClientManager` configurado para Client Credentials Flow, como abaixo:

```java
@Configuration
class ClientSecurityConfig {

    @Bean
    public OAuth2AuthorizedClientManager authorizedClientManager(ClientRegistrationRepository clientRegistrationRepository,
                                                                 OAuth2AuthorizedClientService clientService) {

        OAuth2AuthorizedClientProvider provider 
                    = OAuth2AuthorizedClientProviderBuilder.builder()
                                        .clientCredentials()
                                        .build();

        AuthorizedClientServiceOAuth2AuthorizedClientManager manager 
                    = new AuthorizedClientServiceOAuth2AuthorizedClientManager(clientRegistrationRepository, clientService);
        authorizedClientManager.setAuthorizedClientProvider(provider);

        return manager;
    }
}
```

Perceba que nosso `OAuth2AuthorizedClientManager`, que é uma interface, tem como implementação a classe `AuthorizedClientServiceOAuth2AuthorizedClientManager`, na qual recebe como dependência os beans `ClientRegistrationRepository` e `OAuth2AuthorizedClientService`, que serão os responsáveis, respectivamente, por carregar as informações dos clients registrados no `application.yml` e gerenciar os tokens obtidos do Authorization Server. Além disso, também definimos um Provider através da classe `OAuth2AuthorizedClientProvider`, que é responsável por definir quais estratégias serão usadas para autenticar/autorizar nossos clients, que neste momento se limita a apenas Client Credentials Flow.

Este objeto `OAuth2AuthorizedClientManager` pode ser utilizado para obter os tokens manualmente se assim desejarmos, pois ele é quem de fato cuida de 90% deste workflow. Mas por se tratar de um workflow que se repetirá para cada requisição enviada, faz todo sentido que ele seja encapsulado num interceptor e configurado no nosso Feign Client.

> **E se o token expirar?** <br/>
> O que acontece quando um token obtido pelo `OAuth2AuthorizedClientManager` expira ou recebe um erro `401 (Unauthorized)` do Resource Server?
>
> Como sabemos, a classe `OAuth2AuthorizedClientManager`, através de suas dependências, se encarrega de gerenciar todo o workflow relacionado aos tokens obtidos e autorizados pelo Authorization Server, incluindo o ciclo de vida destes tokens. Isso siginica que ele:
> 
> - Obtem os tokens do Authorization Server;
> - Armazena estes tokens em memória (ou outro storage);
> - Relaciona os tokens obtidos a seus respectivos registros de clients configurados no `application.yml` (na verdade carregados pelo `ClientRegistrationRepository`);
> - Se o token expirar ele será renovado (refreshed) através do fluxo OAuth2 Refresh Token Flow (se o fluxo original permitir);
> - Substitui o token expirado pelo novo token;
> 
> Como podemos ver, se tivessemos que fazer tudo isso na mão, sem o uso do `OAuth2AuthorizedClientManager`, teríamos um trabalho imenso e passível de erros, mas por sorte o Spring Security resolve isto para nós 😉


Por fim, precisamos configurar o OpenFeign para trabalhar com o fluxo OAuth 2.0 esperado pelo Authorization Server e Resource Server. A idéia é **configurarmos um interceptor** no OpenFeign que se integre ao `OAuth2AuthorizedClientManager` do Spring Security a fim de habilitar o fluxo OAuth 2.0.

> ⚠️ **OpenFeign sem suporte a OAuth 2.0 com Spring Security 5.x** <br/>
> Embora o OpenFeign tenha uma boa integração com Spring Security há anos, seu interceptor padrão funcniona somente a versão antiga do Spring Security.
>
> Infelizmente, até este momento, o OpenFeign não possui suporte a OAuth 2.0 na versão mais recente do Spring Security, a versão 5.x, portanto ele não nos fornece um interceptor out-of-the-box para esta versão.

Infelizmente o OpenFeign não nos fornece um interceptor de fluxo OAuth 2.0 pronto para Spring Security 5.x, mas não se preocupe, implementar um interceptor não é uma tarefa difícil. A idéia básica deste interceptor é obter um Access Token via `OAuth2AuthorizedClientManager` e adiciona-lo como cabeçalho em cada requisição HTTP antes dessa requisição ser de fato enviada ao Resource Server.

A implementação deste interceptor seria algo como abaixo:

```java
import feign.RequestInterceptor;
import feign.RequestTemplate;
import org.springframework.http.HttpHeaders;
import org.springframework.security.authentication.AnonymousAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.oauth2.client.OAuth2AuthorizeRequest;
import org.springframework.security.oauth2.client.OAuth2AuthorizedClient;
import org.springframework.security.oauth2.client.OAuth2AuthorizedClientManager;
import org.springframework.security.oauth2.core.OAuth2AccessToken;

/**
 * Infelizmente a lib Open Feign ainda não suporta a versão do OAuth2 no Spring Security 5. Seu interceptor
 * padrão funciona somente com a versão antiga.
 *
 * Inpirado na implementação do @loesak
 * - https://github.com/loesak/spring-security-openfeign/blob/master/README.md
 * - https://www.baeldung.com/spring-cloud-feign-oauth-token
 */
public class OAuth2FeignRequestInterceptor implements RequestInterceptor {

    private static final Authentication ANONYMOUS_AUTHENTICATION = new AnonymousAuthenticationToken(
            "anonymous", "anonymousUser", AuthorityUtils.createAuthorityList("ROLE_ANONYMOUS"));

    private final OAuth2AuthorizedClientManager authorizedClientManager;
    private final String clientRegistrationId;

    public OAuth2FeignRequestInterceptor(OAuth2AuthorizedClientManager authorizedClientManager, String clientRegistrationId) {
        this.authorizedClientManager = authorizedClientManager;
        this.clientRegistrationId = clientRegistrationId;
    }

    @Override
    public void apply(RequestTemplate request) {
        if (this.authorizedClientManager == null) {
            return;
        }

        OAuth2AuthorizeRequest authorizeRequest = OAuth2AuthorizeRequest
                                            .withClientRegistrationId(this.clientRegistrationId)
                                            .principal(ANONYMOUS_AUTHENTICATION)
                                            .build();

        OAuth2AuthorizedClient authorizedClient = this.authorizedClientManager.authorize(authorizeRequest);
        if (authorizedClient == null) {
            throw new IllegalStateException(
                    "This client uses an authorization grant type that's not supported by the " +
                            "configured provider. ClientRegistrationId = " + this.clientRegistrationId);
        }

        OAuth2AccessToken accessToken = authorizedClient.getAccessToken();
        request
            .header(HttpHeaders.AUTHORIZATION,
                    String.format("Bearer %s", accessToken.getTokenValue()));
    }
}
```

Com o interceptor no classpath da aplicação, o próximo passo é torna-lo um bean gerenciado pelo container do Spring. Para isso, ainda na classe `ClientSecurityConfig`, **declare e configure nosso interceptor** `OAuth2FeignRequestInterceptor` através de um método de fábrica do Spring:

```java
@Configuration
class ClientSecurityConfig {

    @Bean
    public OAuth2FeignRequestInterceptor oAuth2FeignRequestInterceptor(OAuth2AuthorizedClientManager clientManager) {
        return new OAuth2FeignRequestInterceptor(clientManager, "meus-contatos");
    }

    // ...
}
```

Repare que injetamos a nossa instância configurada de `OAuth2AuthorizedClientManager` para passa-la como dependência para nosso interceptor `OAuth2FeignRequestInterceptor`. **Este interceptor se encarregará de obter um token válido antes de cada requisição disparada ao Resource Server**. Em adição a isto, nós também indicamos ao interceptor qual registro de client padrão (default) ele deve utilizar para se comunicar com o Authorization Server, que no nosso caso é justamente o registro `meus-contatos` que configuramos no `application.yml` anteriormente.

Ao declarar o interceptor `OAuth2FeignRequestInterceptor` como bean do Spring, o OpenFeign se encarrega de detecta-lo no startup da aplicação e configura-lo para cada um de seus clients (interfaces anotadas com `@FeignClient`).

> ⚠️ **Cuidado com `OAuth2FeignRequestInterceptor` global na aplicação** <br/>
> Nós criamos e declaramos nosso interceptor `OAuth2FeignRequestInterceptor` no contexto do Spring, desta forma todos as requisições enviadas **por todos** os clients do OpenFeign receberão o Access Token obtido do Authorization Server. Apesar de prático, devemos ter cuidado ao utilizar esta configuração em funcionalidades que não participam de um fluxo OAuth 2.0, caso contrário podemos expor o token para serviços externos, o que abre brechas sérias de segurança.
>
> Se você tem dois ou mais serviços externos que sua aplicação precisa se integrar, lembre-se de revisar o uso do seu interceptor. Nestes cenários, você provavelmente terá configurações especificas para cada cliente do OpenFeign.

Pronto! A partir de agora nosso cliente OpenFeign `MeusContatosClient` consegue se comunicar com a API REST protegida do Resource Server (Meus Contatos). Isso acontece pois o interceptor vai obter um token válido do Authorization Server e em seguida adiciona-lo ao cabeçalho HTTP de todas as requisições enviadas ao Resource Server.

E agora, vamos utiliza-lo?

### 6. Consuma o sistema externo protegido por OAuth 2.0

Agora, vamos consumir a API REST do sistema Meus Contatos, para isso vamos implementar nosso controller para expor nossa própria API REST que, por sua vez, consumirá o endpoint do sistema Meus Contatos.

Lembre-se, nossa API REST precisa expor um endpoint que permita filtrar todos os contatos pelo nome da empresa, deste modo o código do nosso controller seria algo semelhante a este:

```java
@RestController
public class FiltraContatosPorEmpresaController {

    @Autowired
    private MeusContatosClient client;

    @GetMapping("/api/contatos-por-empresa")
    public ResponseEntity<?> listaPorEmpresa(@RequestParam(required = true) String empresa) {

        List<ContatoFiltradoResponse> contatos = client.lista().stream()
                                .filter(c -> empresa.equals(c.getEmpresa()))
                                .map(contato -> new ContatoFiltradoResponse(contato))
                                .collect(toList());

        return ResponseEntity
                    .ok(contatos);
    }
}


/**
 * Mapeado com base no payload de resposta
 */
public class ContatoFiltradoResponse {

    private Long id;
    private String nome;
    private String empresa;

    public ContatoFiltradoResponse(ContatoResponse contato) {
        this.id = contato.getId();
        this.nome = contato.getNome();
        this.empresa = contato.getEmpresa();
    }

    // getters

}
```

Perceba que injetamos nosso serviço cliente `MeusContatosClient` diretamente no controller, e o usamos para consumir o sistema externo através do seu método `lista()`, o resto do código basicamente filtra a coleção de contatos pela empresa informada e transforma cada contato em um DTO de saída do nosso controller, no caso `ContatoFiltradoResponse`.

Para testar nossa API REST, basta levantar a aplicação e executar o seguinte comando cURL na linha de comando:

```sh
curl --request GET \
  --url 'http://localhost:8080/api/contatos-por-empresa?empresa=Apple'
```

Como resposta temos um Status HTTP `200 (OK)` com o seguinte payload:

```json
[
    {
        "id": 9,
        "nome": "Steve Jobs",
        "empresa": "Apple"
    }
]
```

Nosso código do controller utilizou a classe `MeusContatosClient` que fez toda comunicação com o serviço externo. Por debaixo dos panos nosso interceptor juntamente com a configuração de fluxo OAuth 2.0 que fizemos entrou em ação obtendo um Access Token do Authorization Server, adicionando-o no cabeçalho da requisição e, por fim, enviando a requisição para o Resource Server.

Apesar de toda configuração do Spring Security e conhecimento sobre fluxos OAuth 2.0, não é uma tarefa dificil assim, não é mesmo?

## Habilite os logs do OpenFeign

Embora nosso código tenha funcionado como esperado não te incomoda saber que toda mágica ocorreu por debaixo dos panos?

O problema não é a mágica acontecer, mas sim não podermos visualiza-la enquanto desenvolvemos e testamos nosso código. Visualizar o que acontece  **através de logs** é muito importante para fazermos troubleshooting (analisar, identificar e corrigir erros) quando as coisas parecem não funcionar como esperado.

A logger is created for each Feign client created. By default the name of the logger is the full class name of the interface used to create the Feign client. Feign logging only responds to the DEBUG level.

Um logger é criado para cada Feign client criado. Por padrão o nome do logger é o full class name da interface usada para criar o client do Feign. Para habilitar o logging para nosso cliente `MeusContatosClient` em modo `DEBUG` basta declarar no `application.yml` as seguintes linhas:

```yml
logging:
    level:
        org.springframework.web.client: DEBUG
        br.com.zup.edu.meuscontatos.client.MeusContatosClient: DEBUG
```

Perceba que além de configurar o logging do nosso Feign client `MeusContatosClient` eu também tive que configurar o logging do pacote `org.springframework.web.client` do Spring Boot, pois o Spring Boot utiliza seu HTTP client padrão, o `WebClient`, em vez do Open Feign para obter o token do Authorization Server. Sem esta configuração seria impossível visualizar os logs das requisições para obter o Access Token.

Apenas com estas linhas temos uma boa idéia das requisições HTTP enviadas pelo OpenFeign, o que já é bastante útil para resolver muitos tipos de problemas, mas infelizmente **não temos detalhes sobre os headers e body da requisição**. Para habilitar o logging dos headers e body precisamos configurar o `Logger.Level` do nosso cliente:

```yml
feign:
  client:
    config:
      meusContatos:
        loggerLevel: full
```

A partir de agora ao rodarmos a aplicação e tentarmos nos comunicar com o Resource Server veremos algumas linhas de log semelhantes a estas:

```log
o.s.web.client.RestTemplate              : HTTP POST http://localhost:18080/auth/realms/meus-contatos/protocol/openid-connect/token
o.s.web.client.RestTemplate              : Accept=[application/json, application/*+json]
o.s.web.client.RestTemplate              : Writing [{grant_type=[client_credentials], scope=[contatos:read]}] as "application/x-www-form-urlencoded;charset=UTF-8"
o.s.web.client.RestTemplate              : Response 200 OK
o.s.web.client.RestTemplate              : Reading to [org.springframework.security.oauth2.core.endpoint.OAuth2AccessTokenResponse] as "application/json"
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] ---> GET http://localhost:8080/meus-contatos/api/contatos HTTP/1.1
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICIwR085RC1BTlhzU3pwWHBENlN2cm9Ca2tEaUlIb0otRnBiSkgxOTNnOUNVIn0.eyJleHAiOjE2NTQ1MjY0MjcsImlhdCI6MTY1NDUyNjEyNywianRpIjoiMzdhNjVkZDYtMGZiOC00ZDY4LWI1NTctOGQxOTk1NmY3MjliIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDoxODA4MC9hdXRoL3JlYWxtcy9tZXVzLWNvbnRhdG9zIiwiYXVkIjoiYWNjb3VudCIsInN1YiI6IjEyNDFiM2QzLWE1OTItNDExYy05OThlLTZlMTNhOTUxNDAyMSIsInR5cCI6IkJlYXJlciIsImF6cCI6Im1ldXMtY29udGF0b3MtY2xpZW50IiwiYWNyIjoiMSIsInJlYWxtX2FjY2VzcyI6eyJyb2xlcyI6WyJkZWZhdWx0LXJvbGVzLW1ldXMtY29udGF0b3MiLCJvZmZsaW5lX2FjY2VzcyIsInVtYV9hdXRob3JpemF0aW9uIl19LCJyZXNvdXJjZV9hY2Nlc3MiOnsiYWNjb3VudCI6eyJyb2xlcyI6WyJtYW5hZ2UtYWNjb3VudCIsIm1hbmFnZS1hY2NvdW50LWxpbmtzIiwidmlldy1wcm9maWxlIl19fSwic2NvcGUiOiJwcm9maWxlIGVtYWlsIGNvbnRhdG9zOnJlYWQiLCJlbWFpbF92ZXJpZmllZCI6ZmFsc2UsImNsaWVudElkIjoibWV1cy1jb250YXRvcy1jbGllbnQiLCJjbGllbnRIb3N0IjoiMTcyLjI1LjAuMSIsInByZWZlcnJlZF91c2VybmFtZSI6InNlcnZpY2UtYWNjb3VudC1tZXVzLWNvbnRhdG9zLWNsaWVudCIsImNsaWVudEFkZHJlc3MiOiIxNzIuMjUuMC4xIn0.dffKxEynYT94d6_z0pJy_gQfRLB582Ue9EC0TcrRD1UTRZlcCJHuC2gvwUzJGXBVJ4ysCh03t8UBpV-Ro-ovGlKZP-frbLa2VhA6sCIE4OlEBzoYmJv8Tvl3iLSBa-lW832K9orJZxajG0JHHcQTfWECro9R5N6CjmMhaiccqYOnd_deIALtMJEiKjwtTSBlFzEowIoi_S1nS_sh_5KAvHNe6zxoFZIcPdwg7U1zplcxe8HCnpNrfyuW7X8aZ7sVQW_SvM6g-e2m99b0aTR-8wjAKu6nq7bTyvxlPiJB6ljt4TR_WkCsYZpAdZIHLQlNeVwIxVSjt4PhcDyEI4EJ4g
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] ---> END HTTP (0-byte body)
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] <--- HTTP/1.1 200 (593ms)
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] cache-control: no-cache, no-store, max-age=0, must-revalidate
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] connection: keep-alive
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] content-type: application/json
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] date: Mon, 06 Jun 2022 14:35:30 GMT
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] expires: 0
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] keep-alive: timeout=60
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] pragma: no-cache
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] transfer-encoding: chunked
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] vary: Access-Control-Request-Headers
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] vary: Access-Control-Request-Method
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] vary: Origin
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] x-content-type-options: nosniff
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] x-frame-options: DENY
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] x-xss-protection: 1; mode=block
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] 
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] [{"id":1,"nome":"Yuri Matheus","empresa":"Zup"},{"id":9,"nome":"Steve Jobs","empresa":"Apple"},{"id":2,"nome":"Alberto Souza","empresa":"Zup"},{"id":3,"nome":"Jordi Silva","empresa":"Apple"},{"id":4,"nome":"Jackson","empresa":"Zup"},{"id":5,"nome":"Paula Santana","empresa":"Apple"},{"id":6,"nome":"Leonardo","empresa":"Zup"},{"id":7,"nome":"Rafa","empresa":"Apple"},{"id":8,"nome":"Miguel","empresa":"Facebook"}]
b.c.z.e.m.c.f.MeusContatosFeignClient    : [MeusContatosFeignClient#lista] <--- END HTTP (413-byte body)
o.s.w.c.HttpMessageConverterExtractor    : Reading to [java.util.List<br.com.zup.edu.meuscontatos.clients.feign.ContatoResponse>]
```

Os logs acima são referentes a primeira requisição ao Resource Server, mas a partir da segunda requisição para o Resource Server não vai haver outra requisição para o Authorization Server para obter o Access Token, ou seja, o token foi obtido na primeira requisição pelo interceptor e cacheada em memória, somente quando ele expirar é que veremos o interceptor tentando obter um novo token via fluxo Refresh Token do OAuth 2.0.

> ⚠️ **Lembre-se de desligar os logs em produção** <br/>
> Habilitar os logs em modo `DEBUG` do OpenFeign faz sentido somente em ambiente de desenvolvimento e testes, mas não em produção. Em produção os mesmos acabariam gerando muito I/O devido ao volume de logs produzidos que poderia levar a problemas de performance na aplicação, especialmente se estamos falando de distributed logging na rede.

Com os logs habilitados podemos ver o que acontece por debaixo dos panos e, em caso de problemas, podemos analisa-los e resolve-los de maneira mais assertiva possível. Sem estes logs seria MUITO dificil fazer troubleshooting na nossa aplicação.

## Configurando interceptor para um único OpenFeign client

Como comentamos acima, ter um interceptor global é prático e facilita nossas vidas quando temos diversos endpoints que apontam para o mesmo Authorization Server ou utilizam o mesmo fluxo OAuth 2.0. Contudo, algumas vezes sua aplicação ou microsserviço precisa acessar dois ou mais sistemas externos que não compartilham o mesmo fluxo OAuth 2.0 usando OpenFeign, e nesse caso utilizar um interceptor global é perigoso na perspectiva de segurança.

Para resolver isso, podemos configurar cada Feign client com sua configuração especifica, incluindo nosso interceptor. Para isso, basta declararmos nosso Feign client como abaixo:

```java
@FeignClient(
    name = "meusContatos",
    url = "http://localhost:8080/meus-contatos",
    configuration = MeusContatosClient.MyConfiguration.class // habilita interceptor para este client
)
public interface MeusContatosClient {

    @GetMapping("/api/contatos")
    public List<ContatoResponse> lista();

    /**
     * Configurações especificas para este Feign client
     **/
    class MyConfiguration {

        @Bean
        public OAuth2FeignRequestInterceptor oAuth2FeignRequestInterceptor(OAuth2AuthorizedClientManager clientManager) {
            return new OAuth2FeignRequestInterceptor(clientManager, "meus-contatos");
        }
    }
}
```

Perceba que a classe de configurações agora está dentro da interface `MeusContatosClient`, e que também adicionamos o atributo `configuration` na anotação `@FeignClient` para indicar qual configuração utilizar (essa feature do OpenFeign te permite sobrescrever todas as configurações de um Feign client). Um detalhe importante é que a classe `MyConfiguration` **não está anotada** com a anotação `@Configuration` do Spring Boot, esse é um detalhe importante para que ela não se torne global na aplicação.

Se você achar mais conveniente ou precisar reutilizar esta configuração com múltiplos Feign clients, você pode extrair a classe de configurações `MyConfiguration` para um arquivo separado no classpath, mas lembre-se de ter o cuidado para não anota-la com `@Configuration`.

## Links e referências

- [OAuth 2.0 Client Credentials Grant](https://oauth.net/2/grant-types/client-credentials/)
- [RFC6749: Client Credentials](https://datatracker.ietf.org/doc/html/rfc6749#section-1.3.4)
- [OAuth 2.0 Refresh Token](https://oauth.net/2/grant-types/refresh-token/)
- [Spring Security: Configuring Custom Provider Properties](https://docs.spring.io/spring-security/site/docs/5.3.2.RELEASE/reference/html5/#oauth2login-custom-provider-properties)
- [Spring Security: OAuth 2.0 Client](https://docs.spring.io/spring-security/site/docs/5.3.2.RELEASE/reference/html5/#oauth2client)
- [Spring Security: OAuth2AuthorizedClientRepository / OAuth2AuthorizedClientService](https://docs.spring.io/spring-security/site/docs/5.3.2.RELEASE/reference/html5/#oauth2Client-authorized-repo-service)
- [Spring Security: OAuth2AuthorizedClientManager / OAuth2AuthorizedClientProvider](https://docs.spring.io/spring-security/site/docs/5.3.2.RELEASE/reference/html5/#oauth2Client-authorized-manager-provider)
- [Spring Cloud OpenFeign](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/#spring-cloud-feign)
- [Spring Cloud: Release train Spring Boot compatibility](https://spring.io/projects/spring-cloud/)
- [Github: OpenFeign](https://github.com/OpenFeign/feign)
- [Baeldung: Introduction to Spring Cloud OpenFeign](https://www.baeldung.com/spring-cloud-openfeign)
- [Baeldung: Feign Logging Configuration](https://www.baeldung.com/java-feign-logging)
- [Baeldung: Provide an OAuth2 Token to a Feign Client](https://www.baeldung.com/spring-cloud-feign-oauth-token)
- [Sensible feign client configuration](https://blog.wick.technology/sensible-feign/)
- [OAuth2FeignRequestInterceptor: original implementation](https://github.com/loesak/spring-security-openfeign)

