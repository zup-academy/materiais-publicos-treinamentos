# Apresentando dados com UITableView

> Nota: Esse material teórico foi traduzido e adaptado das fontes originais disponíveis em:
>
> * https://developer.apple.com/documentation/uikit/uitableview
> * https://developer.apple.com/documentation/uikit/views_and_controls/table_views/filling_a_table_with_data
> * https://developer.apple.com/documentation/uikit/views_and_controls/table_views/configuring_the_cells_for_your_table
> * https://developer.apple.com/documentation/uikit/views_and_controls/table_views/adding_headers_and_footers_to_table_sections


A maior parte dos casos de uso de aplicativos do mercado prevê alguma forma de apresentação de conjuntos de dados organizados. Seja nas configurações do seu celular, na sua lista de contatos, no seu app favorito para pedir comida ou ouvir musica por streaming, estamos sempre acessando seções de múltiplos valores com diferentes usabilidades. Esse material teórico visa introduzir o componente de `UITableView` do UIKit _framework_, que é a forma padrão de apresentação de listas ou conjuntos de valores organizados das aplicações iOS.

## UITableView

Uma `UITableView` é essencialmente uma _view_ que apresenta dados usando linhas. As _table views_ no iOS exibem conteúdo através dessas linhas permitindo rolagem (_scroll_) vertical em única coluna. Cada linha na tabela contém uma parte do conteúdo do seu aplicativo. Por exemplo, o aplicativo Contatos exibe o nome de cada contato em uma linha separada e a página principal do aplicativo Configurações exibe os grupos de configurações disponíveis. 

<p align="center">
<img alt="Imagem contendo os exemplos de table view dos apps de contatos e configurações do iPhone" src="https://docs-assets.developer.apple.com/published/82818afa8d/3148900@2x.png" width="65%" />
</p>

Você pode configurar uma tabela para exibir uma única longa lista de linhas ou pode agrupar linhas relacionadas em seções para facilitar a navegação pelo conteúdo.

<p align="center">
<img alt="Imagem contendo os exemplos de configurações de estilo de table views, sendo eles: uma table view com estilo de seção agrupada, uma table view simples somente com uma lista longa de linhas e um table view com célular customizadas divididas em seções." src="https://docs-assets.developer.apple.com/published/2253500436/3148899@2x.png" width="75%" />
</p>

Tabelas são comuns em aplicativos com dados estruturados ou organizados hierarquicamente. Os aplicativos que contêm dados hierárquicos geralmente usam tabelas em conjunto com um _view controllers_ de navegação, o que facilita a navegação entre os diferentes níveis da hierarquia. Por exemplo, o aplicativo Configurações usa tabelas e um controlador de navegação para organizar as configurações do sistema.

`UITableView` gerencia a aparência básica da tabela, mas o código do seu aplicativo fornece a configuração das células (instâncias de `UITableViewCell`) que exibem o conteúdo real. As configurações de célula padrão exibem uma combinação simples de texto e imagens, mas você pode definir células personalizadas que exibem qualquer conteúdo desejado. Você também pode fornecer _views_ para _header_ e _footer_ para exibir informações adicionais para grupos de células ou mesmo para a tabela como um todo.

> Nota: As _table views_ são uma colaboração entre muitos objetos diferentes, incluindo:
>
>* Célula. Uma célula fornece a representação visual do seu conteúdo. Você pode usar as células padrão fornecidas pelo UIKit ou definir células personalizadas para atender às necessidades do seu aplicativo.
>
>* Data Source (seu objeto de fonte de dados). Este objeto adota o protocolo `UITableViewDataSource` e fornece os dados para a tabela através de uma interface de comunicação conhecida.
>
>* Delegate (seu objeto de delegate, para quem a table view delega uma série de responsabilidades). Este objeto adota o protocolo `UITableViewDelegate` e gerencia as interações do usuário com o conteúdo da tabela.
>
>* Table View Controller. Você normalmente usa um objeto `UITableViewController` para gerenciar uma _table view_. Você também pode usar outros _view controllers_, mas um Table View Controller é necessário para que alguns recursos relacionados à tabela funcionem no iOS.

### Adicionando uma Table View à sua interface

