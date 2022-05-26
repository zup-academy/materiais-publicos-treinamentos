# Configurando uma aplicação Spring Boot como OAuth2 Resource Server

Nesse conteúdo veremos como podemos criar ou configurar uma aplicação Spring Boot com OAuth2 Resource Server. Para isso precisaremos adicionar e configurar a dependência do Spring Security no nosso projeto.

## Criando uma nova aplicação com Spring Initialzer

xxx

## Configurando uma aplicação

### 1. Adicione a dependência do Maven no projeto

A primeira coisa que temos que fazer é configurar nosso projeto Spring Boot com a dependência do Maven, neste caso estamos falando do **Spring Boot Starter OAuth2 Resource server**. Para isso, basta adicionar a dependência abaixo no seu `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

O artefato `spring-boot-starter-oauth2-resource-server` inclui a dependência `spring-security-oauth2-resource-server`, que por sua vez contem o suporte a Resource Server. Esta dependência também inclui as bibliotecas core do Spring Security. Por esse motivo, podemos remover a dependencia `spring-boot-starter-security` do nosso `pom.xml`. 

Se estiver numa IDE, lembre-se de recarregar as dependências do Maven.

### 2. Configure `application.yml` com informações do Authorization Server

O próximo passo é configurar as informações do Authorization Server (Keycloak) na nossa aplicação Spring Boot. Essa é a forma como estabelecemos uma relação de confiança entre Resource Server e Authorization Server, ou seja, precisamos desse passo para que o Resource Server consiga validar o Access Token assinado pelo Authorization Server.

Dado que temos um Keycloak local na nossa máquina rodando na porta `18080` e temos o Realm `meus-contatos` configurado, a configuração no `application.yml` seria semelhante a esta:

```yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:18080/realms/meus-contatos
```

Nesta configuração também estamos indicando ao Resource Server que o Access Token segue o formato JWT em vez de um formato opaco (Opaque Tokens).

Na configuração acima, nossa aplicação (Resource Server) tentará determinar (e fazer download) das chaves públicas usando os metadados fornecidos pelo Authorization Server. Contudo, algumas vezes por medidas de segurança este endpoint de metadados não está aberto publicamente ou mesmo na rede interna da empresa, neste cenário precisamos também indicar o endpoint das chaves publicas explicitamente através da propriedade `jwk-set-uri`:

```yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri : http://localhost:18080/realms/meus-contatos
          jwk-set-uri: http://localhost:18080/realms/meus-contatos/protocol/openid-connect/certs
```

A partir de agora a propriedade `issuer-uri` se torna opcional, contudo **entendemos como boa prática manter ambas as propriedades** pois assim o Spring Security pode utiliza-la como opçõa extra de segurança em alguns cenários.


### 3. Configure as regras de acesso (access rules)

Com as informações do Authorzation Server devidamente configuradas no `application.yml`, o próximo passo é configurar as regras de acesso a API REST da nossa aplicação (Resource Server).

A maneira mais simples e comum de declarar estas regras de acesso é via configuração Java. Para isso, basta criarmos a classe `ResourceServerConfig` que estende a classe `WebSecurityConfigurerAdapter` do Spring Security e, por fim, sobreescrever o método `configure(HttpSecurity)`, como podemos ver no código abaixo:

```java
@Configuration
public class ResourceServerConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
            .and()
                .oauth2ResourceServer()  
                    .jwt(); // atencao: necessario pois estamos sobrescrevendo a conf do application.yml
                ;
    }
}
```

Como estamos sobrescrevendo a configuração default do Spring Security, nós **precisamos declarar explicitamente que nosso aplicação deve se comportar como Resource Server** através da DSL `oauth2ResourceServer()`. É partir dessa DSL e do método `jwt()` que declaramos que esperamos um Access Token no formato JWT.

Como não estamos explicitamente passando a configuração que usaremos para validação do token JWT, o Spring Security continuará lendo estas informações do `application.yml`.

#### Configure as regras de acesso por endpoints

A configuração acima é suficiente para proteger a API REST da nossa aplicação, ou seja, ela está indicando que qualquer endpoint da nossa API REST está protegida e somente poderá ser acessada por uma request que possua um Access Token válido. Porém, é importante que especificar quais os Scopes necessários para consumir cada um dos endpoints da nossa API REST.

Dado que temos os seguintes endpoints na nossa API REST:

```
Listagem: GET  /contatos
Detalhes: GET  /contatos/{id}
Cadastro: POST /contatos
```

Poderíamos configurar as regras de acesso para os endpoints acima onde listar e detalhar um contato precisariam do Scope `contatos:read`, enquanto a cadastrar um novo contato precisaria do Scope `contatos:write`:

```java
@Configuration
public class ResourceServerConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers(HttpMethod.GET, "/api/contatos").hasAuthority("SCOPE_contatos:read")
                .antMatchers(HttpMethod.GET, "/api/contatos/**").hasAuthority("SCOPE_contatos:read")
                .antMatchers(HttpMethod.POST, "/api/contatos").hasAuthority("SCOPE_contatos:write")
                .anyRequest()
                    .authenticated()
            .and()
                .oauth2ResourceServer()  
                    .jwt(); // atencao: necessario pois estamos sobrescrevendo a conf do application.yml
                ;
    }
}
```

Repare que as authorities declaradas no método `hasAuthority` são os Scopes configurados no Keycloack. Para que o Spring Security reconheça estas authorities como Scopes, se faz necessário usar o prefixo `SCOPE_`.

#### Configure as regras de acesso por anotações

Por fim, vamos habilitar o controle de acesso via anotações. Também conhecido como **Expression-Based Access Control**, ele nos permite ter controle fino das regras de acesso a nível de métodos usando as anotações `@PreAuthorize`, `@PreFilter`, `@PostAuthorize` and `@PostFilter`. Este tipo de controle de acesso é útil para que possamos ter um controle mais fino das regras de acesso a nível de métodos de controllers, services etc.

Para isso, basta configurar nossa classe `ResourceServerConfig` com a anotação `@EnableGlobalMethodSecurity` e definir seu atributo `prePostEnabled` como `true`:

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResourceServerConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // ...
    }
}
```

