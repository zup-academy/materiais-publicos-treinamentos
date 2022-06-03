# Consumindo uma API REST protegida por OAuth2 via WebClient

Nesse conteúdo veremos como podemos consumir uma API REST protegida por OAuth 2.0 de um sistema externo utilizando o cliente HTTP reativo `WebClient` do Spring Boot.

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

### 4. Crie uma instância de WebClient

Precisamos de um simples HTTP client para consumir a API REST do Resource Server (sistema Meus Contatos). Mas em vez de usarmos uma biblioteca de HTTP client qualquer, utilizaremos o HTTP client reativo do Spring Boot, que se tornou padrão no Spring 5, o **WebClient**.  

A vantagem de usar o WebClient aqui é que não precisaremos nos preocupar de solicitar o token ao Authorization Server, armazena-lo em um espaço de memória compartilhado e em seguida envia-lo como cabeçalho HTTP em cada requisição ao Resource Server. Este processo acontecerá automaticamente e de forma totalmente transparente para nós.

Para isso, precisamos adicionar as dependências mínimas do WebClient no nosso `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webflux</artifactId>
</dependency>
<dependency>
    <groupId>io.projectreactor.netty</groupId>
    <artifactId>reactor-netty</artifactId>
</dependency>
```

Embora o WebClient faça parte da stack Reativa (Reactive) do Spring, nós podemos utiliza-lo em uma stack Servlet sem problemas.

Agora, para obter uma instância de `WebClient`, podemos simplesmente cria-la via `WebClient.Builder`. Para isso, basta tilizar o builder para criar e customizar nosso objeto `WebClient` como no código abaixo:

```java
@Configuration
class ClientSecurityConfig {

    @Bean
    public WebClient webClient() {
        return WebClient.builder()
                    .build();
    }

}
```

A partir de agora, podemos injeta-lo em qualquer outro bean gerenciado pelo container do Spring. Por exemplo, vamos criar uma classe que **represente nosso serviço cliente (consumidor)** e em seguida injetar a instância de `WebClient` configurada para que assim possamos consumir a API REST do sistema Meus Contatos:

```java
@Service
class MeusContatosClient {

    @Autowired
    private WebClient client;

    public List<ContatoResponse> lista() {

        String meusContatosUri = "http://localhost:8080/meus-contatos/api/contatos";

        List<ContatoResponse> contatos = webClient.get()
                .uri(meusContatosUri)
                .retrieve()
                .bodyToMono(new ParameterizedTypeReference<List<ContatoResponse>>() {})
                .block();

        return contatos;
    } 
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

Perceba que invocamos o método `block()` do `WebClient` justamente para que ele funcione de forma blocante e não reativo. Em uma aplicação que segue o paradigma reativo este passo não se faz necessário e certamente seria até desaconselhado.

> **Por que não usar o `WebClient` diretamente num controller?** <br/>
> Você percebeu que criamos uma nova classe (`MeusContatosClient`) para só então injetar e usar a instância de `WebClient`?
> 
> Apesar de podermos usar o `WebClient` diretamente em qualquer bean do Spring, como um controller ou service por exemplo, nós entendemos que **criar uma camada de indireção (camada intermediária de abstração) entre o controller e `WebClient` sejá uma boa prática**.
>
> Essa nova classe de serviço funciona como uma interface pública (API) com métodos e tipos bem definidos. Dessa forma, não só podemos reutiliza-la em outras classes e camadas, como também encapsulamos os detalhes de implementação e minimizamos o acoplamento entre as camadas; de quebra, podemos testa-la de forma integrada e ainda facilitamos o uso de mocks ao escrever testes automatizados quando ela for uma dependência de outra classe.

Nosso HTTP client está configurado e pronto para uso, porém ele ainda não está completo, pois se tentarmos utiliza-lo para acessar endpoint do sistema Meus Contatos nós receberemos um erro com Status HTTP `401 (Unauthorized)`, afinal não estamos repassando nenhum Access Token na requisição, não é mesmo?

Portanto, vamos dar um jeito de repassar este token em cada uma das requisições 😉

### 5. Configure o `WebClient` para trabalhar com fluxo OAuth 2.0

Para que nosso `WebClient` consiga se comunicar de maneira autenticada/autorizada com o Resource Server, precisamos configurá-lo para que ele leve o fluxo OAuth 2.0 em consideração.

**A idéia aqui é obter um Access Token válido do Authorization Server e em seguida adiciona-lo como cabeçalho HTTP (`Authorization: Bearer <token>`) em cada requisição enviada para o Resource Server**. Para não termos que implementar esse workflow manualmente, precisamos registrar um interceptor no `WebClient` que cuidará exatamente destes passos para nós.

Antes de tudo, isso significa criar e configurar uma instância de `OAuth2AuthorizedClientManager`, que é o objeto responsável por gerenciar todo o workflow de solicitação de tokens para o Authorization Server assim como gerenciar o ciclo de vida destes tokens.

Para isso, na nossa classe de configurações `ClientSecurityConfig`, vamos declarar um novo métoddo de fábrica para criar nosso `OAuth2AuthorizedClientManager` configurado para Client Credentials Flow, como abaixo:

```java
@Configuration
class ClientSecurityConfig {