Para adicionar uma Table View à sua interface, arraste uma referência de `UITableView` para dentro de algum controlador no seu _storyboard_, ou um novo `UITableViewController`. O Xcode cria uma nova cena que inclui o _view controller_ e uma _table view_, pronta para você configurar e usar.

As _table views_ são orientadas por dados, normalmente obtendo-os de um objeto de _data source_ que você fornece. O objeto de _data source_ gerencia os dados do seu aplicativo e é responsável por criar e configurar as células da tabela.

### Provendo dados à uma Table View

As _table views_ são elementos da sua interface orientados por dados. Você fornece os dados do seu aplicativo, juntamente com as _views_ necessárias para renderizar cada parte desses dados na tela, usando um objeto de _data source_ - um objeto que adota o protocolo `UITableViewDataSource`. A _table view_ organiza suas _views_ na tela e trabalha com seu objeto de _data source_ para manter esses dados atualizados.

As _table views_ organizam seus dados em linhas e seções. As linhas exibem itens de dados individuais e as seções agrupam linhas relacionadas. As seções não são obrigatórias, mas são uma boa maneira de organizar dados que são hierárquicos. Por exemplo, o aplicativo Contatos exibe o nome de cada contato em uma linha e agrupa as linhas em seções com base na primeira letra do sobrenome da pessoa.

<p align="center">
<img alt="Imagem exemplificando separação visual do que é uma seção e o que é uma linha" src="https://docs-assets.developer.apple.com/published/0e083c5df8/tableview-basics@2x.png" width="35%" />
</p>

#### Fornecendo o número de Linhas e Seções

Antes de aparecer na tela, uma _table view_ solicita que você especifique o número total de linhas e seções. Seu objeto de _data source_ fornece essas informações usando dois métodos:

``` swift
func numberOfSections(in tableView: UITableView) -> Int // Opcional

func tableView(_ tableView: UITableView, seção numberOfRowsInSection: Int) -> Int
```

Ao implementar estes métodos, retorne as contagens de linhas e seções o mais rápido possível. Isso pode exigir que você estruture seus dados de uma maneira que facilite a recuperação das informações de linha e seção. Por exemplo, considere usar _arrays_ para gerenciar os dados da sua tabela. Arrays são boas ferramentas para organizar seções e linhas porque combinam com a organização natural da própria _table view_.

O código de exemplo abaixo mostra uma implementação dos métodos de _data source_ que retornam o número de linhas e seções em uma tabela de várias seções. Nesta tabela, cada linha exibe uma _string_, portanto, a implementação armazena um _array_ de _strings_ para cada seção. Para gerenciar as seções, a implementação usa um _array_ de _arrays_ (chamado `dadosHierarquicos`). Para obter o número de seções, o _data source_ retorna o número de itens no _array_ `dadosHierarquicos`. Para obter o número de linhas em uma seção específica, a fonte de dados retorna o número de itens no respectivo _array_ filho.

``` swift
var dadosHierarquicos = [[String]]()
 
override func numberOfSections(in tableView: UITableView) -> Int {
   return dadosHierarquicos.count
}
   
override func tableView(_ tableView: UITableView, 
                        numberOfRowsInSection section: Int) -> Int {
   return dadosHierarquicos[section].count
}
```

#### Definindo a aparência das linhas

Você define a aparência das linhas da sua tabela em seu arquivo de storyboard usando células. Uma célula é um objeto `UITableViewCell` que atua como um modelo para as linhas de sua tabela. As células são _views_ e podem conter quaisquer _subviews_ necessárias para exibir seu conteúdo. Você pode adicionar _labels_, _image views_ e outras _views_ ao seu conteúdo e organizar essas _views_ usando _constraints_ de layout. 

Quando você adiciona uma _table view_ à interface do seu aplicativo, ela inclui uma célula protótipo para você configurar. Para adicionar mais células protótipo, selecione a _table view_ e atualize seu atributo _Prototype Cells_. Cada célula tem um estilo, que define sua aparência. Você pode selecionar um dos estilos padrão fornecidos pelo UIKit ou definir seus próprios estilos personalizados.

A ilustração a seguir mostra uma tabela com duas células protótipo, cada uma delas usando um dos estilos de célula padrão.

<p align="center">
<img alt="Imagem mostrando uma table view com dois protótipos de célula diferentes adicionados em seu conteúdo no storyboard" src="https://docs-assets.developer.apple.com/published/b2a42cb85b/3148901@2x.png" width="35%" />
</p>

