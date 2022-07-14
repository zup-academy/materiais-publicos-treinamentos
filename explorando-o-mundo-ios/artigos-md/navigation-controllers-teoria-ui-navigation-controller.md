# Apresentando o `UINavigationController`

> Nota: Esse material teórico foi traduzido e adaptado da fonte original em: https://developer.apple.com/documentation/uikit/uinavigationcontroller

Alguns aplicativos no mercado conseguem cumprir suas funções baseados em uma única tela (_single screen app_), mas a maior parte deles, para contar com uma boa experiência de utilização, precisam dividir partes do trabalho em várias telas compondo um fluxo de navegação. Esse material teórico visa introduzir o `UINavigationController` do UIKit _framework_, que é o controlador de views que provê a infraestrutura mínima para obter esse tipo de comportamento nos apps iOS.

## `UINavigationController`

>`@MainActor class UINavigationController : UIViewController`

Um Contêiner View Controller que define um esquema baseado em pilha para navegar pelo conteúdo hierárquico de telas.

## Overview

Um _navigation controller_ é um _contêiner view controller_ que gerencia um ou mais _view controllers_ filho em uma interface de navegação. Nesse tipo de interface, apenas um _view controller_ filho é visível por vez. A seleção de um item no _view controller_ aciona a apresentação de um novo _view controller_ na tela usando uma animação, ocultando assim o _view controller_ anterior. Tocar no botão _Voltar_ na barra de navegação na parte superior da tela remove o _view controller_ superior, revelando assim o _view controller_ abaixo.

Use uma interface de navegação que se assemelhe à organização dos dados hierárquicos gerenciados pelo seu aplicativo. Isso favorece o entendimento do fluxo e a usabilidade do seu aplicativo provendo um meio natural de consumo da informação. Em cada nível da hierarquia, você fornece uma tela apropriada (gerenciada por um _view controller_ personalizado) para exibir o conteúdo nesse nível. A figura abaixo mostra um exemplo da interface de navegação apresentada pelo aplicativo Configurações no iOS Simulator. A primeira tela apresenta ao usuário a lista de aplicativos que contêm preferências. A seleção de um aplicativo revela configurações individuais e grupos de configurações para esse aplicativo. Selecionar um grupo produz mais configurações e assim por diante. Para todas as _views_, exceto a raiz, o _navigation controller_ fornece um botão _Voltar_ para permitir que o usuário volte a subir na hierarquia.

<p align="center">
<img alt="Imagem mostrando três telas do aplicativo de Configurações do iOS, com navegação hierárquica, onde a seleção de um item em uma tela apresenta a próxima na hierarquia" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/navigation-controllers-teoria-imagem-exemplo-interface-com-navegacao.jpg?raw=true" width="80%"/>
</p>

Um objeto de _navigation controller_ gerencia seus _view controllers_ filho usando um _array_, conhecido como pilha de navegação _(navigation stack)_. O primeiro _view controller_ no array é o _root view controller_ e representa a parte inferior da pilha (base da pilha). O último _view controller_ no array é o item mais alto da pilha e representa o _view controller_ que está sendo exibido no momento _(top view controller)_. Você adiciona e remove _view controllers_ da pilha usando _segues_ ou usando os métodos da classe `UINavigationController`. O usuário também pode remover o _view controller_ superior usando o botão _Voltar_ na barra de navegação ou usando um gesto de deslizar na borda esquerda.

O _navigation controller_ gerencia a barra de navegação na parte superior da interface e uma barra de ferramentas opcional na parte inferior da interface. A barra de navegação está sempre presente e é gerenciada pelo próprio _navigation controller_, que atualiza a barra de navegação apropriadamente usando o conteúdo fornecido por seus _view controllers_ filho. Quando a propriedade `isToolbarHidden` é `false`, o _navigation controller_ atualiza a barra de ferramentas de forma semelhante com o conteúdo fornecido pelo _view controller_ superior (exibido no momento).

<p align="center">
<img alt="Imagem mostrando um diagrama exemplificando a gestão das views para tab bar e view com o conteúdo do controlador atualmente visível por parte do navigation controller" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/navigation-controllers-teoria-imagem-estrutura-views-navigation-controller.jpeg?raw=true" width="70%"/>
</p>

A figura abaixo mostra os relacionamentos entre o _navigation controller_ e os objetos que ele gerencia. Use as propriedades especificadas do _navigation controller_ para acessar esses objetos.

<p align="center">
<img alt="Imagem mostrando um diagrama contendo o navigation controller e seus objetos de view controllers, navigation bar e toolbar" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/navigation-controllers-teoria-imagem-navigation-controller-e-seus-objetos.jpeg?raw=true" width="80%"/>
</p>

## Views do Navigation Controller

