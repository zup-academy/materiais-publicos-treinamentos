# Apresentando dados com `UICollectionView`

> Nota: Esse material teórico foi traduzido e adaptado da fonte original em: https://developer.apple.com/documentation/uikit/uicollectionview

Assim como é bastante comum o uso da tabelas que organizam múltiplos items através de linhas em uma coluna, uma série de aplicativos no mercado prevê alguma forma de apresentação de items em múltiplas colunas com layout customizado. Seja na própria app da loja de aplicativos do iPhone, como em diversos outros controles onde você _scrolla_ items com identidade visual de _cards_ temos exemplos destas visualizações. Esse material teórico visa introduzir o componente de `UICollectionView` do UIKit _framework_, que é a forma padrão de apresentação destas coleções de layout customizáveus das aplicações iOS.

## `UICollectionView`

>`@MainActor class UICollectionView : UIScrollView`

Um objeto que gerencia uma coleção de itens ordenados e os apresenta usando layouts customizáveis.

### Overview

Quando você adiciona uma _Collection View_ à sua UI, oatarefa principal do seu aplicativo é gerenciar os dados associados a essa _Collection View_. A _Collection View_ obtém seus dados a partir do objeto de _data souce_, armazenado na propriedade `dataSource` da _Collection View_. Para seu _data source_, você pode usar um objeto `UICollectionViewDataSource`. Alternativamente, você pode criar um objeto de _data source_ adotando o protocolo `UICollectionViewDiffableDataSource`, que fornece o comportamento necessário para gerenciar de forma simples e eficiente as atualizações dos dados da sua _Collection View_.

Os dados na _Collection View_ são organizados em itens individuais, que você pode agrupar em seções para apresentação. Um item é a menor unidade de dados que você pode apresentar. Por exemplo, em um aplicativo de fotos, um item pode ser uma única imagem. A _Collection View_ apresenta itens na tela usando uma célula, que é uma instância da classe `UICollectionViewCell`, configurada e fornecida pela implementação do _data source_.

<p align="center">
<img alt="Imagem contendo um overview básico de uma UICollectionView contendo agrupamento em seções, com uma Supplementary view em destaque acima, e um Cell em destaque abaixo" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/collection-views-teoria-ui-collection-view-imagem-overview.jpg?raw=true" width="100%" />
</p>

Além de suas células, uma _Collection View_ pode apresentar dados usando outros tipos de _views_. Essas _views_ complementares (ou _supplementary views_) podem ser, por exemplo, _headers_ e _footers_ de seção separados das células individuais. O suporte para _supplementary views_ é opcional e definido pelo objeto de layout da _Collection View_, que também é responsável por definir o posicionamento dessas _views_.

Além de incorporar uma `UICollectionView` na sua UI, você pode usar os métodos da _Collection View_ para garantir que a apresentação visual dos itens corresponda à ordem em seu objeto de _data source_. Um objeto `UICollectionViewDataSource` gerencia esse processo.

### Layouts

Um objeto de layout define a organização visual do conteúdo na _Collection View_. O objeto layout, uma subclasse de `UICollectionViewLayout`, define a organização e a localização de todas as células e visualizações suplementares dentro da _Collection View_. Embora defina seus posicionamentos, o objeto de layout não aplica essas informações às _views_ correspondentes na coleção. A própria _Collection View_ aplica informações de layout às _views_ correspondentes porque a criação de células e _views_ suplementares envolve a coordenação entre a _view_ da coleção e seu objeto _data source_. O objeto de layout é como outra _data source_, exceto que fornece informações visuais em vez de dados para os itens.

Normalmente, você especifica um objeto de layout ao criar uma _Collection View_, mas também pode alterar o layout de uma _Collection View_ dinamicamente. O objeto de layout é armazenado na propriedade `collectionViewLayout`.

### Células e Supplementary views

O objeto de _data source_ da _Collection View_ fornece o conteúdo dos itens e as _views_ usadas para apresentar esse conteúdo. Quando a _Collection View_ carrega seu conteúdo pela primeira vez, ela solicita que sua fonte de dados forneça uma _view_ para cada item visível. A _Collection View_ mantém uma fila (_queue_) ou lista de objetos de _view_ que o _data source_ disponibilizou para reutilização. Em vez de criar novas _views_ explicitamente em seu código, você sempre deve reutilizar as _views_ da fila.

Existem dois métodos para desenfileirar _views_. A que você usa depende de qual tipo de _view_ foi solicitada:

* Use o `dequeueReusableCell(withReuseIdentifier:for:)` para obter uma célula para um item na _Collection View_.
* Use o método `dequeueReusableSupplementaryView(ofKind:withReuseIdentifier:for:)` para obter uma _view_ suplementar solicitada pelo objeto de layout.

Antes de chamar qualquer um desses métodos, você deve informar à _Collection View_ como criar a _view_ correspondente, caso ainda não exista uma. Para isso, você deve registrar uma classe com a _Collection View_ através do método `register(_:forCellWithReuseIdentifier:)`. Como parte do processo de registro, você especifica o identificador de reutilização (_reuse identifier_) que é a mesma string que você usa para desenfileirar a _view_. Esse processo é habilitado automaticamente pelo uso da _Collection View_ através do Interface Builder ao informar os atributos _Identifier_ (no _Attributes Inspector_) e _Custom Class_ (no _Identity Inspector_).

Depois de desenfileirar a _view_ apropriada em seu método de _data source_, você deve configurar seu conteúdo e devolvê-la à _Collection View_ para uso. Depois de obter as informações de layout do objeto de layout, a _Collection View_ as aplica à _view_ e a apresenta.

### Atributos do Interface Builder

| Atributo | Descrição |
|:---------|:----------|
| Items | O número de protótipo de células. Esta propriedade controla o número especificado de protótipos de células para configurar em seu arquivo storyboard. As _Collection Views_ devem sempre ter pelo menos uma célula e podem ter várias células para exibir diferentes tipos de conteúdo ou para exibir o mesmo conteúdo de maneiras diferentes. |
| Layout | O objeto de layout a ser usado. Use este controle para selecionar entre o objeto `UICollectionViewFlowLayout` ou um objeto de layout personalizado que você define. Quando o _flow layout_ é selecionado, você também pode configurar a direção de rolagem para o conteúdo da _Collection View_ e se o _flow layout_ tem _views_ suplementares de _header_ e _footer_. A ativação das _views_ de _header_ e _footer_ adiciona _views_ reutilizáveis ao seu storyboard que você pode configurar com o conteúdo adequado. Você também pode criar essas exibições programaticamente. Quando um layout personalizado é selecionado, você deve especificar a subclasse `UICollectionViewLayout` a ser usada. |

Quando o _flow layout_ é selecionado, o _Size Inspector_ para a _Collection View_ contém atributos adicionais para configurar as dimensões de layout. Use esses atributos para configurar o tamanho de suas células, o tamanho dos _headers_ e _footers_, o espaçamento mínimo entre as células e quaisquer margens ao redor de cada seção de células. Para obter mais informações sobre o significado das propriedades de flow layout, consulte a documentação de [UICollectionViewFlowLayout](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout).