Em seu arquivo de storyboard, execute as seguintes ações para cada célula protótipo:

* Defina o estilo de célula como personalizado ou defina-o como um dos estilos de célula padrão.
* Atribua uma string não vazia ao atributo _Identifier_ da célula.
* Para células personalizadas, adicione _subviews_ e _constraints_ à célula.
* Especifique a classe de células personalizadas no _Identity Inspector_.

Ao criar uma célula com _views_ personalizadas, defina uma subclasse de `UITableViewCell` para gerenciar essas _views_. Em sua subclasse, adicione _outlets_ para as _views_ personalizadas que exibem os dados do seu aplicativo e conecte esses _outlets_ às _views_ reais em seu arquivo de storyboard. Você precisará dessas conexões para configurar a célula em tempo de execução.

#### Criando e configurando células para cada linha

Antes de uma _table view_ aparecer na tela, ela solicita que seu objeto de _data source_ forneça células para as linhas que estão na parte visível da tabela ou próximas desta parte visível. O método `tableView(_:cellForRowAt:)` do seu objeto de fonte de dados deve responder rapidamente. Implemente este método com o seguinte padrão:

1. Chame o método `dequeueReusableCell(withIdentifier:for:)` da _table view_ para recuperar um objeto de célula.
2. Configure as _views_ da célula com os dados personalizados do seu aplicativo.
3. Retorne a célula para a _table view_.

Para células com estilo padrão, `UITableViewCell` contém propriedades com as _views_ que você precisa configurar. Para células personalizadas, você adiciona _views_ à sua célula em tempo de design e _outlets_ para acessá-las. Uma boa prática é também fornecer métodos que facilitem a configuração das _subviews_ na implementação da sua `UITableViewCell` personalizada.

O código de exemplo abaixo mostra uma versão do método do _data source_ que configura uma célula contendo um único _label_. A célula usa o estilo `Basic`, um dos estilos de célula padrão. Para células de estilo básico, a propriedade `textLabel` de `UITableViewCell` contém uma _view_ de _label_ que você configura com seus dados.

``` swift
override func tableView(_ tableView: UITableView,
                        cellForRowAt indexPath: IndexPath) -> UITableViewCell {
   // 1
   let celula = tableView.dequeueReusableCell(withIdentifier: "basicStyleCell", for: indexPath)
        
   // 2
   celula.textLabel!.text = "Linha \(indexPath.row)"
   return celula
}

// O código acima divide sua implementação entre:
//   1. Pedir uma célula do tipo apropriado; e
//   2. Configurar o conteúdo da célula com o número da linha. O estilo de célula Basic garante que um _label_ esteja presente em textLabel.
```



As _table views_ não solicitam que você crie células para cada uma das linhas da tabela. Em vez disso, as _table views_ gerenciam as células utilizando _lazy loading_, solicitando apenas as células que estão dentro ou próximas à parte visível da tabela. Criar células através de _lazy loading_ reduz a quantidade de memória que a tabela usa. No entanto, isso também significa que seu objeto de _data source_ deve criar células rapidamente. Não use o método `tableView(_:cellForRowAt:)` para carregar os dados da sua tabela ou realizar operações demoradas.

> Nota: Além de usar os estilos de célula padrão, você pode definir células personalizadas que contenham todas as _views_ desejadas. Veremos mais sobre isso a seguir.

### Configurando células personalizadas para sua tabela

As células fornecem a representação visual das linhas da sua tabela. Para a maioria das tabelas, você fornece apenas um ou dois tipos diferentes de células. Projete suas células para garantir que as informações mais importantes se destaquem. Faça isso usando uma escolha cuidadosa de _views_ e configurações de _view_ em suas células.

Você especifica a aparência das células em tempo de design em seu arquivo de storyboard. O Xcode fornece uma célula protótipo para cada tabela e você pode adicionar mais células protótipo conforme necessário. Uma célula protótipo funciona como um modelo para a aparência da sua célula. Inclui as _views_ que você deseja exibir e sua organização na área de conteúdo da célula. Em tempo de execução, o objeto de _data source_ da tabela cria células reais dos protótipos e as configura com os dados do seu aplicativo.

<p align="center">
<img alt="Imagem mostrando uma table view no storyboard com algumas células protótipo vazias" src="https://docs-assets.developer.apple.com/published/a5b5fd18a8/3148904@2x.png" width="35%" />
</p>

