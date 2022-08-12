# Conhecendo o `UITabBarController`

> Nota: Esse material teórico foi traduzido e adaptado da fonte original em: https://developer.apple.com/documentation/uikit/uitabbarcontroller

Até este ponto do treino trabalhamos com apps com diferentes experiências quanto à divisão do fluxo de trabalho dos _view controllers_. De apps _single screen_ na maior parte do tempo, até apps que se apoiam na infraestrutura de _navigation controllers_ para organizar fluxos de navegação hierárquicos sobre a informação. Nesta seção vamos adicionar mais uma possibilidade para nossos aplicativos.

Existem situações onde desejamos resolver problemas distintos no contexto de aplicações móveis, onde é possível identificar seções específicas e independentes entre si na experiência do usuário. Por exemplo, podemos citar uma aplicação de editora dividindo conceitualmente em seções o consumo de informações sobre livros e sobre autores, ou uma aplicação de fitness dividindo seções para o consumo de atividades físicas e planos alimentares, ou mesmo aplicações que você usar diariamente como o Relógio, que divide seções para o consumo de informações sobre horas em diferentes regiões do globo, programação de alarmes, cronômetro e timers.

Os dois primeiros exemplos mencionados acima serão utilizados como objeto de prática durante esta seção. E você vai aprender como podemos tirar proveito do `UITabBarController` para propor soluções simples para experiências como estas.

## `UITabBarController`

>`@MainActor class UITabBarController : UIViewController`

Um contêiner _view controller_ que gerencia uma interface de seleção múltipla entre abas _(tabs)_, em que a seleção determina qual _view controller_ filho exibir.

## Overview

A interface da Tab Bar exibe abas _(tabs)_ na parte inferior da janela para selecionar entre os diferentes modos e para exibir as _views_ desse modo. Esta classe é geralmente usada como está, mas também pode ser estendida.

Cada _tab_ de uma interface do _tab bar controller_ está associada a um _view controller_ personalizado. Quando o usuário seleciona uma _tab_ específica, o _tab bar controller_ exibe a _root view_ do _view controller_ correspondente, substituindo quaisquer _views_ anteriores. Como a seleção de uma _tab_ substitui o conteúdo da interface, o tipo de interface gerenciado em cada _tab_ não precisa ser semelhante de forma alguma. Na verdade, as interfaces de _tab bar_ são comumente utilizadas para apresentar diferentes tipos de informação ou para apresentar a mesma informação usando um estilo de interface completamente diferente. A figura abaixo mostra a interface da _tab bar_ apresentada pelo aplicativo Relógio, cada aba apresenta um tipo de informação baseada em tempo.

<p align="center">
<img alt="Imagem mostrando quatro telas do aplicativo de Relógio do iOS, com divisão entre tabs, onde a seleção de uma tab exibe seu conteúdo relacionado na janela do aplicativo. Existem quatro diferentes tabs no aplicatico: horário em regiões do globo, alarmes, cronômetro e timers" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/tab-bar-controllers-teoria-imagem-exemplo-tabs-app-relogio.png?raw=true" width="80%"/>
</p>

Você nunca deve acessar a _view_ da _tab bar_ de um _tab bar controller_ diretamente. Para configurar as _tabs_ de um _tab bar controller_, você atribui os _view controllers_ que fornecem a _root view_ para cada _tab_ à propriedade `viewControllers`. A ordem na qual você especifica os _view controllers_ determina a ordem em que eles aparecem na _tab bar_. Ao definir essa propriedade programaticamente, você também deve atribuir um valor à propriedade `selectedViewController` para indicar qual _view controller_ está selecionado inicialmente. (Você também pode selecionar _view controllers_ por índice de array usando a propriedade `selectedIndex`.) Quando você incorpora a _view_ do _tab bar controller_ (obtida usando a propriedade `view` herdada) na janela do seu aplicativo, o _tab bar controller_ seleciona automaticamente esse _view controller_ e exibe seu conteúdo, redimensionando-os conforme necessário para caber na interface da _tab bar_.

<p align="center">
<img alt="Imagem mostrando o modelo de classes no design da solução do tab bar controller, relacionando o objeto de UITabBarController com seus view controllers filho através da propriedade viewControllers e também com seus objetos para a content view e view da tab bar" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/tab-bar-controllers-teoria-imagem-modelo-tab-bar-controllers.jpg?raw=true" width="75%"/>
</p>