Dessa forma, agora podemos usar anotações para ter um controle ainda mais fino das regras de acesso a um endpoint. No código abaixo anotamos um método do controller com a anotação `@PreAuthorize` juntamente com a regra `hasAuthority` para verificar se o Access Token da requisição possui um determinado Scope:

```java
@DeleteMapping(value = "/contatos/{id}")
@PreAuthorize("hasAuthority('SCOPE_contatos:write')")
public ProjectDto remove(@PathVariable Long id) {
    // ...
}
```

### 4. Inicie a aplicação e teste seus endpoints

Pronto, agora basta iniciar a aplicação e exercitar os endpoints da nossa API REST!

Aqui podemos usar qualquer ferramenta de cliente HTTP, como POSTman, Insomnia ou mesmo cURL para obter o Access Token e por fim consumir nossa API REST.

Mas primeiramente assuma toda a configuração no Keycloak tenha sido devidamente feita, como Realm, Users, Client e Scopes. Portanto, existe um Client `meus-contatos-client` configurado no nosso Realm `meus-contatos` no Keycloak; existe também um usuário `rafael.ponte` com senha `123`. E, por simplificidade, assuma que este Client também esteja configurado para trabalhar com o fluxo **Resource Owner Password Flow**, pois é um fluxo mais simples e fácil de simular para um desenvolvedor(a) backend. 

Dessa forma, conseguimos obter o Access Token com apenas uma requisição HTTP via comando cURL:

```sh
curl --location --request POST 'http://localhost:18080/realms/meus-contatos/protocol/openid-connect/token' \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'grant_type=password' \
  --data-urlencode 'username=rafael.ponte' \
  --data-urlencode 'password=123' \
  --data-urlencode 'client_id=meus-contatos-client' \
  --data-urlencode 'client_secret=1tgU88Pko1bpQ8LtHz7pK7tAqnYIRNhn' \
  --data-urlencode 'scope=contatos:read contatos:write'
```