#### Atribuindo um _reuse identifier_ a cada célula

Os identificadores de reutilização (_reuse identifiers_) facilitam a criação e a reciclagem das células da sua tabela. Um _reuse identifier_ é uma _string_ que você atribui a cada protótipo de célula da sua tabela. Em seu storyboard, selecione seu protótipo de célula e atribua um valor não vazio ao seu atributo _Identifier_. Cada célula em uma _table view_ deve ter um _reuse identifier_ exclusivo.

Quando você precisar de um objeto de célula em tempo de execução, chame o método `dequeueReusableCell(withIdentifier:for:)` da _table view_, passando o _reuse id_ para a célula desejada. A _table view_ mantém uma fila interna de células já criadas. Se a fila contiver uma célula do tipo solicitado, a _table view_ retornará essa célula. Caso contrário, ele cria uma nova célula usando o protótipo de célula em seu storyboard. A reutilização de células melhora o desempenho minimizando as alocações de memória durante momentos críticos, como durante o _scroll_.

``` swift
var celula = tableView.dequeueReusableCell(withIdentifier: "myCellType", for: indexPath)
```

#### Configurar uma célula com um estilo integrado

A maneira mais simples de configurar uma célula é usar um dos estilos internos fornecidos por `UITableViewCell`. Você usa esses estilos como estão; você não precisa fornecer uma subclasse customizada para gerenciar a célula. Cada estilo incorpora um ou dois _labels_, com o estilo determinando as posições dos _labels_ na área de conteúdo da célula. A maioria dos estilos também incorpora uma imagem na borda principal do conteúdo da célula.

Para configurar um protótipo de célula com um dos estilos padrão, selecione a célula em seu storyboard e defina a propriedade `Style` da célula com um valor diferente de `Custom`.

<p align="center">
<img alt="Imagem mostrando uma table view no storyboard com células protótipo de todos os estilos, sendo eles: basic, right detail, left details, subtitle" src="https://docs-assets.developer.apple.com/published/17a7f11520/3148906@2x.png" width="35%" />
</p>

Em seu método `tableView(_:cellForRowAt:)`, configure o conteúdo de sua célula usando as propriedades `textLabel`, `detailTextLabel` e `imageView` de `UITableViewCell`. Essas propriedades contêm _views_, mas o objeto de célula só atribui uma _view_ se o estilo oferecer suporte ao conteúdo correspondente. Por exemplo, o estilo de célula `Basic` não oferece suporte a uma string de detalhes, portanto, a propriedade `detailTextLabel` é nula para esse estilo. O código de exemplo a seguir mostra como configurar uma célula que usa o estilo de célula `Basic`.

``` swift
override func tableView(_ tableView: UITableView,
                        cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    
    // 1
    let celula = tableView.dequeueReusableCell(withIdentifier: "basicStyle", for: indexPath)

    // 2
    celula.textLabel!.text = "Texto do título"
    celula.imageView!.image = UIImage(named: "Flor")
    return celula
}

// O código acima divide sua implementação entre:
//   1. Pedir para reutilizar ou criar uma célula; e
//   2. Para uma célula de estilo padrão, usar as propriedades de UITableViewCell para configurar os valores.
```

#### Configurando uma célula com _views_ customizadas

Para aparências diferentes dos estilos padrão, use o estilo de célula `Custom`. Com uma célula customizada, você especifica as _views_ que deseja na célula, suas configurações, tamanhos e posições. _Views_ estáticas, como _labels_ e imagens, são o melhor conteúdo para células. Evite exibições que exijam interações do usuário, como controles. Não inclua _scroll views_ (_views_ com suporte à _scroll_, especialmente de mesma direção ao da tabela), _table views_, _collection views_ ou outras _views_ de contêiner complexas nas células. Você pode incluir _stack views_ em suas células, mas tenha em mente que minimizar o número de itens em sua _stack view_ pode melhorar o desempenho da tabela.

Para configurar uma célula personalizada, arraste as _views_ para o protótipo de célula de sua tabela. A ilustração a seguir mostra uma célula com layout e formatação personalizados para suas _views_. Você usa _constraints_ para posicionar suas _views_ na área de conteúdo da célula (_cell content view_). Ao configurar _constraints_, prefira a opção _"Constrain to margins"_ para preservar o respiro entre as áreas de conteúdo de suas células.

