# Entendendo _View Controllers_

>Esse material teórico foi traduzido e adaptado da fonte original disponível em https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/index.html

## O papel dos _View Controllers_

Os _View Controllers_ (controladores de _views_) são a base da estrutura interna do seu aplicativo. Cada aplicativo tem pelo menos um _View Controller_, e a maioria dos aplicativos tem vários. Cada _View Controller_ gerencia uma parte da UI do seu aplicativo, bem como as interações entre essa interface e os dados subjacentes. Os _View Controllers_ também facilitam as transições entre as diferentes partes da interface do usuário.

Como eles desempenham um papel tão importante em seu aplicativo, os _View Controllers_ estão no centro de quase tudo o que você faz. A classe `UIViewController`, provida pelo UIKit Framework, define os métodos e propriedades para gerenciar suas _views_, manipular eventos, fazer a transição de um _View Controller_ para outro e interagir com outras partes de seu aplicativo. Em geral, para implementar o comportamento do seu aplicativo, você cria classes que sobrescrevem `UIViewController` (ou uma de suas subclasses) e adiciona o código necessário ao seu contexto.

Existem dois tipos de _View Controllers_:

- Os _Content View Controllers_ (_View Controllers_ com conteúdo) gerenciam uma parte bem definida da UI do seu aplicativo e são o principal tipo de _View Controller_ que você cria.

- Os _Container View Controllers_ (controladores contêineres) mantém referências e coletam informações de outros _View Controllers_ (conhecidos como _Child View Controllers_) e as apresentam de uma forma que facilita a navegação ou apresenta o conteúdo desses _controllers_ de maneira customizada.

A maioria dos aplicativos é uma mistura dos dois tipos de _View Controllers_.

### Gerenciamento de _Views_

A função mais importante de um _View Controller_ é gerenciar uma hierarquia de _views_. Cada _View Controller_ tem uma única _view_ raiz (_root view_) que inclui todo o conteúdo do _View Controller_. A essa _view_ raiz, você adiciona as _views_ de que precisa para exibir seu conteúdo. A figura a seguir ilustra o relacionamento interno entre o _View Controller_ e suas _views_. O _View Controller_ sempre tem uma referência à sua _root view_, assim como cada _root view_ tem referências fortes a suas _subviews_.

![Imagem ilustrando relação entre ViewController e suas views](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-vc-view-controller-e-views.jpg?raw=true)

>Nota: É uma prática comum usar _[outlets](<to do: ligar com material teorico de outlets>)_ para acessar outras _views_ na hierarquia de _views_ do _View Controller_. Como um _View Controller_ gerencia o conteúdo de todas as suas _views_, os _outlets_ permitem que você guarde referências diretas às _views_ de que precisa. Os próprios _outlets_ são conectados aos objetos das _views_ automaticamente quando as _views_ são carregadas do storyboard. No próximo material teórico estudamos a utilidade de um _outlet_.

### Gerenciamento dos dados

Um _View Controller_ atua como um intermediário entre as _views_ que ele gerencia e os dados de seu aplicativo que estão relacionados a elas. Os métodos e propriedades da classe `UIViewController` permitem gerenciar a apresentação visual de seu aplicativo. Ao criar uma subclasse de `UIViewController`, você adiciona quaisquer variáveis de que precisa para gerenciar seus dados nessa subclasse. Adicionar variáveis personalizadas cria um relacionamento como o da figura abaixo, onde o _View Controller_ tem referências aos seus dados e às _views_ usadas para apresentar esses dados. Atualizar os dados a medida que o aplicativo executa suas funções e gerenciar a relação deles com suas _views_ é sua responsabilidade.

![Imagem ilustrando ViewController como intermediário entre dados e views que gerencia](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-vc-views-e-dados.jpg?raw=true)

*Um View Controller media a relação entre objetos de dados e views.*

Você deve sempre manter uma separação clara de responsabilidades dentro de seus _View Controllers_ e objetos de dados. A maior parte da lógica para garantir a integridade de suas estruturas de dados pertence aos próprios objetos de dados. O _View Controller_ pode validar a entrada proveniente de _views_ e, em seguida, empacotar essa entrada no formato que seus objetos de dados exigem, mas você deve minimizar a função do _View Controller_ no gerenciamento dos dados reais.

## Relações de um _View Controller_ na estrutura da aplicação

As relações entre os _View Controllers_ do seu aplicativo definem os comportamentos necessários para a aplicação. O UIKit espera que você use _View Controllers_ de maneiras predefinidas. Manter os relacionamentos dos controladores respeitando as decisões do _framework_ garante que comportamentos automáticos funcionem adequadamente. Qualquer quebra em relação aos relacionamentos entre entidades fundamentais do design podem fazer com que partes do seu aplicativo parem de funcionar conforme o esperado.

### UIWindow, Root View Controller e demais controladores

Para que entendamos como estão organizados hierarquicamente e como se relacionam os controladores, precisamos primeiro estabelecer uma relação precedente fundamental: a de um controlador com a janela do aplicativo (definida pela classe `UIWindow`), que adiante será tratada apenas por _window_.