Um _navigation controller_ é um _contêiner view controller_, ou seja, ele incorpora o conteúdo de outros _view controllers_ dentro de si mesmo. Você acessa a _view_ de um _navigation controller_ a partir de sua propriedade `view`. Essa _view_ compreende a barra de navegação, uma barra de ferramentas opcional e a _view_ de conteúdo, correspondente à _view_ do _view controller_ superior. A figura abaixo mostra como essas _views_ são montadas para apresentar a interface de navegação geral. (Nesta figura, a interface de navegação é ainda incorporada dentro de uma interface de tab bar como veremos adiante neste treino.) Embora o conteúdo da barra de navegação e as _views_ da barra de ferramentas mudem, as próprias _views_ não mudam. A única _view_ que realmente muda é a _view_ de conteúdo, que apresenta a _view_ personalizada fornecida pelo _view controller_ mais alto na pilha de navegação no momento.

<p align="center">
<img alt="Imagem mostrando um modelo com a composição de views desde a view do view controller apresentado no momento, até as views de base do navigation controller, um possível controlador de tabs e a window do UIKit" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/navigation-controllers-teoria-imagem-hierarquia-de-views-do-navigation.jpg?raw=true" width="80%"/>
</p>

O _navigation controller_ gerencia a criação, configuração e exibição da barra de navegação e da barra de ferramentas de navegação opcional. É permitido personalizar as propriedades relacionadas à aparência da barra de navegação, mas não é recomendável que você altere seus frames, bounds ou alfa diretamente. Se você criar subclasses de `UINavigationBar`, deverá inicializar seu _navigation controller_ usando o método `init(navigationBarClass:toolbarClass:)`. Para ocultar ou mostrar a barra de navegação, use a propriedade `isNavigationBarHidden` ou o método `setNavigationBarHidden(_:animated:)`.

Um _navigation controller_ cria o conteúdo da barra de navegação dinamicamente usando os objetos de item de navegação (instâncias da classe `UINavigationItem`) associados aos _view controllers_ na pilha de navegação. Para personalizar a aparência geral de uma barra de navegação, use as APIs `UIAppearance`. Para alterar o conteúdo da barra de navegação, você deve, portanto, configurar os itens de navegação (`navigationItem`) de seus _view controllers_ personalizados.

<p align="center">
<img alt="Imagem mostrando um modelo de classes explicando a relação do navigation controller para com sua ui navigation bar e por sua vez do view controller apresentado para com seu navigation item, que serve de repositório para a informação que deve ser apresentada na barra de navegação quando da sua apresentação" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/navigation-controllers-teoria-imagem-modelo-classes-navigation-bar.jpeg?raw=true" width="80%"/>
</p>

## Atualizando a barra de navegação

Cada vez que o _view controller_ no topo da _stack_ é alterado, o _navigation controller_ atualiza a barra de navegação de acordo. Especificamente, o _navigation controller_ atualiza os itens do botão da barra exibidos em cada uma das três posições da barra de navegação: esquerda, central e direita. Os itens de botão da barra de navegação são instâncias da classe `UIBarButtonItem`. Você pode criar itens com conteúdo personalizado ou criar itens de sistema padrão, dependendo de suas necessidades.

A tonalidade da barra de navegação é controlada pelas propriedades da própria barra de navegação. Use a propriedade `tintColor` para alterar a cor dos itens na barra e use a propriedade `barTintColor` para alterar a cor da própria barra. As barras de navegação não herdam sua cor do _view controller_ exibido no momento.

### O item à esquerda na barra de navegação

Para todos, exceto o _root view controller_ na pilha de navegação, o item no canto esquerdo da barra de navegação fornece navegação de volta ao _view controller_ anterior. O conteúdo deste botão mais à esquerda é determinado da seguinte forma:

* Se o novo _view controller_ no topo da _stack_ tiver um item de botão personalizado para a esquerda da barra _(left bar button item)_, esse item será exibido. Para especificar um item de botão personalizado, defina a propriedade `leftBarButtonItem` do `navigationItem` do _view controller_.

* Se o _view controller_ no topo da _stack_ não tiver um item de botão personalizado para a esquerda da barra _(left bar button item)_, mas o item de navegação do _view controller_ anterior tiver um objeto em sua propriedade `backBarButtonItem`, a barra de navegação exibirá esse item.

* Se um item de botão personalizado não for especificado por nenhum dos _view controllers_, um botão _Voltar_ padrão será usado e seu título será definido como o valor da propriedade `title` do _view controller_ anterior, ou seja, o _view controller_ um nível abaixo na _stack_. (Se houver apenas um _view controller_ na pilha de navegação, nenhum botão _Voltar_ será exibido.)

>**Nota:** Nos casos em que o título de um botão _Voltar_ é muito longo para caber no espaço disponível, a barra de navegação pode substituir o título real do botão pela string `“Voltar”`. A barra de navegação faz isso apenas se o botão _Voltar_ for fornecido pelo _view controller_ anterior. Se o novo _view controller_ no topo da _stack_ tiver um item de botão personalizado para a barra à esquerda — um objeto na propriedade `leftBarButtonItem` ou `leftBarButtonItems` de seu item de navegação — a barra de navegação não altera o título do botão.