    // ...

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

Este objeto `OAuth2AuthorizedClientManager` pode ser utilizado para obter os tokens manualmente se assim desejarmos, pois ele é quem de fato cuida de 90% deste workflow. Mas por se tratar de um workflow que se repetirá para cada requisição enviada, faz todo sentido que ele seja encapsulado num interceptor e configurado na nossa instância do `WebClient`.

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


Por fim, precisamos configurar o `WebClient` para trabalhar com o fluxo OAuth 2.0 esperado pelo Authorization Server e Resource Server. Para isso, ainda na classe `ClientSecurityConfig`, precisamos **configurar um interceptor** diretamente na nossa instância de `WebClient`, no momento de sua criação:

```java
@Configuration
class ClientSecurityConfig {

    @Bean
    public WebClient webClient(OAuth2AuthorizedClientManager manager) {

        ServletOAuth2AuthorizedClientExchangeFilterFunction oauth2
                = new ServletOAuth2AuthorizedClientExchangeFilterFunction(manager);
        oauth2.setDefaultClientRegistrationId("meus-contatos");

        return WebClient.builder()
                .apply(oauth2.oauth2Configuration()) // configura o interceptor
                .build();
    }

    // ...
}
```

Repare que injetamos a nossa instância configurada de `OAuth2AuthorizedClientManager` para passa-la como dependência para nosso interceptor `ServletOAuth2AuthorizedClientExchangeFilterFunction`. **Este interceptor se encarregará de obter um token válido antes de cada requisição disparada ao Resource Server**. Em adição a isto, nós também indicamos ao interceptor qual registro de client padrão (default) ele deve utilizar para se comunicar com o Authorization Server, que no nosso caso é justamente o registro `meus-contatos` que configuramos no `application.yml` anteriormente.

> ⚠️ **Cuidado com `WebClient` global na aplicação** <br/>
> Nós criamos e declaramos nossa instância de `WebClient` no contexto do Spring, desta forma todos as requisições enviadas receberão o Access Token obtido do Authorization Server. Apesar de prático, devemos ter cuidado ao utilizar esta instância em funcionalidades que não participam de um fluxo OAuth 2.0, caso contrário podemos expor o Access Token para serviços externos, o que abre brechas sérias de segurança.
>
> Se você tem 2 ou mais serviços externos que sua aplicação precisa se integrar, lembre-se de revisar o uso do seu `WebClient`. Nestes cenários, você provavelmente terá instâncias distintas do `WebClient` para cada uma delas.

Pronto! A partir de agora nosso `WebClient` consegue se comunicar com a API REST protegida do Resource Server (Meus Contatos). Isso acontece pois o interceptor vai obter um token válido do Authorization Server e em seguida adiciona-lo ao cabeçalho HTTP de todas as requisições enviadas ao Resource Server.

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

Perceba que injetamos nosso serviço cliente `MeusContatosClient` em vez do `WebClient` diretamente, e o usamos para consumir o sistema externo através do seu método `lista()`, o resto do código basicamente filtra a coleção de contatos pela empresa informada e transforma cada contato em um DTO de saída do nosso controller, no caso `ContatoFiltradoResponse`.

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

Nosso código do controller utilizou a classe `MeusContatosClient` para  invocar o `WebClient` que fez toda comunicação com o serviço externo. Por debaixo dos panos nosso interceptor e configuração de fluxo OAuth 2.0 entrou em ação obtendo um Access Token do Authorization Server, adicionando-o no cabeçalho da requisição e, por fim, enviando a requisição para o Resource Server.

Apesar de toda configuração do Spring Security e conhecimento sobre fluxos OAuth 2.0, não é uma tarefa dificil assim, não é mesmo?

## Habilite os logs do `WebClient`

Embora nosso código tenha funcionado como esperado não te incomoda saber que toda mágica ocorreu por debaixo dos panos?

O problema não é a mágica acontecer, mas sim não podermos visualiza-la enquanto desenvolvemos e testamos nosso código. Visualizar o que acontece  **através de logs** é muito importante para fazermos troubleshooting (analisar, identificar e corrigir erros) quando as coisas parecem não funcionar como esperado.

A forma mais simples de habilitar os logs do `WebClient` é adicinando as seguintes linhas no `application.yml` da nossa aplicação:

```yml
logging: 
  level:
    org.springframework.web.client: DEBUG
    org.springframework.web.reactive.function.client: DEBUG
```

Apenas com estas linhas temos uma boa idéia das requisições HTTP enviadas pelo `WebClient`, o que já é bastante útil para resolver muitos tipos de problemas, mas infelizmente **não temos detalhes sobre os headers e body da requisição**. Para habilitar o logging dos headers e body precisamos ir mais a fundo na configuração do `WebClient`.

Dessa forma, vamos configurar os logs do `WebClient` com suporte a headers e body para entendermos o que acontece por trás das cenas. Por o `WebClient` ser uma abstração de alto nível, ele utiliza outras bibliotecas reativas (non-blocking) de comunicação HTTP como implementação que são chamadas de **Client Http Connectors**, por esse motivo precisamos habilitar os logs destas implementações.

Como estamos utilizando `WebClient` configurado com o client connector **Netty** (dependência `reactor-netty` no `pom.xml`), nós podemos habilitar seu log através do método `wiretap` de sua classe `HttpClient` (que é de fato nosso connector). Portanto, na classe `ClientSecurityConfig` basta configurar o `HttpClient` e passa-lo para nosso `WebClient`, como abaixo:

```java
@Configuration
class ClientSecurityConfig {

    @Bean
    public WebClient webClient(OAuth2AuthorizedClientManager manager) {

        ServletOAuth2AuthorizedClientExchangeFilterFunction oauth2
                = // ...;

        HttpClient httpClient = HttpClient.create()
                                .wiretap(
                                    "reactor.netty.http.client.HttpClient", 
                                    LogLevel.DEBUG, 
                                    AdvancedByteBufFormat.TEXTUAL
                                );

        return WebClient.builder()
                .apply(oauth2.oauth2Configuration())
                .clientConnector(new ReactorClientHttpConnector(httpClient)) // configura connector com logging habilitado
                .build();
    }

    // ...
}
```

Por fim, precisamos habilitar o log em modo `DEBUG` para categoria que configuramos no `wiretap`, neste caso `reactor.netty.http.client.HttpClient`. Para isso, no seu `application.yml` adicione a seguinte linha de logging:

```yml
logging: 
  level:
    reactor.netty.http.client: DEBUG
```

> ⚠️ **Lembre-se de desligar os logs em produção** <br/>
> Habilitar os logs em modo `DEBUG` do `WebClient` faz sentido somente em ambiente de desenvolvimento e testes, mas não em produção. Em produção os mesmos acabariam gerando muito I/O devido ao volume de logs produzidos que poderia levar a problemas de performance na aplicação, especialmente se estamos falando de distributed logging na rede.

A partir de agora ao rodarmos a aplicação e tentarmos nos comunicar com o Resource Server veremos algumas linhas de log semelhantes a estas:

```log
[oundedElastic-1] o.s.web.client.RestTemplate              : HTTP POST http://localhost:18080/auth/realms/meus-contatos/protocol/openid-connect/token
[oundedElastic-1] o.s.web.client.RestTemplate              : Accept=[application/json, application/*+json]
[oundedElastic-1] o.s.web.client.RestTemplate              : Writing [{grant_type=[client_credentials], scope=[contatos:read]}] as "application/x-www-form-urlencoded;charset=UTF-8"
[oundedElastic-1] o.s.web.client.RestTemplate              : Response 200 OK
[oundedElastic-1] o.s.web.client.RestTemplate              : Reading to [org.springframework.security.oauth2.core.endpoint.OAuth2AccessTokenResponse] as "application/json"
[oundedElastic-1] o.s.w.r.f.client.ExchangeFunctions       : [7ff49fb7] HTTP GET http://localhost:8080/meus-contatos/api/contatos
[ctor-http-nio-3] reactor.netty.http.client.HttpClient     : [cd8f0fe6] REGISTERED
[ctor-http-nio-3] reactor.netty.http.client.HttpClient     : [cd8f0fe6] CONNECT: localhost/127.0.0.1:8080
[ctor-http-nio-3] reactor.netty.http.client.HttpClient     : [cd8f0fe6, L:/127.0.0.1:11497 - R:localhost/127.0.0.1:8080] ACTIVE
[ctor-http-nio-3] r.netty.http.client.HttpClientConnect    : [cd8f0fe6-1, L:/127.0.0.1:11497 - R:localhost/127.0.0.1:8080] Handler is being applied: {uri=http://localhost:8080/meus-contatos/api/contatos, method=GET}
[ctor-http-nio-3] reactor.netty.http.client.HttpClient     : [cd8f0fe6-1, L:/127.0.0.1:11497 - R:localhost/127.0.0.1:8080] WRITE: 1462B GET /meus-contatos/api/contatos HTTP/1.1
user-agent: ReactorNetty/1.0.18
host: localhost:8080
accept: */*
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICIwR085RC1BTlhzU3pwWHBENlN2cm9Ca2tEaUlIb0otRnBiSkgxOTNnOUNVIn0.eyJleHAiOjE2NTQyNjY1MjgsIml...
```

Os logs acima são referentes a primeira requisição ao Resource Server, mas a partir da segunda requisição para o Resource Server não vai haver outra requisição para o Authorization Server para obter o Access Token, ou seja, o token foi obtido na primeira requisição pelo interceptor e cacheada em memória, somente quando ele expirar é que veremos o interceptor tentando obter um novo token via fluxo Refresh Token do OAuth 2.0.

Com os logs habilitados podemos ver o que acontece por debaixo dos panos e, em caso de problemas, podemos analisa-los e resolve-los de maneira mais assertiva possível. Sem estes logs seria MUITO dificil fazer troubleshooting na nossa aplicação.

## Links e referências

- []()
- []()
- []()
- []()
- []()

