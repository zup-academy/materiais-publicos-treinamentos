# Introdução a OAuth 2.0 e OpenID Connect

Nesse conteúdo teremos uma visão de alto nível sobre o que e como funciona o protocolo OAuth 2.0 e OpenID Connect. A idéia aqui é termos uma visão clara de como o protocolo (também chamado de framework) OAuth 2.0 funciona, seus principais fluxos e seus atores (roles).

## Segurança: Autenticação Vs Autorização

Sempre que falamos sobre segurança de uma aplicação web ou sistema distribuído é comum encontrarmos dois conceitos comums porém de extrema importância: **Autenticação** e **Autorização**. Embora exista uma relação estreita entre eles, eles tem semânticas e papeis diferentes numa arquitetura.

Quando falamos de **autenticação** estamos falando sobre **identificar um usuário** no sistema, estamos falando do sistema ter que lidar com suas credenciais (username e password), valida-las e carregar as informações pertinentes deste usuário para uma sessão ativa dentro do sistema, e somente a partir daí o usuário poderia começar a navegar pelo sistema.

Resumindo, queremos responder a seguinte pergunta:

> "Quem é este usuário?"

Por outro lado, quando falamos de **autorização**, estamos falando sobre **determinar o que um usuário pode fazer** no sistema. Nesse momento, entende-se que o sistema já sabe quem é o usuário pois ele está autenticado (logou com suas credenciais), porém o sistema deve verificar se o mesmo possui determinadas permissões (ou pápeis) para executar alguma ação ou acessar dados no sistema.

Aqui, queremos responder a seguinte pergunta:

> "Este usuário autenticado possui a permissão X para executar esta ação?"

Entender esta diferença é importante, não somente pelo fato da comunicação com outros membros do seu time ou para fazer troubleshooing, mas também porque os frameworks e tecnologias são implementados em cima destes conceitos, o que influencia nas suas documentações, exemplos de código, pontos de extensão, integração com outras bibliotecas e muito mais.

## A web está mais complexa e interativa do que imaginamos

Há um pouco mais de 10 anos atrás estavamos a acostumados a construir aplicações web que viviam sozinhas, autocontidas e com pouquissimos pontos de integração, onde todos os softwares e serviços geralmente rodavam na mesma infraestrutura sob o mesmo firewall. Basicamente cada aplicação web tinha sua base de usuários própria e era responsável por implementar e oferecer todas as funcionalidades relacionadas a autenticação e autorização, como página de login, controle de usuários e acessos, gerenciamento de sessões ativas, controle de senhas, configuração de permissões e papéis e muito mais. Os poucos pontos de integração entre sistemas eram "frouxos" sem muita complexidade de segurança pois tudo rodava no mesmo datacenter e era mantida pela mesma equipe de segurança ou infraestrutura, além disso, muitas vezes essas integrações se resolviam com soluções simples de autenticação do protocolo HTTP, como HTTP Basic Auth ou mesmo através da troca de tokens entre as aplicações e serviços.

Com o tempo, os sistemas foram ficando maiores e mais complexos, onde uma aplicação maior (monolito por exemplo) era dividida em dezenas ou centenas de serviços menores que precisavam ser distribuídas na rede mas que ao mesmo tempo se comportavam como se fossem uma única aplicacação. Com essa maneira de desenhar e construir sistemas ficou inviável que a base e gerenciamento de usuários continuasse como uma mera funcionaldiade existente em única aplicação ou mesmo que fosse copiada ou distribuída em todas as aplicações e serviços: já imaginou as dezenas de serviços com sua própria base de usuários?

Nesse contexto, extrair o gerenciamento de usuários e acesso para um serviço centralizado, independente e isolado dos demais faz todo sentido, afinal de contas, seu único papel seria lidar com autenticação e autorização. Não à toa, nesse mundo de sistemas distribuídos um dos primeiros serviços a ser implantado na infraestrutura de uma empresa é este serviço de autenticação e autorização que certamente será utilizado por todos os demais serviços na rede. Este serviço ficou conhecido como **Identity Provider (IdP)**.

Por manter e gerenciar toda a base de usuários, suas credenciais e regras de acesso um dos principais papéis do IdP é permitir que o **usuário se logue (autentique) somente um única vez** e assim consiga acessar qualquer sistema ou serviço que confie neste IdP, desta forma o usuário não precisa ter múltiplos usernames e passwords dentro de uma empresa. Esse conceito de o usuário ter uma credencial única e ter que se logar somente uma única vez na empresa é conhecido como **Single Sign-On (SSO)**. A feature de SSO é comumente ofertada por todos os IdP's do mercado.