### O item central

O _navigation controller_ atualiza o centro da barra de navegação da seguinte forma:

* Se o novo _view controller_ no topo da _stack_ tiver uma _view_ de título personalizada, a barra de navegação exibirá essa _view_ no lugar da _view_ de título padrão. Para especificar uma _view_ de título personalizada, defina a propriedade `titleView` do item de navegação do _view controller_ (`navigationItem`).

* Se nenhuma _view_ de título personalizada for definida, a barra de navegação exibirá um _label_ contendo o título padrão do _view controller_. A string para esse _label_ geralmente é obtida da propriedade `title` do próprio _view controller_. Se você deseja exibir um título diferente daquele associado ao _view controller_, defina a propriedade `title` do item de navegação do _view controller_ (`navigationItem`).

### O item à direita na barra de navegação

O _view controller_ atualiza o lado direito da barra de navegação da seguinte forma:

* Se o novo _view controller_ no topo da _stack_ tiver um item personalizado de botão para a direita da barra, esse item será exibido. Para especificar um item personalizado do botão da barra à direita, defina a propriedade `rightBarButtonItem` do item de navegação do _view controller_.

* Se nenhum item de botão personalizado para a barra à direita for especificado, a barra de navegação não exibirá nada no lado direito da barra.

## Adaptando-se a diferentes orientações

A interface de navegação permanece a mesma em orientações horizontais. Ao alternar entre as orientações, apenas o tamanho da _view_ do _navigation controller_ é alterado. O _navigation controller_ não altera sua hierarquia de _view_ ou o layout de suas _views_.

Ao configurar _segues_ entre _view controllers_ em uma pilha de navegação, os _segues_ padrão _Show_ e _Show Details_ se comportam da seguinte forma:

* _Show segue_: O _navigation controller_ envia o _view controller_ especificado para sua pilha de navegação.

* _Show Detail segue_: O _navigation controller_ apresenta o _view controller_ especificado de forma **modal**.

## Fazendo transições através de `UIStoryboardSegue`

Para além de se utilizar dos métodos da API de `UINavigationController`, para executar transições entre telas em uma navegação hierárquica é possível utilizar _segues_. Através do Interface Builder você pode utilizar objetos de `UIStoryboardSegue` para representar essas transições programadas na definição de seus arquivos de storyboard. Um `UIStoryboardSegue` é objeto que prepara e executa a transição visual entre dois _view controllers_.

### Overview

A classe `UIStoryboardSegue` suporta as transições visuais padrão disponíveis no UIKit _framework_. Você também pode criar subclasses para definir transições personalizadas entre os _view controllers_ em seu arquivo de storyboard.

Os objetos Segue contêm informações sobre os _view controllers_ envolvidos em uma transição. Quando uma _segue_ é acionada, antes que a transição visual ocorra, o runtime do storyboard chama o método `prepare(for:sender:)` do _view controller_ atual para que ele possa passar todos os dados necessários para o _view controller_ que está prestes a ser exibido.

Você não cria objetos _segue_ diretamente. Em vez disso, o runtime do storyboard os cria quando deve executar uma transição entre dois _view controllers_. Você ainda pode iniciar uma _segue_ programaticamente usando o método `performSegue(withIdentifier:sender:)` de `UIViewController`, se desejar. Você pode fazer isso para iniciar uma _segue_ de uma fonte que foi adicionada programaticamente e, portanto, não está disponível no Interface Builder, por exemplo.

#### Acessando atributos de uma _segue_

``` swift
var source: UIViewController
```
O _view controller_ de origem da _segue_.

``` swift
var destination: UIViewController
```
O _view controller_ de destino da _segue_.

``` swift
var identifier: String?
```
Uma String que identifica a _segue_.

#### Preparando para a transição com a _segue_

A partir da definição do Storyboard é possivel associar a execução de uma _segue_ a uma ação de botão ou outro elemento de View. Em diversas situações é preciso que informações sejam levadas para o contexto de apresentação do próximo _view controller_. Como mencionado acima, é possível sobrescrever o método `prepare(for:sender:)` para configurar tudo que for necessário para apresentar o segundo controlador adequadamente.

Segue abaixo um exemplo de código que prepararia para a transição cuja origem é o controlador base do app de Configurações do iOS e o destino é o controlador de segundo nível que apresenta um grupo de configurações:

``` swift
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        guard segue.identifier == "verGrupoDeConfigurações" else { return }
        
        guard let cell = sender as? SettingsMenuItemCell,
              let controller = segue.destination as? SettingsGroupViewController else {
            fatalError("Não foi possível executar a segue \(segue.identifier!)")
        }
        
        controller.settingsGroup = cell.group
    }
```