Os itens da _tab bar_, como título e imagens, são configurados por meio de seu _view controller_ correspondente. Para associar um item da _tab bar_ a um _view controller_ programaticamente, você pode criar uma nova instância da classe `UITabBarItem`, configurá-la adequadamente para o _view controller_ e atribuí-la à propriedade `tabBarItem` do _view controller_. Se você não fornecer um _tab bar item_ personalizado para seu _view controller_, o _view controller_ criará um item padrão que não contém imagem e o texto da propriedade `title` será o mesmo da propriedade `title` do _view controller_.

<p align="center">
<img alt="Imagem mostrando o modelo de classes no design da solução dos itens de tab bar, relacionando o objeto de UITabBar com o UITabBarItem de cada respectivo controlador apresentado no contexto de tabs" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/tab-bar-controllers-teoria-imagem-modelo-tab-bar-x-tab-bar-items.jpg?raw=true" width="75%"/>
</p>

À medida que o usuário interage com uma interface de _tab bar_, o objeto _tab bar controller_ envia notificações sobre as interações para seu _delegate_. O _delegate_ pode ser qualquer objeto que você especificar mas deve estar em conformidade com o protocolo `UITabBarControllerDelegate`. É possível utilizar o _delegate_ para conter um ajuste mais fino para suas funcionalidades, como por exemplo, para impedir que itens específicos da _tab bar_ sejam selecionados em determinado estado da app ou para executar tarefas adicionais quando _tabs_ forem selecionadas. Você também pode usar o _delegate_ para monitorar as alterações na _tab bar_ feitas pelos _navigation controllers_, por exemplo.

## As _views_ de um _Tab Bar Controller_

Como a classe `UITabBarController` herda da classe `UIViewController`, os _tab bar controllers_ têm sua própria _view_ que pode ser acessada por meio da propriedade `view`. A _view_ de um _tab bar controller_ é apenas um contêiner para uma _view_ de _tab bar_ (`UITabBar`) e a _view_ que contém seu conteúdo personalizado (`UIView`). A _view_ da _tab bar_ fornece os controles de seleção para o usuário e consiste em um ou mais itens de _tab_. A figura abaixo mostra como essas _views_ são montadas para apresentar a interface geral da _tab bar_. Embora os itens nas _views_ da _tab bar_ possam ser alterados, as _views_ que os gerenciam não. Somente a _view_ de conteúdo personalizado é alterada para refletir o _view controller_ da _tab_ selecionada no momento.

<p align="center">
<img alt="Imagem mostrando a composição de view de um layout baseado em tabs com seus respectivos controladores filho" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/navigation-controllers-teoria-imagem-hierarquia-de-views-do-navigation.jpg?raw=true" width="80%"/>
</p>

Você pode usar _navigation controllers_ ou _view controllers_ personalizados como o _root view controller_ para uma _tab_. Se o _root view controller_ for um _navigation controller_, o _tab bar controller_ fará ajustes adicionais no tamanho do conteúdo de navegação exibido para que ele não se sobreponha à _tab bar_.

## O _Navigation Controller **More**_

A _tab bar_ tem espaço limitado para exibir seus itens. Se você adicionar seis ou mais _view controllers_ personalizados a um _tab bar controller_, o _tab bar controller_ exibirá apenas os primeiros quatro itens mais o item padrão **Mais _(More)_** na _tab bar_. Tocar no item _More_ abre uma interface padrão para selecionar os itens restantes.

A interface para o item padrão _More_ inclui um botão Editar que permite ao usuário reconfigurar a _tab bar_. Por padrão, o usuário tem permissão para reorganizar todos os itens na _tab bar_. Se você não quiser que o usuário modifique alguns itens, você pode remover os _view controllers_ apropriados do array na propriedade `customizeViewControllers`.

<p align="center">
<img alt="Animação com exemplo do More Navigation Controller integrado em um app com muitas tabs e sua possibilidade de edição" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/tab-bar-controllers-teoria-gif-exemplo-more-view-controller.gif?raw=true" width="60%"/>
</p>