Esse modelo de segurança funciona muito bem numa empresa que possui uma infraestrutura estritamente fechada, com seu próprio datacenter onde há controle sobre todos os usuários e serviços, mas que apresenta desafios quando estes mesmos sistemas e serviços precisam ir para as nuvens ou precisam se integrar com aplicações terceiras (third-party services). Imagine que sua empresa rode um sistema de agendamento de férias e precisa se integrar com Google Calendar dos seus usuários, você acha mesmo que seu usuário vai informar as suas credenciais do Google para seu sistema acessar a agenda dele? Quem garante que seu sistema não vai acessar outras informações da conta Google deste usuário? Quem garante que seu sistema não vai também ler os emails, acessar os contatos ou alterar a senha da conta deste usuário?

Simplesmente o usuário repassar suas credenciais para este sistema de agendamento de férias siginifica dar **muito poder de acesso** para um único sistema, além de abrir diversas brechas de segurança. Se pensarmos bem, esse sistema não quer ter acesso a conta deste usuário, mas apenas acesssar sua agenda no Google Calendar para registrar um novo evento de férias na agenda dele. O que esse sistema de fato quer é executar uma determinada ação **em nome do usuário** que é dono da conta Google.

Você pode até pensar que sua conta do Google não tem nada de importante, mas pense que em vez dela você tivesse que repassar suas credenciais da sua conta corrente do seu banco?

Perceba que nesse modelo de integração entre sistemas e serviços de forma segura o usuário precisa **conceder acesso** a determinada ação para um sistema. Aqui, paramos de ter as credenciais do usuário como ponto central na integração e passamos a ter concessões (permissões) concedidas pelo usuário para que um sistema acesse e execute ações noutro sistema **em nome deste usuário**. Passamos a focar em **autorizações** mais do que autenticações, identidade ou mesmo senhas dos usuários.

Para que esse modelo funcione, foi necessário desenhar e criar um framework e protocolo de autorização conhecido como **OAuth 2.0**.

## Entendendo o protocolo OAuth 2.0

OAuth 2.0, ou simplesmente OAuth2, que significa "Open Authorization", é basicamente um padrão sobre HTTP para autorização entre sistemas, serviços, aplicações e dispositivos que permite a um usuário (ou outra aplicação) conceder acesso limitado a seus recursos para aplicações terceiras (third-party applications). Ele se baseia na emissão e troca de tokens de acessso (access tokens) entre as aplicações e serviços. Por ser um padrão ela possui sua especificação aberta na [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749), onde o mesmo é descrito como abaixo:

> OAuth 2.0 is an authorization framework that enables a third-party application to obtain limited access to an HTTP service, either on behalf of a resource owner by orchestrating an approval interaction between the resource owner and the HTTP service, or by allowing the third-party application to obtain access on its own behalf.

Apesar de chamarmos OAuth2 de protocolo, o que não está errado, ele está mais para um framework de autorização por permitir que muitas features sejam opcionais para quem resolver implementa-lo, o que acaba não garantindo uma interoperabilidade 100% entre serviços. 

Aqui é importante entender que **OAuth2 é sobre AUTORIZAÇÃO**, e não autenticação. Tanto é que sua RFC nem entra muito nesses detalhes. Quem cuida de fato da autenticação no OAuth2 é o protocolo **OpenID Connect**, que funciona como uma camada fina sobre o OAuth2 para padronizar aspectos relacionados a autenticação, endpoints, tokens e formatos e também permitir que desenvolvedores(as) consigam autenticar seus usuários em sites ou aplicativos sem a necessidade de manter e gerenciar credenciais. 

> **OpenID Connect, Single Sign-On e Social Logins** <br/>
> OpenID Connect é o protocolo em cima do OAuth2 que nos permite implementar **Single Sign-On (SSO)**. É justamente ele que habilita um usuário se autenticar uma única vez com o IdP (Identity Provider) e consiga acessar qualquer aplicação ou sistema que tenha uma relação de confiança com esse IdP, ou seja, que esteja configurado neste IdP.
>
> Também é através OpenID Connect que implementamos o que chamamos de **Social Logins**, ou seja, usamos nossas contas de redes sociais como Facebook, Instagram, Google etc para criar contas e/ou se autenticar em sites e serviços na internet, como seu e-commerce preferido ou algum site de jogos.
>
> No fim, podemos dizer que OpenID Connect está em todo lugar assim como OAuth2 😉

Separar autenticação de autorização nem sempre é fácil, mas para recaptular, em termos simples:

- authentication: autenticação é sobre identificar um usuário;
- authorization: autorização é sobre determinar o que um usuário pode fazer ou acessar;

**OAuth2 é sobre conceder autorização (permissão) para que aplicações terceiras (third-party applications) consigam executar ações ou acessar dados a um determinado recurso em nome do usuário.**

