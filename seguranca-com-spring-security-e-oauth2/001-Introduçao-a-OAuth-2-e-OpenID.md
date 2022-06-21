# Introdução a OAuth 2.0 e OpenID Connect

Nesse conteúdo teremos uma visão de alto nível sobre o que e como funciona o protocolo OAuth 2.0 e OpenID Connect. A idéia aqui é termos uma visão clara de como o protocolo (também chamado de framework) OAuth 2.0 funciona, seus principais fluxos e seus atores (roles).

## Segurança: Autenticação Vs Autorização

Sempre que falamos sobre segurança de uma aplicação web ou sistema distribuído, é comum encontrarmos dois conceitos comums mas muito importantes: **Autenticação** e **Autorização**. Embora exista uma relação estreita entre eles, eles tem semântica e papeis diferentes numa arquitetura.

Quando falamos de **autenticação** estamos falando sobre **identificar um usuário** no sistema, estamos falando do sistema ter que lidar com suas credenciais (username e password), valida-las e carregar as informações pertinentes deste usuário para uma sessão ativa dentro do sistema, e somente a partir daí o usuário poderia começar a navegar pelo sistema.

Resumindo, queremos responder a seguinte pergunta:

> "Quem é este usuário?"

Por outro lado, quando falamos de **autorização**, estamos falando sobre **determinar o que o usuário pode fazer** no sistema. Nesse momento, entende-se que o sistema já sabe quem é o usuário pois ele está autenticado, porém o sistema deve verificar se o mesmo possui determinadas permissões (ou papeis) para executar alguma ação ou se pode acessar dados no sistema.

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

OAuth 2.0, ou simplesmente OAuth2, é basicamente um padrão para autorização entre sistemas, serviços, aplicações e dispositivos. Aqui é importante entender que **OAuth2 é sobre AUTORIZAÇÃO**, e não autenticação.

Para recaptular, em termos simples:

- autenticação (authentication) é sobre identificar um usuário;
- autorização (authorization) é sobre determinar o que um usuário pode fazer ou acessar;

**OAuth2 é sobre conceder autorização (permissão) para que aplicações terceiras (third-party applications) consigam executar ações ou acessar dados a um determinado recurso em nome do usuário.**

Sem perceber nós usamos OAuth2 a todo momento hoje em dia, especialmente nos nossos smart-phones. Por exemplo, ao configurar seu iPhone para acessar sua conta Google diretamente para ler e enviar emails via GMail, para carregar e adicionar contatos ou mesmo registrar eventos no seu Google Calendar; ao permitir que seu Android leia e grave arquivos diretamente no seu Dropbox; ou que um site ou aplicativo consiga postar um tweet na sua timeline do Twitter ou mesmo publicar um vídeo editado na sua página de Stories do seu Instagram.

> **OAuth2 em todos os lugares, não só aplicativos e sites** <br/>
> O modelo de autorização do OAuth2 está em praticamente tudo hoje em dia, não apenas aplicativos no seu smartphone ou sites e redes sociais,mas também em dispositivos embarcados, SmartTVs, IoT, consoles de video-game etc.  Por exemplo, eu consigo publicar mensagens, imagens e vídeos no meu Twitter diretamente do meu Nintendo Switch. O console posta esses conteúdos em meu nome.


Com isto em mente, fica mais fácil perceber que OAuth2 permite que aplicações obtenham **acesso limitado** a um serviço ou um recurso.

Falando em acesso limitado, o protocolo tem como uma de suas principais caracteristicas ajudar usuários a **não informarem suas credenciais** diretamente para aplicações terceiras. Isso é uma caracteristica importante pois estas aplicações terceiras terão somente o acesso minimo suficiente para fazer o que elas precisam fazer (e que foram autorizadas) e nada mais, o que naturalmente acaba por diminuir os riscos de comprometer as credenciais do usuário.

No fim, o usuário basicamente autoriza uma aplicação terceira a acessar seus recursos como se fosse o próprio usuário executando a ação. Por se tratar de autorizações especificas para determinadas ações, fica fácil para o usuário limitar o que diferentes aplicações terceiras podem ou não fazer em seu nome.