<p align="center">
<img alt="Imagem mostrando uma table view no storyboard com célula customizada, contendo subviews para titulo, descrição e uma image view posicionada do lado direito através de constraints, gerenciadas de forma direta ou através de stack views" src="https://docs-assets.developer.apple.com/published/08c9b2469b/3148905@2x.png" width="35%" />
</p>

Para células customizadas, você precisa definir uma subclasse de `UITableViewCell` para acessar as _views_ de sua célula. Adicione _outlets_ à sua subclasse e conecte esses _outlet_ às _views_ correspondentes em seu protótipo de célula.

``` swift
class RefeicaoTableViewCell: UITableViewCell {
    @IBOutlet var nome: UILabel?
    @IBOutlet var descricao : UILabel?
    @IBOutlet var imagem: UIImageView?
}
```

No método `tableView(_:cellForRowAt:)` do seu _data source_, use os _outlets_ de sua célula para atribuir valores às _views_.

``` swift
override func tableView(_ tableView: UITableView,
                        cellForRowAt indexPath: IndexPath) -> UITableViewCell {

   // 1
   let celula = tableView.dequeueReusableCell(withIdentifier: "refeicaoCell", for: indexPath) as! RefeicaoTableViewCell

   // 2
   let refeicao = refeicoes[indexPath.row]
        
   // 3
   celula.nome?.text = refeicao.nome
   celula.descricao?.text = refeicao.descricao
   celula.imagem?.image = refeicao.imagem
        
   return celula
}

// O código acima divide sua implementação entre:
//   1. Pedir para reutilizar ou criar uma célula do tipo apropriado.
//   2. Buscar os dados para a linha.
//   3. Configurar o conteúdo da célula com dados do objeto buscado.
```

> Nota: Ao invés de pedir a um objeto por seus dados (no exemplo, as referências para suas _views_ customizadas) e trabalhar sobre eles, você deveria dizer ao objeto o que fazer. Isso tende a levar o comportamento para perto de onde os dados estão e maximizar o uso de `self`. Orientação a Objetos é justamente sobre isso. Esse príncipio ou modelo de pensamento é conhecido como _Tell-Don't-Ask_.
>
>O comportamento ficaria melhor compartimentado através de `celula.setup(refeicao)`. Assim o acoplamento do código do _data source_ com a célula se dá pela interface da função `setup(_:)` e, desta forma, os detalhes de configuração dos valores para a célula podem mudar livremente sem quebrar a implementação do _data source_.

#### Alterando a altura das linhas

Uma _table view_ rastreia a altura das linhas separadamente das células que as representam. `UITableView` fornece tamanhos padrão para linhas, mas você pode substituir a altura padrão atribuindo um valor personalizado à propriedade `rowHeight` da _table view_. Sempre use esta propriedade quando a altura de todas as suas linhas for a mesma. Fazer isso é mais eficiente do que retornar os valores de altura através de seu objeto _delegate_.

Se as alturas das linhas não forem todas iguais ou puderem ser alteradas dinamicamente, forneça as alturas usando o método `tableView(_:heightForRowAt:)` do seu objeto _delegate_. Ao implementar esse método, você deve fornecer valores para cada linha em sua tabela. O código de exemplo a seguir mostra como retornar uma altura personalizada para a primeira linha de cada seção e usar a altura padrão para todas as outras linhas.

``` swift
override func tableView(_ tableView: UITableView, 
                        heightForRowAt indexPath: IndexPath) -> CGFloat {
    // 1
    if indexPath.row == 0 {
      return 80
    }

    // 2
    return UITableView.automaticDimension
}

// O código acima divide sua implementação entre:
//   1. Aumentar a altura da primeira linha para acomodar uma célula personalizada.
//   2. Usar o tamanho padrão para todas as outras linhas.
```

#### Restaurando a aparência original da sua célula antes de reutilizá-la

Quando uma célula sai da tela, a _table view_ a remove de sua hierarquia de _views_ e a coloca em uma fila de reciclagem gerenciada internamente. Quando você solicita uma nova célula usando o método `dequeueReusableCell(withIdentifier:for:)` da _table view_, a _table view_ retorna primeiro as células da fila de reciclagem. Se a fila estiver vazia, a _table view_ instancia uma nova célula do seu storyboard.