Sem perceber nós usamos OAuth2 a todo momento hoje em dia, especialmente nos nossos smart-phones. Por exemplo, ao configurar seu iPhone para acessar sua conta Google diretamente para ler e enviar emails via GMail, para carregar e adicionar contatos ou mesmo registrar eventos no seu Google Calendar; ao permitir que seu Android leia e grave arquivos diretamente no seu Dropbox; ou que um site ou aplicativo consiga postar um tweet na sua timeline do Twitter ou mesmo publicar um vídeo editado na sua página de Stories do seu Instagram.

> **OAuth2 em todos os lugares, não só aplicativos e sites** <br/>
> O modelo de autorização do OAuth2 está em praticamente tudo hoje em dia, não apenas aplicativos no seu smartphone ou sites e redes sociais,mas também em dispositivos embarcados, SmartTVs, IoT, consoles de video-game etc.  Por exemplo, eu consigo publicar mensagens, imagens e vídeos no meu Twitter diretamente do meu Nintendo Switch. O console posta esses conteúdos em meu nome.


Com isto em mente, fica mais fácil perceber que OAuth2 permite que aplicações obtenham **acesso limitado** a um serviço ou um recurso.

Falando em acesso limitado, o protocolo tem como uma de suas principais caracteristicas ajudar usuários a **não informarem suas credenciais** diretamente para aplicações terceiras. Isso é uma caracteristica importante pois estas aplicações terceiras terão somente o acesso minimo suficiente para fazer o que elas precisam fazer (e que foram autorizadas) e nada mais, o que naturalmente acaba por diminuir os riscos de comprometer as credenciais do usuário.

No fim, o usuário basicamente autoriza uma aplicação terceira a acessar seus recursos como se fosse o próprio usuário executando a ação. Por se tratar de autorizações especificas para determinadas ações, fica fácil para o usuário limitar o que diferentes aplicações terceiras podem ou não fazer em seu nome.

## OAuth2 Flows, Roles e Access Tokens

Um aspecto muito importante do OAuth2 são seus fluxos (flows) de autorização, mas para entendê-los temos antes que nos acostumarmos com seus atores/pápeis (roles) e terminologia adotada pelo framework. A grosso modo existem **4 atores** num fluxo de autorização OAuth2, são eles:

- **Resource Owner**: é o dono do recurso. Pode ser uma pessoa ou um serviço. Por exemplo, você é o dono das suas "coisas no Facebook (fotos, perfil etc) e é você quem permite ou não acesso a elas;
- **Resource Server**: é onde ficam os recursos protegidos (suas fotos do Instagram, arquivos do Dropbox, eventos do Google Calendar", por exemplo). Esse servidor precisa ser capaz de lidar com tokens de acesso (access tokens) pra fornecer, negar e revogar os acessos aos recursos protegidos;
- **Client**: é basicamente a aplicação que quer acessar os recursos protegidos. A aplicação acessa os recursos EM NOME do Resource Owner (do dono do recurso). Por exemplo, a aplicação que você usa para acessar o Twitter seria um Client no fluxo OAuth2;
- **Authorization Server**: é a entidade (server) que emite os tokens de acesso. É para ela que o Client envia as requisições solicitando os tokens de acesso para consumir algum recurso do Resource Owner. Apesar dele geralmente ser um serviço separado, ele pode ser o mesmo serviço do Resource Server;

Vamos entrar um pouco mais em detalhes para entendermos melhor o papel de cada um destes atores:

**O Resource Owner é o ator capaz de conceder acesso a seus recursos protegidos**. Na maioria das vezes ele será um usuário, mas ele também pode ser outra aplicação ou serviço.

**O Resource Server hospeda os recursos protegidos**. Geralmente é uma API REST, como API do Facebook, Google Drive ou Instagram.

Enquanto o **Authorization Server é a aplicação que emite os tokens** para autorizar os Clients. Existem diversos serviços opensource que podemos rodar localmente ou no nosso próprio datacenter, como Keycloak, e SaaS (Software as a Service) como Okta e Auth0. Geralmente tem-se um Authorization Server centralizado para atender múltiplos Resource Servers.

Por fim, **o Client é a aplicação fazendo as requisições para os recursos protegidos** em nome do Resource Owner. Ela pode ser um website, aplicativo mobile, microsserviço, SmartTV etc.

### Existe uma relação de confiança entre os atores

Para que os fluxos do OAuth2 funcionem, deve haver uma **relação de confiança** entre os atores.

Primeiro, entre o Resource Server e Authorization Server: o Resource Server precisa ser capaz de validar as keys e tokens emitidas pelo Authorization Server antes de permitir acesso aos recursos solicitados pelos Clients.

O Client também deve se registrar no Authorization Server para que seja possível solicitar e receber os tokens, ou seja, o Authorization Server precisa ter conhecimento da existência de cada Client.

### Overview do OAuth2 Flow

Como vimos, OAuth2 tem a ver com os fluxos de interação entre seus atores (resource owner, client, resource server e authorization server), por esse motivo é importante que você se sinta confortável com estes fluxos. 

Tecnicamente, temos **4 tipos de fluxos (grant types)** que podem ser usados ao implementarmos autorização com OAuth2:

1. Authorization Code;
2. Client Credentials;
3. Implicit;
4. Resource Owner Password Credentials;

Estes 4 fluxos são utilizados para obter um access token. Mas antes de discutirmos em detalhes seus principais fluxos de autorização (grant types), vamos ter uma **visão geral e de alto nível** do fluxo deste protocolo:

```
+--------+                               +---------------+
|        |--(A)- Authorization Request ->|   Resource    |
|        |                               |     Owner     |
|        |<-(B)-- Authorization Grant ---|               |
|        |                               +---------------+
|        |
|        |                               +---------------+
|        |--(C)-- Authorization Grant -->| Authorization |
| Client |                               |     Server    |
|        |<-(D)----- Access Token -------|               |
|        |                               +---------------+
|        |
|        |                               +---------------+
|        |--(E)----- Access Token ------>|    Resource   |
|        |                               |     Server    |
|        |<-(F)--- Protected Resource ---|               |
+--------+                               +---------------+
```

A figura acima descreve em alto nível as interações entre os 4 atores e inclui os seguintes passos:

- (A) O client pede autorização do resource owner. Este pedido de autorização pode ser feito diretamente ao resource owner (como mostrado na figura), ou preferencialmente ao authorization server, na qual funcionaria como um intermediário;

- (B) O client recebe uma concessão de autorização (authorization grant), na qual é uma credencial representando a autorização do resource owner. Aqui esta concessão de autorização pode ser expressada pelos 4 grant types definidos na especificação;

- (C) Agora, o client solicita um token de acesso (access token) autenticando-se com o authorization server e apresentando sua concessão de autorização;

- (D) O authorization server autentica o client e valida sua concessão de autorização, e se ela for válida, ele emite um token de acesso para o client;

- (E) Com o access token em mãos, o client solicita acesso ao recurso protegido hospeado pelo resource server apresentando o token que foi obtido do authorization server;

- (F) Por fim, o resource server valida o access token, e se válido, libera acesso ao recurso protegido;

Perceba que OAuth2 tenta a todo custo não induzir o resource owner (end-user) a informar suas credenciais (username e password) ao client (aplicação terceira). Por isso a preferência de ter um authorization server como intermediário e a importância de toda a discussão sobre relação de confiança entre os atores.

### Access Tokens

Outro conceito importante na terminologia do OAuth2 e que discutimos anteriormente é o **Access Token**, que é utilizado como concessão de autorização para um Client em vez das credenciais (username e password) do Resource Owner (end-user). 

**Estes access tokens são emitidos pelo Authorization Server para os Clients e são utilizados para acessar recursos protegidos hospedados pelo Resource Server**. Eles podem ter diferentes formatos e estruturas e podem ser utilizados de diferentes formas de acordo com os requisitos do Resource Server.

Sem dúvida o formato mais popular e utilizado hoje em dia com OAuth2 para access token é o **JWT (JSON Web Token)**, que nada mais do que um JSON com informações e detalhes sobre o token, resource owner, authorization server entre outras, tanto é que você pode copiar este token (em formato derivado de Base64) e decodifica-lo no site [JWT.io](https://jwt.io/). Não à toa, por questões de segurança, o JWT pode ser (e geralmente é) assinado e até encriptado para evitar que o mesmo seja adulterado ou lido no meio do caminho.

Mas não se engane, o protocolo OAuth2 permite que outros formatos sejam adotados, como os formatos opacos (opaque tokens). Não há restrições na especificação sobre quais formatos e estruturas devem ser adotados.

## Links e referências

Alguns são alguns links de artigos, referências oficiais e não oficiais que podem te ajudar no aprendizado e aprofundamento:

- [RFC6749: The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749)
- [OpenID Connect FAQ and Q&As](https://openid.net/connect/faq/)
- [What’s the Difference Between OAuth, OpenID Connect, and SAML?](https://www.okta.com/identity-101/whats-the-difference-between-oauth-openid-connect-and-saml/)
- [[Conceito] - Básico sobre OAuth 2.0](https://dev.to/zanfranceschi/conceito-basico-sobre-oauth-20-3bfb)
- [Wikipedia: OAuth2](https://en.wikipedia.org/wiki/OAuth)
- [Identity Providers (IdPs): What They Are and Why You Need One](https://www.okta.com/identity-101/why-your-company-needs-an-identity-provider/)