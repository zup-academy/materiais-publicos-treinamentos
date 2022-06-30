# Introdução a JWT

Neste conteúdo teremos um overview geral sobre JWT, sua estrutura, o que ele é e como o mesmo pode ser utilizado dentro dos fluxos de autorização e obtenção de access tokens.

## O que é JWT?

JSON Web Token, ou JWT (pronuncia-se "jot"), é um formato padrão na industria para troca de informações seguras em ambientes que possuem alguma restrição de segurança. O mesmo possui sua [própria especificação](https://datatracker.ietf.org/doc/html/rfc7519), que define sua estrutura, atributos, semântica e detalhes, e já há alguns bons anos costuma ser a opção número um entre frameworks e tecnologias quando falamos de aplicações baseadas em tokens.

Apesar do mesmo se tratar de um JSON, que é uma estrutura textual aberta, nós podemos assinar e encriptar seu payload; um token JWT pode ser assinado tanto via private secret como chave pública/privada. Ao assinar o payload nós garantimos que o mesmo não possa ser adulterado durante seu trânsito entre aplicações, garantindo assim sua validade, e caso seja importante esconder as informações contidas num token JWT nós podemos encripta-lo.

Por ser um padrão conhecido e bastante adotado na industria, o mesmo também é suportado e utilizado por diversos protocolos de segurança, entre eles OAuth2 e OpenID Connect.

## Sumário de um token JWT

Entender a estrutura e como funciona um token JWT não é dificil. Para ter uma idéia, na imagem abaixo você pode ter um sumário de alto nível sobre tokens JWT:

![Sumário de alto nível de um token JWT](/seguranca-com-spring-security-e-oauth2/imagens/jwt-sumary.jpg)

Caso tenha interesse em aprofundar mais alguns detalhes sobre o JWT, basta ler o restante deste conteúdo.

## A estrutrura de um token JWT

Um token em formato JWT tem exatamente essa aparência (usamos quebras de linhas para facilitar a legibilidade):

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.
TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

Acredite ou não, existe uma estrutura JSON na string acima, porém a mesma está encodada em Base64 (na verdade uma variação da mesma:  Base64url Encoding) para facilitar seu transporte e assinatura entre serviços e browsers.

Embora este formato pareça não fazer muito sentido, ele é muito compacto e representa muito bem um conjunto de claims (informações), juntamente com uma assinatura utilizada para garantir sua autenticidade. Repare que cada linha é separada por um `.` (ponto), na qual divide o token em 3 partes: header, payload e signature.

> **O que são claims?** <br/>
> Claims são **definitions** ou **assertions** feitas sobre alguém ou algo. Alguns destes claims e seus significados fazem parte da especificação do JWT, enquanto outros são definididos pelo usuário ou desenvolvedor(a).
>
> A magia por trás do JWT é que ele padroniza certas claims que são úteis no contexto de operações comuns. Um exemplo de operação comum é estabelecer a identidade de alguém/algo. Portanto, uma das claims padrões encontradas no JWT é a `sub` (que significa "subject"), na qual serve para indentificar o usuário por exemplo que autorizou que uma aplicação execute uma ação em seu nome.   

Para decodificar um token JWT que esteja encodado em base64 podemos copia-lo e examina-lo no site [JWT.io](https://jwt.io/). O mesmo também nos permite gerar JWTs.

Agora, vamos ver cada uma destas partes que definem o token JWT acima:

### Header

A primeira parte, `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`, representa o header (cabeçalho) do token JWT. Ao decodifica-lo nós podemos ver a seguinte estrutura JSON:

```json
{
    "alg": "HS256",
    "typ": "JWT"
}
```

O header identifica o algoritimo (`alg`) usado para gerar a assinatura. Neste caso, `HS256` indica que o este token é assinado usando HMAC-SHA256.

Apesar de raros na prática, um token JWT inseguro (Unsecured JWT) pode ser gerado, na qual teria o header abaixo:

```json
{
    "alg": "none",
}
```

Por ser inseguro, ou seja, não há assinatura nem criptografia, o mesmo apresenta apenas 2 elementos, como abaixo:

```
eyJhbGciOiJub25lIn0.
eyJzdWIiOiJ1c2VyMTIzIiwic2Vzc2lvbiI6ImNoNzJnc2IzMjAwMDB1ZG9jbDM2M
2VvZnkiLCJuYW1lIjoiUHJldHR5IE5hbWUiLCJsYXN0cGFnZSI6Ii92aWV3cy9zZXR0aW5ncyJ9.
```

Perceba que existe um `.` (ponto) no fim da string, mas como não temos o elemento de signature ele fica vazio.

### Payload

A segunta parte, `eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9`, representa o payload (body) do token JWT. Ao decodifica-lo nós podemos ver a seguinte estrutura JSON:

```json
{
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true
}
```

O payload é a parte onde todas as informações interessantes do usuário são adicionadas, geralmente consta as informações que fazem mais sentido para as aplicações. Além disso, algumas claims padrões da especificação podem também ser adicionadas, ao todo existem 7 claims padrão na especificação do JWT, como podemos ver a seguir:

- `iss`: significa `issuer`, que representa a entidade que emitiu o token JWT, que no caso do OAuth 2.0 geralmente será o Authorization Server;
- `sub`: significa `subject`, que representa alguém ou algo, geralmente o usuário que autorizou a ação;
- `aud`: significa `audience`, que representa a audiência para qual esse token foi gerado. No fluxo OAuth 2.0, o Resource Server seria faria parte dessa audiência; 
- `exp`: significa `expiration` (time), que representa o tempo de vida deste token, ou seja, é uma data/hora que indica o momento exato na qual esse token se torna inválido;
- `nbf`: significa `not before` (time), que representa o oposto do da clim `exp`, isto é, ele indica o momento exato na qual este token passa a ser válido;
- `iat`: significa `issued at` (time), que representa o momento que este token foi emitido;
- `jti`: significa `JWT ID`, que representa a identidade deste token JWT, que é utilizada para garantir unicidade do token e evitar alguns tipos de ataques;

### Signature

A terceira parte, `TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ`, representa a signature (assinatura) do token JWT. Seu proposito é permitir que uma ou mais aplicações consigam estabelecer a autenticidade do token JWT.

Esta assinatura é calculada ao encodar o header e payload usando [Base64url Encoding RFC 4648](https://en.wikipedia.org/wiki/Base64#The_URL_applications) e concatenando as duas partes separadas por `.` (ponto). Essa string é então processada pelo algoritimo de criptografia definido no header, neste caso HMAC-SHA256. A implementação desse algoritimo para gerar a assinatura e token seria algo parecido com o pseudo-código abaixo:

```javascript
const signature = HMAC_SHA256(
  secret,
  base64urlEncoding(header) + '.' +
  base64urlEncoding(payload)
)

const token = base64urlEncoding(header)  + '.' + 
              base64urlEncoding(payload) + '.' + 
              base64urlEncoding(signature)
```

Certamente essa assinatura é uma das principais e mais úteis features da especificação JWT, pois ela combina um formato de dado com algoritimos de assinaturas bem conhecidos, permitindo assim que um token JWT seja o formato ideal para compartilhar dados de forma segura entre aplicações e serviços.

## JWT, OAuth 2.0 e OpenID Connect

Num fluxo de autorização OAuth2, o Client precisa obter um access token do Authorization Server antes de consumir os recursos hospedados no Resource Server. Esse access token retornado pelo Authorization Server e trocado entre Client e Resource Server é justamente o token JWT, que possui exatamente a estrutura discutida acima.

Para você ter uma idéia de como é um access token e seu tamanho gerado pelo Keycloak, vamos decodificar o token abaixo que foi emitido pelo Keycloak (quebras de linhas adicionadas para melhorar legibilidade):

```
eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJIQXNaYjlROEtvX0pEcnZ4N09XdXZWMnZpbS1BUno1ZFlZWHM0eC1FcDdnIn0.
eyJleHAiOjE2NTYwODY2NzgsImlhdCI6MTY1NjA4NjM3OCwianRpIjoiODljMjhlYzEtOGZhZi00NGU4LWJmMGYtMzdiYmQxNjIzZjBiIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDoxODA4MC9hdXRoL3JlYWxtcy9tZXVzLWNvbnRhdG9zIiwiYXVkIjoiYWNjb3VudCIsInN1YiI6IjZlNmY4ZDQyLWI3NDEtNGYwOS1iNWIxLWI2ZTZjZDY0ODgwZSIsInR5cCI6IkJlYXJlciIsImF6cCI6Im1ldS1pbnNvbW5pYSIsInNlc3Npb25fc3RhdGUiOiJjNDY1NTAyYi04ZmNkLTQzMGItODM1OC0wMDlmYmRkOTBiZjgiLCJhY3IiOiIxIiwicmVhbG1fYWNjZXNzIjp7InJvbGVzIjpbImRlZmF1bHQtcm9sZXMtbWV1cy1jb250YXRvcyIsIm9mZmxpbmVfYWNjZXNzIiwidW1hX2F1dGhvcml6YXRpb24iXX0sInJlc291cmNlX2FjY2VzcyI6eyJhY2NvdW50Ijp7InJvbGVzIjpbIm1hbmFnZS1hY2NvdW50IiwibWFuYWdlLWFjY291bnQtbGlua3MiLCJ2aWV3LXByb2ZpbGUiXX19LCJzY29wZSI6ImVtYWlsIHByb2ZpbGUiLCJzaWQiOiJjNDY1NTAyYi04ZmNkLTQzMGItODM1OC0wMDlmYmRkOTBiZjgiLCJlbWFpbF92ZXJpZmllZCI6ZmFsc2UsInByZWZlcnJlZF91c2VybmFtZSI6InJhZmFlbC5wb250ZSJ9.
FcTLl5LPmoEhBS0wzee9Zs7eJBVkQt4GvVB6ii1hdbFTfNk_e9YkH4o5FRMPeomLyrEjAs6SfaJum1JAnFWXRD09Wa00PmWSO8K2VufHSdSGFt8yz7Z0mNkocPjjrhDHXIZGEC2R1KmGZxZDGWhsDxEvZqpPivTLKkjcUQGqtPDao1sCsBS-_aXfodemsFKeC1dk8p-EJlJYwSpPA6aD9XufSDB6CgJjA2iFcYAHnH1pipzHlVOkWT3YH3Agubs9tG02jj6TkUaJ_GY-YEa_3Lw2IIYK0lZ43j9Rb54j-_dgNNf8IhM_zTisPE4lrjSVqdaLRvywZFzl1RuYDJHNCg
```

Se acessarmos o site [JWT.io](https://jwt.io/) e copiarmos o access token acima, teremos o seguinte resultado:

```json
/**
 * Header
 */
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "HAsZb9Q8Ko_JDrvx7OWuvV2vim-ARz5dYYXs4x-Ep7g"
}

/**
 * Payload
 */
{
  "exp": 1656086678,
  "iat": 1656086378,
  "jti": "89c28ec1-8faf-44e8-bf0f-37bbd1623f0b",
  "iss": "http://localhost:18080/auth/realms/meus-contatos",
  "aud": "account",
  "sub": "6e6f8d42-b741-4f09-b5b1-b6e6cd64880e",
  "typ": "Bearer",
  "azp": "meu-insomnia",
  "session_state": "c465502b-8fcd-430b-8358-009fbdd90bf8",
  "acr": "1",
  "realm_access": {
    "roles": [
      "default-roles-meus-contatos",
      "offline_access",
      "uma_authorization"
    ]
  },
  "resource_access": {
    "account": {
      "roles": [
        "manage-account",
        "manage-account-links",
        "view-profile"
      ]
    }
  },
  "scope": "email profile",
  "sid": "c465502b-8fcd-430b-8358-009fbdd90bf8",
  "email_verified": false,
  "preferred_username": "rafael.ponte"
}
```

Como podemos ver, além das claims padrões da especificação, o Keycloak adiciona diversas outras claims.

## Links e referências

Aqui são alguns links de artigos, referências oficiais e não oficiais que podem te ajudar no aprendizado e aprofundamento:

- [Wikipedia: JSON Web Token](https://en.wikipedia.org/wiki/JSON_Web_Token)
- [RFC 7519 - JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519)
- [JWT.io](https://jwt.io/)