Se você alterar a aparência das _views_ de sua célula personalizada, sobrescreva o método `prepareForReuse()` em sua subclasse de célula. Na implementação do método, retorne a aparência das _views_ de sua célula ao estado original. Por exemplo, se você alterar a propriedade `alpha` de uma _view_ da sua célula, retorne essa propriedade ao seu valor original. Você não precisa limpar o texto do _label_, definir imagens como `nil` ou fazer qualquer coisa que a implementação do método `tableView(_:cellForRowAt:)` já corrija ao configurar a célula para exibição.

### _Headers_ e _Footers_ para seções da tabela

Adicionar _views_ de _header_ e _footer_ às seções da _table view_ ajudam a diferenciar visualmente grupos de linhas. Use as _views_ de _header_ e _footer_ como marcadores visuais para o início e o fim das seções. As _views_ de _header_ e _footer_ são opcionais e você pode personalizá-las livremente.

<p align="center">
<img alt="Imagem exemplificando separação visual de seções através do uso de uma header view como marcador de início e footer view como marcador de final" src="https://docs-assets.developer.apple.com/published/22a5331c82/header-footer-sample@2x.png" width="35%" />
</p>

Para criar um _header_ ou _footer_ básico com um _label_ simples, sobrescreva o método `tableView(_:titleForHeaderInSection:)` ou `tableView(_:titleForFooterInSection:)` do objeto de _data source_ da sua tabela. A _table view_ cria então um _header_ ou _footer_ padrão para você e o insere na tabela no local especificado.

``` swift
// Cria um header padrão com o texto retornado pelo método
override func tableView(_ tableView: UITableView, 
                        titleForHeaderInSection section: Int) -> String? {
   return "Header \(section)"
}

// Cria um footer padrão com o texto retornado pelo método
override func tableView(_ tableView: UITableView, 
                        titleForFooterInSection section: Int) -> String? {
   return "Footer \(section)"
}
```

> Nota: _Headers_ e _footers_ normalmente se aplicam a uma única seção, mas você também pode fornecer uma _view_ de _header_ ou _footer_ para a tabela inteira usando as propriedades `tableHeaderView` ou `tableFooterView` de sua _table view_. O _header_ global aparece na parte superior do conteúdo da sua tabela e o _footer_ global aparece na parte inferior.
>
> Simplesmente passe uma referência de UIView para a propriedade `tableHeaderView` ou `tableFooterView` no carregamento de seu _view controller_ e ela automaticamente será disposta no local adequado dentro do conteúdo _scrollavel_ da _table view_. Certifique-se apenas de definir uma altura para o frame da _view_.

#### Personalizando as _views_ de _header_ e _footer_

As _views_ de _header_ e _footer_ personalizadas dão às seções da sua tabela uma identidade visual. Com _headers_ e _footers_ personalizados, você especifica as _subviews_ desejadas e as posiciona em qualquer lugar dentro do espaço alocado. Você também pode fornecer diferentes _views_ de _header_ ou _footer_ para diferentes seções de sua tabela.

Para criar _views_ personalizadas de _header_ ou _footer_:

1. Use objetos de `UITableViewHeaderFooterView` para definir a aparência.
2. Registre seus objetos de `UITableViewHeaderFooterView` na sua _table view_.
3. Implemente os métodos `tableView(_:viewForHeaderInSection:)` e `tableView(_:viewForFooterInSection:)` no objeto _delegate_ da _table view_ para criar e configurar suas _views_ para cada seção adequadamente.

Sempre use um objeto de `UITableViewHeaderFooterView` para seus _headers_ e _footers_. Essa _view_ suporta o mesmo modelo de reutilização que as células empregam, permitindo que você recicle _views_ em vez de criá-las a esmo. Depois de registrar sua _view_ de _header_ ou _footer_, use o método `dequeueReusableHeaderFooterView(withIdentifier:)` da _table view_ para solicitar instâncias dessa _view_. Se as _views_ de _header_ ou _footer_ recicladas estiverem disponíveis, a _view_ de tabela as retornará primeiro, antes de criar novas.

