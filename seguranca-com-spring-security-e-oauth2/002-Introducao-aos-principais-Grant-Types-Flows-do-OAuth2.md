# Introdução aos principais Grant Types/Flows do OAuth2

Neste material vamos aprender sobre os principais fluxos de autorização (grant types) do OAuth2 e como escolhê-los de acordo com o caso de uso e cenário.

## Conhecendo os Grant Types do OAuth2

Tecnicamente, temos **4 tipos de fluxos de autorização (grant types)** que podem ser usados para implementar autorização com OAuth2:

1. **Authorization Code flow**;
2. **Client Credentials flow**;
3. **Implicit flow**;
4. **Resource Owner Password Credentials flow**;

Embora estes 4 fluxos estejam definidos na especificação do OAuth2, **a orientação atual desencoraja o uso dos fluxos Resource Owner Password Credentials flow e Implicit flow**. Por esse motivo, neste material iremos focar exclusivamente nos fluxos que sobraram: Authorization Code flow e Client Credentials flow; além disso, na perspectiva do desenvolvedor(a) backend não importa muito qual o fluxo utilizado, afinal de contas ele(a) geralmente ficará responsável de implementar o Resource Server, como uma API REST, que tem apenas interesse no access token.

### Fluxos auxiliares e extensões

Além destes 4 principais fluxos, a especificação possui outros fluxos auxiliares e também extensões que melhoram ou corrigem os fluxos existentes:

- [**PKCE**](https://oauth.net/2/pkce/): definida mais recentemente na especificação, ela significa Proof Key for Code Exchange, e tem como objetivo endereçar limitações no fluxo padrão Authorization Code; a sigla é pronunciada como "Pixie";
- [**Refresh Token**](https://oauth.net/2/grant-types/refresh-token/): fluxo utilizado para "prolongar" o tempo de vida de um access token, ou seja, permitindo que Clients troquem um refresh token por um novo access token quando o access token estiver expirado;
- [**Device Code**](https://oauth.net/2/grant-types/device-code/): utilizado para permitir que dispositivos que não possuem browser ou possuem restrição de entrada a obterem um access token, como aplicativos de SmartTVs, consoles de video-game, impressoras, fitness trackers etc;

## Qual fluxo utilizar? 

Existem 2 pontos determinantes na hora de decidir qual fluxo OAuth2 utilizar e como implementa-los:

- Client confidentiality
- Quem é o Resource Owner?

**Client Confidentiality** tem a ver com entender como um client lida com confidencialidade. Quando falamos de confidencialidade estamos nos referindo a capacidade deste client de guardar suas credenciais de acesso de forma privada e segura.

Lembra que comentamos que existe uma relação de confiança entre os atores/pápeis no OAuth2? Pois bem, um Client precisa estar registrado no Authorization Server para que o mesmo consiga obter um access token e em seguida acessar os dados no Resource Server. E é justamente nesse momento de registrar o Client no Authorization Server que configuramos seu nível de confidencialidade: ao configurá-lo como Confidential Client ele recebe uma **Client Secret** (como uma senha) que deve ser enviada ao Authorization no momento de solicitar um access token. 

Baseado nisso, a especificação define 2 tipos de clients (Client Types):

- **Confidential clients**: esse tipo de client consegue armazenar suas credenciais (Client Secret) de forma privada e segura, aqui temos aplicações server-side onde possui acesso limitado e protegido na rede;
- **Public clients**: em contrapartida esse client **não** consegue guardar a Client Secret de forma segura e privada, como aplicativos mobile ou mesmo SPAs (Single Page Applications) que rodam no browser do usuário;

Como você pode perceber, aplicações que rodam no browser ou aplicativos nativos mobile não podem ser considerados como confidential clients simplesmente porque eles são executados em um ambiente na qual não temos o devido controle.

Outro fator determinante é o saber **quem assume o papel de Resource Owner** na nossa arquitetura. Geralmente o Resoure Owner será o próprio usuário final (end-user), isto é, uma pessoa.

Por exemplo, o Github possui informações sobre mim, meu perfil e informações dos meus repositórios. Tudo isso são recursos, e aqui eu sou o dono destes recursos. Por esta razão eu posso conceder acesso para alguns ou todos os recursos.

Mas em muitos casos o Resource Owner é o próprio client (aplicação). Isto é bem comum em serviços que não possuem a interação de um usuário, como serviços e aplicações que rodam em background, que fazem processamento em lote, microsserviços que se integram diretamente com outros serviços etc. Aqui o serviço assume o papel de Client e Resource Owner ao mesmo tempo.

## Authorization Code flow

Certamente o fluxo mais comum e seguro do OAuth2, ele pode ser utilizado tanto para o Confidential Clients como para Public Clients, no entanto ele foi otimizado para Confidential Clients. A diferença é que Public Clients não usarão a Client Secret para validar a identidade do Client por se tratar de uma informação sensível.

Por se tratar de um fluxo baseado em redirectionamentos (HTTP Redirects), o Client deve ser capaz de interagir com User-Agent do Resource Owner, que geralmente é um browser. E por meio do browser que o Resource Owner se autentica com o Authorization Server para conceder acesso limitado dos recursos ao Client. Para entender melhor este fluxo, vamos ver a figura abaixo:

```
     +----------+
     | Resource |
     |   Owner  |
     |          |
     +----------+
          ^
          |
         (B)
     +----|-----+          Client Identifier      +---------------+
     |         -+----(A)-- & Redirection URI ---->|               |
     |  User-   |                                 | Authorization |
     |  Agent  -+----(B)-- User authenticates --->|     Server    |
     |          |                                 |               |
     |         -+----(C)-- Authorization Code ---<|               |
     +-|----|---+                                 +---------------+
       |    |                                         ^      v
      (A)  (C)                                        |      |
       |    |                                         |      |
       ^    v                                         |      |
     +---------+                                      |      |
     |         |>---(D)-- Authorization Code ---------'      |
     |  Client |          & Redirection URI                  |
     |         |                                             |
     |         |<---(E)----- Access Token -------------------'
     +---------+       (w/ Optional Refresh Token)
```

O fluxo ilustrado na figura acima inclui os seguintes passos:

- (A) O client inicia o fluxo encaminhando o browser do resource owner (end-user) para o endpoint de autorização. O client inclui seu ID, escopos (permissões), local state e uma URI de redirecionamento para qual o authorization server enviará de volta o browser uma vez que o acesso for concedido ou negado;

- (B) O authorization server autentica o resource owner via browser e estabelece se o resource owner concede ou nega acesso ao client. Aqui é onde o resource owner entra com suas credenciais (username e password) e verifica se as permissões solicitadas pelo client fazem sentido para o que ele precisa fazer ou não;

- (C) Assumindo que o resource owner tenha concedido acesso, o authorization server redireciona o browser de volta para o client usando a URI de redirecionamento fornecida anteriormente. Esta URI de redirecionamento inclui o authorization code e qualquer local state fornecido pelo client anteriormente;

- (D) Com o authorization code em mãos, o client envia uma requisição para o endpoint de emissão de tokens do authorization server para obter um access token, incluindo o authorization code e a mesma URI de redirecionamento usada no passo anterior. É justamente nesse passo que o client tenta se autenticar com o authorization server para obter o access token;

- (E) O authorization server autentica o client, valida seu authorization code e assegura que a URI de redirecionamento recebida confere com a URI usada para redirecionar o client no passo (C). Se tudo estiver válido, o authorization server responde de volta com um access token e, opcionalmente, um refresh token;

> **PKCE Extension** <br/>
> O PKCE, ou Proof Key for Code Exchange extension, pode ser utilizada sobre o Authorization Code Flow.
> 
> Embora ele tenha sido concebido para endereçar possíveis problemas de interceptação para Public Clients, **ele é uma prática recomendada para qualquer tipo de Client** pois ela representa uma melhoria extra de segurança.
>
> A idéia básica do PKCE é o client criar uma chave de uso único (one-time key) e utiliza-la no processo de autorização para que o authorization server possa verificar sua autenticidade. Isso significa que o authorization server precisa fornecer essa feature. 

Se você percebeu, este fluxo precisa obrigatoriamente da interação do usuário (end-user) no processo de autorização. Se não existe a figura do usuário então o Authorization Code flow não se aplica.

## Client Credentials flow

Um caso especial na qual o Authorization Code flow não se aplica é  quando não existe um usuário final (end-user) envolvido no processo de autorização, ou seja, quando o processo lida apenas com comunicação **Machine-to-Machine (M2M)**. Para este cenário nós usamos justamente o Client Credentials flow.

Aqui o client também assume o papel de resource owner. Este é o tipo de cenário comumente encontrado em tarefas agendadas, processamentos em lotes, jobs em background ou comunicação direta entre microsserviços. Não à toa, **este fluxo deve obrigatoriamente ser utilizado somente para Confidential Clients**.

Este fluxo é mais simples comparado ao Authorization Code flow pois o serviço pode obter diretamente o access token apresentando suas credenciais (client ID e client secret), como podemos ver na figura a seguir:

```
     +---------+                                  +---------------+
     |         |                                  |               |
     |         |>--(A)- Client Authentication --->| Authorization |
     | Client  |                                  |     Server    |
     |         |<--(B)---- Access Token ---------<|               |
     |         |                                  |               |
     +---------+                                  +---------------+
```

Os seguintes passos acontecem:

- (A) O client se autentica com o authorization server informando suas credenciais e solicita um access token utilizando o endpoint de emissão de tokens;

- (B) O authorization server autentica o client, e se válido, emite um access token de volta;

Sempre que precisar de um fluxo de autorização onde não existe a figura do usuário (end-user), ou seja, numa comunicação máquina-para-máquina (M2M) você poderá utilizar o fluxo Client Credentials. 

## Resource Owner Password Credentials flow

Este fluxo é uma maneira de trocar as credenciais do usuário por um access token. Por o client ter que coletar o username e password do usuário e em seguida enviar para o authorization server este fluxo **não é mais recomendado** pela especificação e guia de boas práticas de segurança no uso do protocolo OAuth2.

Entre suas limitações, não é possível implementar mecanismos como 2FA (autenticação de dois fatores).

Contudo, o mesmo ainda pode ser útil em cenários onde o resource owner (end-user) tem uma forte relação de confiança com o client, como o aplicativo do seu banco, ou para dispositivos e aplicações altamente privilegiados; o mesmo também pode ser interessante para empresas com sistemas legados que estejam migrando gradualmente suas aplicações e serviços para o OAuth2.

> **Pode user útil em ambiente de desenvolvimento** <br/>
> Apesar deste fluxo não ser recomendado para uso em produção, o mesmo ainda pode ser útil em tempo de desenvolvimento e testes locais, isso porque o mesmo se trata de um fluxo mais simples de utilizar e mais comum para o desenvolvedor(a).

Segue imagem com os passos deste fluxo:

```
     +----------+
     | Resource |
     |  Owner   |
     |          |
     +----------+
          v
          |    Resource Owner
         (A) Password Credentials
          |
          v
     +---------+                                  +---------------+
     |         |>--(B)---- Resource Owner ------->|               |
     |         |         Password Credentials     | Authorization |
     | Client  |                                  |     Server    |
     |         |<--(C)---- Access Token ---------<|               |
     |         |    (w/ Optional Refresh Token)   |               |
     +---------+                                  +---------------+
```

- (A) O resource owner fornece seu username e password no client;

- (B) O client envia uma requisição para o endpoint de emissão de tokens do authorization server para obter um access token, incluindo na requisição as credenciais recebidas do resource owner;

- (C) O authorization server autentica o client e valida as credenciais do resource owner, e se válido, emite um access token de volta;

No fim, você somente deveria utiliza-lo se não for possivel utilizar outro fluxo OAuth2, como Authorization Code ou Client Credentials por exemplo.

## Implicit flow

O Implicit flow era um fluxo simplificado do OAuth recomendado para aplicativos mobile nativos e aplicações JavaScript (como SPAs) onde o access token era retornado imediatamente sem o passo extra de troca de authorization code.

Devido aos riscos inerentes de retornar o access token via HTTP Redirect sem qualquer confirmação de recebimento por parte client **seu uso é desencorajado** pela especificação e guia de boas práticas de segurança do OAuth2. Por esta razão, é recomenado que Public Clients, como aplicativos mobile e SPAs, utilizem o Authorization Code flow com o PCKE extension no lugar do Implicit flow.

Por ser um fluxo totalmente desencorajado, não entraremos em detalhes. De qualquer forma, caso tenha interesse em entender mais sobre este fluxo e os passos de interação no processo de autorização, basta [consultar a especificação do OAuth2](https://datatracker.ietf.org/doc/html/rfc6749#section-4.2).

## Enfim, qual fluxo devo utilizar?

Conhecer os fluxos e suas particularidades ajuda bastante na hora de decidir qual fluxo adotar ao construir um processo de autorização e autenticação entre serviços. Para facilitar esta decisão, segue um flowchart que resume tudo que discutimos neste conteúdo:

![OAuth 2 Decision Flowchart](/seguranca-com-spring-security-e-oauth2/imagens/flowchat-oauth2-decision.png)

Como você pode ver, os fatores determinantes na hora de decidir qual fluxo adotar tem tudo a ver com **Client Type (public ou confidential)** e **quem assume o papel de Resource Owner**. 😉

## Links e referências

Aqui são alguns links de artigos, referências oficiais e não oficiais que podem te ajudar no aprendizado e aprofundamento:

- [OAuth 2.0](https://oauth.net/2/)
- [RFC 6749 Section 2.1: OAuth 2.0 Client Types](https://oauth.net/2/client-types/)
- [RFC 6749: Obtaining Authorization](https://datatracker.ietf.org/doc/html/rfc6749#section-4)
- [RFC 6749: Authorization Code Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1)
- [RFC 6749: Resource Owner Password Credentials Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.3)
- [RFC 6749: Client Credentials Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.4)
- [RFC 6749: Implicit Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.2)
- [RFC 7636: PKCE - Proof Key for Code Exchange](https://oauth.net/2/pkce/)
- [OAuth 2.0 Implicit Grant](https://oauth.net/2/grant-types/implicit/)
- [OAuth 2.0  Refresh Token](https://oauth.net/2/grant-types/refresh-token/)
- [OAuth 2.0 Device Authorization Grant](https://oauth.net/2/grant-types/device-code/)
- [OAuth 2.0 and OpenID Connect Overview](https://developer.okta.com/docs/concepts/oauth-openid/)