Como resposta a solicitação de token, o Keycloak nos retorna um JSON com os tokens solicitados, incluindo nosso Access Token e suas demais informações, como abaixo:

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJrN05tUzlLaXJQRVZNMk9rdTdIakVIRElyZDNoOHQySTc5eERsM3hXalJrIn0.eyJleHAiOjE2NTE2MDY5OTksImlhdCI6MTY1MTYwNjY5OSwianRpIjoiNmFjN2Y0YzQtMzA2NS00ODU5LWI4NmUtMWE5NzU0M2EyYzJhIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDoxODA4MC9hdXRoL3JlYWxtcy9ib290Y2FtcC1zZW5pb3JzIiwiYXVkIjoiYWNjb3VudCIsInN1YiI6IjUzYjljOWNkLTcyZjktNGNjNC05NzhhLTg5MzIwM2YxMzQ3ZiIsInR5cCI6IkJlYXJlciIsImF6cCI6Im1ldS1pbnNvbW5pYSIsInNlc3Npb25fc3RhdGUiOiIzZGVjNzVmYi0yYzg2LTRkYWItOTNkMi02ZTdkNDA0ZTA3MGEiLCJhY3IiOiIxIiwicmVhbG1fYWNjZXNzIjp7InJvbGVzIjpbIm9mZmxpbmVfYWNjZXNzIiwidW1hX2F1dGhvcml6YXRpb24iLCJkZWZhdWx0LXJvbGVzLWJvb3RjYW1wLXNlbmlvcnMiXX0sInJlc291cmNlX2FjY2VzcyI6eyJhY2NvdW50Ijp7InJvbGVzIjpbIm1hbmFnZS1hY2NvdW50IiwibWFuYWdlLWFjY291bnQtbGlua3MiLCJ2aWV3LXByb2ZpbGUiXX19LCJzY29wZSI6ImZ1bmNpb25hcmlvczpyZWFkIGZ1bmNpb25hcmlvczp3cml0ZSBlbWFpbCBwcm9maWxlIiwic2lkIjoiM2RlYzc1ZmItMmM4Ni00ZGFiLTkzZDItNmU3ZDQwNGUwNzBhIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsIm5hbWUiOiJSYWZhZWwgUG9udGUiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJyYWZhZWwucG9udGUiLCJnaXZlbl9uYW1lIjoiUmFmYWVsIiwiZmFtaWx5X25hbWUiOiJQb250ZSIsImVtYWlsIjoicmFmYWVsLnBvbnRlQHp1cC5jb20uYnIifQ.sR7E7qfXtTktIMNvNIOR_SOI2EiEmeYCAVzMkOjlPGVRZUG1GyqTFhGz5DRBHabTj0g-G5MuUrhcrtO2oHH2EM54ULv4ED5l6p_9bF2aczIB5igqskKlIwietpHLnxmIMzm32qDMBIjTnlIlwdJzTH0QzRNQ7vSHS-5pJm55nEV0RrY0QAdao3fBJ3Z8_Loh5FNJxFefZlbo0SxzLQSkZoawUO-cmLl3kQxIzBH_WiNjbxef-FX_OpIIPg-kB82Bkyhwjo1l_9eO2P2uNxepuftVhLTaAQsW3H15c9MH6jxJQGxOUX-QP9bVlpHWuStuhHzrC6pUBMfIv2Bsx3auzw",
    "expires_in": 300,
    "refresh_expires_in": 1800,
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJmMjMzMDI3ZC1lZmExLTQ1YzUtYWQzNy0xMzU5NmJiODQ1ZTAifQ.eyJleHAiOjE2NTE2MDg0OTksImlhdCI6MTY1MTYwNjY5OSwianRpIjoiNDk3NDgzMmItMDJmZS00MTE5LWE0YWUtMjNkN2JlYmNhOWM4IiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDoxODA4MC9hdXRoL3JlYWxtcy9ib290Y2FtcC1zZW5pb3JzIiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdDoxODA4MC9hdXRoL3JlYWxtcy9ib290Y2FtcC1zZW5pb3JzIiwic3ViIjoiNTNiOWM5Y2QtNzJmOS00Y2M0LTk3OGEtODkzMjAzZjEzNDdmIiwidHlwIjoiUmVmcmVzaCIsImF6cCI6Im1ldS1pbnNvbW5pYSIsInNlc3Npb25fc3RhdGUiOiIzZGVjNzVmYi0yYzg2LTRkYWItOTNkMi02ZTdkNDA0ZTA3MGEiLCJzY29wZSI6ImZ1bmNpb25hcmlvczpyZWFkIGZ1bmNpb25hcmlvczp3cml0ZSBlbWFpbCBwcm9maWxlIiwic2lkIjoiM2RlYzc1ZmItMmM4Ni00ZGFiLTkzZDItNmU3ZDQwNGUwNzBhIn0.JZ2-f0sJGR_6cYmUqNiwILuvIIPOtHxYsaoR4Bu1dTE",
    "token_type": "Bearer",
    "not-before-policy": 0,
    "session_state": "3dec75fb-2c86-4dab-93d2-6e7d404e070a",
    "scope": "contatos:read contatos:write email profile"
}
```

Agora, para acessar qualquer endpoint da nossa API REST, basta submeter uma requisição com o header `Authorization` (Bearer) contendo o Access Token obtido do Keycloak na requisição anterior:

```sh
curl --request GET \
  --url 'http://localhost:8080/api/contatos/3' \
  --header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia....v2Bsx3auzw'
```

Show de bola, como resposta temos um `200 (OK)` com o seguinte payload:

```json
{
	"id": 3,
	"nome": "Jordi Silva",
	"empresa": "Zup",
	"criadoPor": "rafael.ponte",
	"telefones": [
		{
			"id": 7,
			"tipo": "celular",
			"numero": "+5511977777777"
		},
	]
}
```

Não foi tão dificil, não é mesmo?

## Links e referências