Você pode usar um objeto `UITableViewHeaderFooterView` como está e adicionar _views_ à sua propriedade `contentView` ou pode sobrescrever a classe e adicionar suas _views_. Posicione suas _subviews_ dentro da _content view_ usando uma _stack view_ ou _constraints_ de Auto Layout. Para alterar o _background_ por trás do seu conteúdo, modifique a propriedade `backgroundView` da _header footer view_. O código de exemplo a seguir mostra uma exibição de _header_ personalizada que posiciona as _views_ de imagem e _label_ no momento da criação.

``` swift
class HeaderPersonalizado: UITableViewHeaderFooterView {
    let imagem = UIImageView()
    let titulo = UILabel()

    override init(reuseIdentifier: String?) {
        super.init(reuseIdentifier: reuseIdentifier)
        setup()
    }

    func setup() {
        imagem.translatesAutoresizingMaskIntoConstraints = false
        titulo.translatesAutoresizingMaskIntoConstraints = false
        
        contentView.addSubview(imagem)
        contentView.addSubview(titulo)

        // Centraliza a imagem verticalmente e a coloca perto da borda 
        // de leading da view. Define sua largura e altura para 50 pontos.
        NSLayoutConstraint.activate([
            imagem.leadingAnchor.constraint(equalTo: 
                contentView.layoutMarginsGuide.leadingAnchor),
            imagem.widthAnchor.constraint(equalToConstant: 50),
            imagem.heightAnchor.constraint(equalToConstant: 50),
            imagem.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
        
            // Centraliza o label verticalmente e o utiliza para 
            // preencher o espaço restante na header view.
            titulo.heightAnchor.constraint(equalToConstant: 30),
            titulo.leadingAnchor.constraint(equalTo: 
                imagem.trailingAnchor, constant: 8),
            titulo.trailingAnchor.constraint(equalTo: 
                contentView.layoutMarginsGuide.trailingAnchor),
            titulo.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
        ])
    }
}
```

> Nota: A _view_ acima é construída e tem sua aparência configurada inteiramente através de código Swift utilizando as APIs do framework UIKit sem o auxílio de arquivos de Interface Builder, numa abordagem chamada de _View Code_.

Registre sua _view_ de _header_ como parte da configuração de sua _table view_.

``` swift
override func viewDidLoad() {
   super.viewDidLoad()
   
   // Registra a header view customizada.
   tableView.register(HeaderPersonalizado.self, 
       forHeaderFooterViewReuseIdentifier: "sectionHeader")
}
```

No método `tableView(_:viewForHeaderInSection:)` do seu objeto _delegate_, crie e configure sua _view_. O código de exemplo a seguir faz o _dequeuing_ (desenfileira, para fins de reciclagem/reutilização) de uma _header view_ registrada e configura suas propriedades de título e imagem.

``` swift
override func tableView(_ tableView: UITableView, 
                        viewForHeaderInSection section: Int) -> UIView? {
    
    let view = tableView.dequeueReusableHeaderFooterView(withIdentifier:
               "sectionHeader") as! HeaderPersonalizado
    view.titulo.text = sections[section]
    view.imagem.image = UIImage(named: sectionImages[section])

    return view
}
```

A imagem a seguir mostra um possível resultado.

<p align="center">
<img alt="Imagem exemplificando o uso de uma header view customizada marcando visualmente o início de uma nova seção, apresentando uma imagem alinhada à esquerda e um label à direita da imagem" src="https://docs-assets.developer.apple.com/published/d45e547c35/3148907@2x.png" width="35%" />
</p>

#### Alterando a altura dos _headers_ e _footers_

Uma _table view_ rastreia a altura dos _headers_ e _footers_ da seção separadamente das _views_ que os representam. `UITableView` fornece tamanhos padrão para ambos os itens. Se todos os seus _headers_ (ou todos os seus _footers_) tiverem a mesma altura, especifique essa altura usando as propriedades `sectionHeaderHeight` e `sectionFooterHeight` da _table view_ ou use os valores padrão fornecidos pela _table view_.

Se as alturas do _header_ ou _footer_ não forem todas iguais ou puderem ser alteradas dinamicamente, forneça as alturas usando os métodos `tableView(_:heightForHeaderInSection:)` e `tableView(_:heightForFooterInSection:)` do seu objeto _delegate_. Ao implementar esses métodos, você deve fornecer valores para cada _header_ e/ou _footer_ em sua tabela com suas possíveis diferenças de implementação.