Uma _window_ fundamentalmente representa o pano de fundo para a interface do usuário do seu aplicativo. O objeto de `UIWindow` trabalha com _View Controllers_ para lidar com eventos e realizar muitas outras tarefas que são fundamentais para a operação do aplicativo. O UIKit lida automaticamente com a maioria das interações relacionadas à _window_, trabalhando com outros objetos conforme necessário para implementar muitos comportamentos do aplicativo.

Sua aplicação usa a _window_, no geral, apenas quando precisa fornecer uma janela principal para a exibição do conteúdo do seu aplicativo. Projetos iOS que usam _Storyboards_ para desenhar as _views_ do aplicativo, exigem a presença de uma propriedade de _Window_ no objeto SceneDelegate, que normalmente o template de projeto Xcode fornece automaticamente e que tem a função de gerir a forma como seu aplicativo é apresentado na tela.

Como a _window_ não tem conteúdo próprio visível, é necessário que a utilizemos apenas para hospedar uma ou mais _views_, que são gerenciadas por um _View Controller_, o _Root View Controller_ da janela. Você configura o _Root View Controller_ em seus arquivos Storyboard, adicionando as _views_ apropriadas para sua interface.

O _Root View Controller_ é a âncora da hierarquia de _View Controllers_. Cada _window_ tem exatamente um _Root View Controller_ cujo conteúdo a preenche e define o que é visto inicialmente pelo usuário. A figura a seguir mostra a relação entre o _Root View Controller_ e a _window_.

![Imagem ilustrando relação do ViewController raiz com as entidades que gerenciam as janelas da aplicação](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-vc-root-view-controller.jpg?raw=true)

Todo o trabalho de _setup_ das relações vistas acima é executado automaticamente pelo UIKit através dos códigos que guiam o carregamento da aplicação. Somada a outras importantes tarefas, é exatamente o que acontece entre o usuário clicar no ícone da sua aplicação e visualizar o conteúdo inicial que você desenhou em seu arquivo `Main.storyboard` através do Interface Builder.

Além da relação fundamental tratada acima, é possível compor uma hierarquia de _View Controllers_ a partir do _root controller_. Apresentando outro _View Controller_ substituimos o conteúdo do _controller_ atual, geralmente ocultando o conteúdo do anterior. A classe `UIViewController` forcene os meios necessários para que esse tipo de transição seja feita de maneira adequada. Algumas seções adiante estudaremos especificamente técnicas para criar essas relações entre _View Controllers_ compondo assim os fluxos de navegação dos nossos aplicativos.

### Dicas: Design de Código

Os _View Controllers_ são uma ferramenta essencial para aplicativos em execução no iOS, e a infraestrutura do _View Controller_ do UIKit facilita a criação de interfaces sofisticadas sem escrever muitos códigos. Ao implementar seus próprios _View Controllers_, use as dicas e diretrizes a seguir para garantir que você não esteja fazendo coisas que possam interferir no comportamento natural esperado pelo sistema.

- **Use _View Controllers_ fornecidos pelo sistema sempre que possível**

    Muitos frameworks iOS definem _View Controllers_ que você pode usar em seus aplicativos. Usar esses _View Controllers_ fornecidos pelo sistema economiza tempo e garante uma experiência consistente para o usuário.

    A maioria dos _View Controllers_ do sistema são projetados para tarefas específicas. Antes de criar seu próprio _View Controller_ customizado, pesquise as estruturas existentes para ver se já existe um _View Controller_ para a tarefa que você deseja executar.

- **Faça de cada _View Controller_ uma ilha**

    Os _View Controllers_ devem ser sempre objetos independentes *(self-contained)*. Nenhum _View Controller_ deve ter conhecimento sobre o funcionamento interno ou hierarquia de visualização de outro _View Controller_. Nos casos em que dois _View Controllers_ precisam se comunicar ou passar dados de um lado para outro, eles sempre devem fazer isso usando interfaces públicas definidas explicitamente.

    Algumas seções a frente neste treinamento, estudaremos padrões de projeto que podem facilitar a integração em casos como o citado acima.

- **Use a _root view_ apenas como um contêiner para outras _views_**

    Use a _root view_ do seu _View Controller_ apenas como um contêiner para o resto do seu conteúdo. Usar a _root view_ como um contêiner fornece a todas as suas _views_ uma _view_ mãe comum, o que torna muitas operações de layout mais simples. Muitas _constraints_ de Auto Layout requerem uma _view_ mãe comum para orientar o layout das _views_ de maneira adequada.

- **Saiba onde seus dados devem estar**

    Na estrutura padrão de uma aplicação iOS, a função de um _View Controller_ é facilitar a movimentação de dados entre seus _data objects_ e seus objetos de _view_. Um _View Controller_ pode armazenar alguns dados em variáveis temporárias e realizar algumas validações, mas sua principal responsabilidade é garantir que suas _views_ contenham informações precisas. Seus objetos de dados são responsáveis por gerenciar os dados reais e por garantir a integridade geral desses dados.
