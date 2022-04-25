# Seguindo os passos para implementar um TableViewController para a tela da conta

Neste passo a passo construiremos juntos a funcionalidade para a tela de consulta ao estado de uma conta em um pseudo-aplicativo de banco digital. Ao final da atividade o aplicativo deve se parecer com a especificação abaixo.

<p align="center">
<img alt="Imagem da especificação de tela" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-lt2-especificacao-alvo.jpg?raw=true" width="80%"/>
</p>

## Preparando a visualização do nosso Table View Controller

Nossos primeiros passos nesta implementação serão preparando a camada de visualização do nosso aplicativo. Com a _view_ ajustada teremos ainda mais contexto sobre a utilização do app e faremos com que os próximos passos sejam mais intuitivos. No entanto, antes de mais nada, abra o projeto, _builde_ e rode o aplicativo, se familiarize com o estado atual do mesmo e sua organização.

<p align="center">
<img alt="Imagem com o estado inicial do projeto no Xcode" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-lt2-imitation-estado-inicial.png?raw=true" width="80%"/>
</p>

### Configurando o controlador inicial

Com o _tour_ pelo projeto feito. Vamos começar nosso desenvolvimento.

Você deve ter notado que tanto na representação do storyboard como no código, nosso controlador ainda não oferece suporte à exibição de uma _table view_ para exibir os múltiplos itens de transação da conta. Esse será nosso primeiro problema a resolver.

Abra o storyboard, selecione a representação do _View Controller_ (atenção, não o _Navigation Controller_) e pressione _delete_ para removê-lo. Precisamos no lugar dele, de um _Table View Controller_.

* Clique no ícone superior direito _(+)_ do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"table"_.
* Utilize o _drag and drop_ para arrastar um elemento de Table View Controller para o canvas, posicionando-o ao lado direito da outra representação de controlador existente
* Pressionando a tecla _control ⌃_, clique sobre a representação de controlador à esquerda e arraste para o novo Table View Controller adicionado. No _pop-up_ que solicita a informação da relação entre os controladores selecione a opção _root view controller_.

<p align="center">
<img alt="Animação com demonstração do passo anterior de relação entre os controladores" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-lt2-imitation-tv-como-root.gif?raw=true" width="80%"/>
</p>

Com isso feito, temos nosso _table view controller_ apresentado logo ao carregar o app.

Um ajuste importante antes de prosseguirmos para a ligação com o código do arquivo `ViewController.swift` é dar a esta tela um nome no fluxo de utilização do aplicativo.

* No _Document Outline_, abaixo da referência de Table View, existe uma referência a Navigation Item. Clique sobre a mesma.
* Agora no _Attributes Inspector_ à direita, digite _Conta_ para o _Title_ dessa tela.

#### Refatorando o código do View Controller base

Para que as recentes alterações no storyboard estejam em conformidade com o código da camada de controle da aplicação, precisamos de ajustes no arquivo `ViewController.swift` dentro da pasta `Scenes/`.

* Abra o arquivo `ViewController.swift`.
* Para que tenhamos um nome com mais contexto para o arquivo, clique com o botão direito do mouse sobre o identificador da classe e selecione _Refactor > Rename_.
* No estado de edição para o nome da classe, substitua o nome atual por _ContaViewController_.
* Faça com que a classe herde de `UITableViewController` alterando o código de declaração para `class ContaViewController: UITableViewController`.
* Volte ao arquivo `Main.storyboard` e selecione a referência do TableViewController.
* No _Identity Inspector_, para o seletor _Class_ da seção _Custom Class_, indique a classe `ContaViewController`.  

<p align="center">
<img alt="Animação com demonstração dos passos anteriores de ligacao do table view controller com o codebase atual" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-lt2-imitation-ligacao-table-view-codigo.gif?raw=true" width="80%"/>
</p>

Temos o _setup_ base do nosso Table View Controller pronto. Nesse momento ao rodar a aplicação tudo deve funcionar e a tela de _Conta_ deve ser exibida inicialmente. _(Caso não tenha este estado atualmente, retorne aos passos anteriores para verificar se algo saiu errado)._

### Ajustando o estilo da tabela e desenhando a Table View Cell para uma transação

Com a configuração pronta, agora a missão é focar em construir a visualização necessária para a tabela e suas células. Comece ajustando sua Table View.

* No _Document Outline_, selecione a referência da Table View.
* Agora no _Attributes Inspector_, para o seletor _Style_ da Table View, selecione a opção _Grouped_.
* Para o seletor _Separator Inset_, selecione a opção _Custom_.
* Nos _inputs_ numéricos que foram revelados, selecione `0` para ambos _Left_ e _Right_. Com isso as linhas separadores das células atravessarão toda a tela horizontalmente, o que é esperado pela especificação de design da tarefa.

A configuração de propriedades deve ficar como a da imagem abaixo.

<p align="center">
<img alt="Imagem com o estado do Attributes Inspector para a table view" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-lt2-imitation-atributos-table-view.png?raw=true" width="25%"/>
</p>

Agora passamos a cuidar do design para a célula da tabela. Vamos dar uma nova olhada na especificação de design alvo para pensar em uma possível abordagem.

<p align="center">
<img alt="Imagem com o exemplo do design apenas da célula da tabela" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-lt2-imitation-especificacao-alvo-celula.jpg?raw=true" width="25%"/>
</p>

Repare nas características da célula na imagem. É possível perceber padrões de grupos de elementos posicionados ao lado ou abaixo uns dos outros. Além de ficar evidente - como já previam os requisitos do exercício - que alguns _labels_, a depender do tipo da transação, podem estar revelados ou ocultos fazendo com que a altura da célula como um todo varie.

Começe pensando numa possível decomposição em camadas dessa _view_ tão particular. É possível supor uma camada base, mais profunda, responsável por assegurar a aplicação das margens do layout e posicionar dois principais grupos horizontalmente. Se você olhou pra essa imagem e já imaginou um primeiro Stack View horizontal, já está indo na direção correta. 

Repare também que na situação onde a altura é mais expandida, o design nos mostra que a imagem com o ícone, por exemplo, não sofre alteração no seu posicionamento e permanece mais próxima ao topo da célula.

* Clique no ícone superior direito _(+)_ do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"horiz"_.
* Utilize o _drag and drop_ para arrastar um elemento de Horizontal Stack View para o _Document Outline_, dentro da Content View da Table View Cell.
* Altere o identificador no _Document Outline_ para `Conteiner Stack View`.

Aproveite para já alterar o valor de alguns atributos nos inspetores de _Attributes_ e _Size_.

* No _Attributes Inspector_, selecione a opção _Top_ para a propriedade _Alignment_, deixando a propriedade _Distribution_ com o padrão _Fill_.
* Selecione também o valor `24` para o input numérico da propriedade _Spacing_.
* No _Size Inspector_ selecione _Language Directional_ para a propriedade _Layout Margins_.
* Para os inputs numéricos revelados, configure `24` para _Leading_ e _Trailing_ e `20` para _Top_ e _Bottom_.

Por fim, para que o componente se ajuste corretamente à caixa da Content View, adicione _constraints_ assegurando ligando todos as bordas dos componentes.

* Certifique-se de estar com o `Conteiner Stack View` selecionado.
* Clique no ícone inferior direito _Add New Constraints_.
* Marque todas as extremidades do box que indica _contraints_ de posicionamento utilizando `0` como constante.
* Clique no botão _Add 4 Constraints_.

Pronto! A camada base para a célula está pronta. Podemos pensar uma camada acima.

Pensando nos componentes internos dessa camada base, podemos identificar dois ou três grupos, a depender do que queiramos favorecer. Podemos no mínimo propor uma separação entre a imagem do ícone e do grupo de informações sobre a transação à direito, ou propor uma terceira _view_ para agrupar a data à direita. Por mais que o design não informa específicamente essa regra, vamos optar por dar o maior espaço possível para horizontalmente para os labels como por exemplo do **interessado** ou **valor**. Dessa forma, vamos com a primeira abordagem, adicionando uma imagem e um stack view vertical.

* Clique no ícone superior direito _(+)_ do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"uiv"_.
* Utilize o _drag and drop_ para arrastar um elemento de View para o _Document Outline_, dentro da Conteiner Stack View.
* No _Attributes Inspector_, selecione _Secondary System Background Color_ para cor de fundo da mesma. 
* Com a nova View ainda selecionada, no _Identity Inspector_, selecione o componente customizado `TransacaoImageView` para o seletor _Class_ da seção _Custom Class_.

Com isso temos o componente que propõe a implementação do ícone customizado já posicionado. Vamos então ao stack view.

* Clique no ícone superior direito _(+)_ do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"vert"_.
* Utilize o _drag and drop_ para arrastar um elemento de Vertical Stack View para o _Document Outline_, dentro da Conteiner Stack View e abaixo de Transacao Image View.
* Altere o identificador para a stack para `Main Stack View`.
* No _Attributes Inspector_, assegure que a stack tenha as opções padrão _Fill_ para _Alignment_ e _Distribution_, além de selecionar `4` para o input numérico para _Spacing_.

Não se preocupe se até o momento o Interface Builder estiver indicando problemas de Auto Layout. Ainda não é possível determinar o tamannho intríseco para os componentes internos e isso causa uma ambiguidade. Nossas próximas ações já irão resolver este problema.

Com mais uma camada estabelecida, podemos pensar na próxima. Dentro de Main Stack View podemos perceber dois labels principais para título e data que são dispostos um ao lado do outro. Além destes, os demais são dispostos um abaixo do outro, portanto já atendidos pela configuração de posicionamento de Main Stack View.

* Clique no ícone superior direito _(+)_ do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"horiz"_.
* Utilize o _drag and drop_ para arrastar um elemento de Horizontal Stack View para o _Document Outline_, dentro de Main Stack View.
* Altere o identificador para a nova stack para `Titulo Stack View`.
* No _Attributes Inspector_, assegure que a stack tenha as opções padrão _Fill_ para _Alignment_ e _Distribution_, além de selecionar `12` para o input numérico para _Spacing_.

Agora podemos passar aos labels.

* Clique no ícone superior direito _(+)_ do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"uil"_.
* Utilize o _drag and drop_ para arrastar um elemento de Label para o _Document Outline_, dentro de Titulo Stack View.
* Altere o identificador para o label para `Titulo Label`.
* No _Attributes Inspector_, altere o valor do Titulo Label para "Transferência enviada".
* Para o seletor de _Font_ selecione a opção _Bold_ para o estilo e `14` para o tamanho.
* No _Size Inspector_, na seção _Content Hugging Priority_, diminua o valor para o input numérico _Horizontal_ para `250`. 

Com o título configurado é hora do label para a data da transação.

* Clique no ícone superior direito _(+)_ do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"uil"_.
* Utilize o _drag and drop_ para arrastar um elemento de Label para dentro de Titulo Stack View, abaixo do Titulo label no **_Document Outline_** (atenção, não no canvas, abaixo em relação à árvore do _Document Outline_).
* Altere o identificador para a novo label para `Data Label`.
* No _Attributes Inspector_, altere o valor de Data Label para "12 SET.".
* Para o seletor de _Color_, selecione a opção _Secondary Label Color_.
* Para o seletor de _Font_ selecione a opção _Light_ para o estilo e `14` para o tamanho.

Temos então toda nossa estrutura de layout para a célula definida, faltando apenas adicionar alguns labels restantes.

* Pressionando a tecla _option ⌥_, clique sobre o `Data Label` e arraste uma cópia do mesmo para baixo do Titulo Stack View dentro da Main Stack View, aproveitando o estilo predefinido.
* Selecione a referência copiada no _Document Outline_ e aproveite para alterar seu identificador para `Interessado Label`
* Altere também seu valor textual para "Rafael Rollo" (ou o seu nome),no _Attributes Inspector_.
* Mais abaixo no _Attributes Inspector_, marque o _checkbox_ _Hidden_ já que nem toda célula tem o label para o interessado na transação visível. O label vai sumir do canvas, porém continua na árvore de visualizações.

Agora vamos ao label para o valor.

* Pressionando a tecla _option ⌥_, clique sobre o `Interessado Label` e arraste uma cópia do mesmo para baixo de sua referência no _Document Outline_.
* Selecione a referência copiada no _Document Outline_ e altere seu identificador para `Valor Transacao Label`.
* Altere seu valor textual para "R$ 2.500,00" (ou qualquer outro valor de exemplo em real), no _Attributes Inspector_.
* Mais abaixo no _Attributes Inspector_, desmarque o _checkbox_ _Hidden_ que veio marcado como cópia do anterior. Assim o componente vai reaparecer no canvas.

Por último temos o label para o subtipo de transação, que também não é visível para todos os tipos de transação. Vamos copiar um novo então a partir do Interessado Label, para aproveitar essa característica.

* Pressionando a tecla _option ⌥_, clique sobre o `Interessado Label` e arraste uma cópia do mesmo para baixo do Valor Transacao Label no _Document Outline_.
* Selecione a referência copiada no _Document Outline_ e altere seu identificador para `Subtipo Transacao Label`.
* Altere também seu valor textual para "Pix", no _Attributes Inspector_.
* No seletor de _Font_ altere o tamanho para `12`.

> Nota: Caso o box da view responsável por renderizar o ícone esteja aparentemente grande no canvas, não se preocupe. O conteúdo da classe `TransacaoImageView` permanece comentado até o momento para evitar problemas de _build_ e é impossível para o Xcode inferir seu tamanho intrínseco ao desenhar no Interface Builder. Conforme prosseguirmos com o código será possível determinar as dimensões adequadas para o componente e a prévia será mais fiel ao resultado esperado.

Atualmente o estado do `Main.storyboard` no Interface Builder deve se parecer com a imagem abaixo.

<p align="center">
<img alt="Animação com demonstração dos passos anteriores de ligacao do table view controller com o codebase atual" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-lt2-imitation-estado-final-interface-builder.png?raw=true" width="80%"/>
</p>

Já temos tudo pronto para continuar nossa implementação.

## Dando vida aos componentes do design

Agora que já temos nossa representação no storyboard pronto, podemos nos voltar para o código do Table View Controller.

### Provendo implementação para os métodos de UITableViewDataSource e UITableViewDelegate

Ao trabalhar com a instância de Table View Controller, já temos a referência direta para sua view de base `tableView`, além de já termos configurada a relação entre o controlador como objeto de fonte de dados e objeto delegado para as possíveis interações com elementos que a tabela apresenta. Assim já podemos avançar para fornecer implementação aos métodos que os protocolos `UITableViewDataSource` e `UITableViewDelegate` designam.

Começando pelos métodos que visam prover os dados, abra o arquivo `ContaViewController` e adicione implementação para chegar ao estado do código abaixo.

``` swift
class ContaViewController: UITableViewController {

    // override func viewDidLoad() { ... }
        
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 3
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = UITableViewCell()
        cell.textLabel?.text = "Teste" 
        return cell
    }
    
}
```

Com o código atual conseguimos exibir um exemplo simples com três linhas de texto fixo na nossa tabela. Mas além do fato do código acima servir apenas para colocarmos no lugar os _snippets_ para os métodos que necessitamos sobrescrever, temos também dois grandes problemas: (1) não estamos fazendo bom uso dos recursos de table view que visam otimizar o uso de memória ao renderizar os elementos e (2) não estamos apresentando nossa célula customizada.

Vamos começar pelo primeiro. Para que seja possível reaproveitar os objetos através do pool de células que a table view oferece, precisamos fornecer um `reuseIdentifier` para nossa célula.

* Abra o arquivo `Main.storyboard` e selecione a referência para Table View Cell no _Document Outline_.
* No _Attributes Inspector_, digite "TransacaoCell" para o input _Identifier_.
* Volte ao arquivo `ContaViewController` e altere sua implementação para que tenhamos o seguinte estado.

``` swift
    // código anterior omitido

    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "TransacaoCell", for: indexPath)
        cell.textLabel?.text = "Teste"
        return cell
    }
```

Com esta implementação já fazemos um uso mais consciente de memória na aplicação. Através do uso de `dequeueReusableCell(withIdentifier:for:)`, antes de decidir por criar uma nova instância a implementação da Table View vai verificar se existe uma disponível para reuso no _pool_ (ou _queue_) de Table View Cell. Se houver, a referência para esse objeto será retornada e ele voltará para a tela na parte inferior da tabela. Com essa espécie de rotação de objetos durante o scroll da nossa tabela conseguimos otimizar memória e ganhar performance.

Ao rodar a aplicação agora teremos algo ainda estranho.

<p align="center">
<img alt="Imagem com o exemplo o estado da célula atualmente. Ela aparece com o text label padrão de qualquer UITableViewCell, além dos componentes customizados com os valores fixos informados no storyboard" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-lt2-imitation-celula-desconfigurada.png?raw=true" width="25%"/>
</p>

A célula apresentada já mostra alguns dos componentes customizados, mas ainda com os dados fixos do storyboard, além do componente `textLabel` padrão de qualquer `UITableViewCell`. Isso aconteceu porque, com a indicação do _reuse identifier_, já foi possível inferir para o desenho dos componentes no storyboard, ainda que usando implementação genérica. Já iremos corrigir esse problema.

Nossa table view cell existe apenas como representação visual no storyboard e, sendo assim, não existe uma forma de referenciar uma instância da mesma (na verdade ainda não existem meios para sequer construir uma). Precisamos de uma representação (classe) para a mesma enquanto TableViewCell.

* No _Project navigator_, clique sobre a referência à pasta `Scenes/Conta/View`.
* Clique em _File > New > File..._ no menu do Xcode ou use o atalho _⌘ + N_.
* Selecione a opção _Cocoa Touch Class_.
* Na próxima janela, selecione _UITableViewCell_ para o seletor _Subclass of:_.
* Digite como nome da classe "TransacaoTableViewCell".
* Clique em _Next_ e em seguida _Create_. (Esta segunda janela deve informar como local para o arquivo a pasta de View contextualizada para a tela de Conta).
* Apague os _snippets_ para o métodos que foram autogerados. Eles não serão necessários.

Agora que temos uma classe para a representação da célula customizada, precisamos referenciar o novo arquivo no storyboard como implementação de base para a célula da table view.

* Abra novamente o arquivo `Main.storyboard`.
* Selecione a TransacaoCell no _Document Outline_.
* Através do _Identity Inspector_, selecione `TransacaoTableViewCell` para o seletor _Class_ da seção _Custom Class_ e pressione Enter.
* Abra o arquivo `ContaViewController` novamente, e altere a implementação do método `tableView(_:cellForRowAt:) -> UITableViewCell` para conter o código abaixo.

``` swift
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(withIdentifier: "TransacaoCell", for: indexPath) as? TransacaoTableViewCell else {
            fatalError("Não foi possível obter célula para a tabela.")
        }
        
        return cell
    }
```

Agora temos uma referência do tipo adequado para a célula, mas ainda apresentamos nossa célula apenas com os dados fixos vindos do storyboard.

Da forma como se encontra a implementação da nossa Table View Cell não temos nenhum comportamento efetivamente diferente da implementação de base. Precisamos adicionar código em `TransacaoTableViewCell` para obter as referências (_IB Outlets_) dos componentes de imagem e labels, e para que seja possível orientar quais são os dados e como eles serão apresentados em cada célula.

* Abra o arquivo `TransacaoTableViewCell` e adicione código para chegar ao estado abaixo.

``` swift
class TransacaoTableViewCell: UITableViewCell {
    
    @IBOutlet weak var transacaoImageView: TransacaoImageView!
    @IBOutlet weak var tituloLabel: UILabel!
    @IBOutlet weak var dataLabel: UILabel!
    @IBOutlet weak var interessadoLabel: UILabel!
    @IBOutlet weak var valorTransacaoLabel: UILabel!
    @IBOutlet weak var subtipoLabel: UILabel!
    
}
```

* Abra um editor de arquivos auxiliar à direita, e através dele, acesse o arquivo `Main.storyboard`.
* Conecte adequadamente os _outlets_ para cada elemento da célula desenhada.

<p align="center">
<img alt="Animação com demonstração do passo anterior de ligacao de outlets da table view cell para o desenho do storyboard" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-lt2-imitation-conectando-outlets-para-celula.gif?raw=true" width="80%"/>
</p>

* Com as conexões estabelecidas, feche o editor secundário.
* No `TransacaoTableViewCell`, adicione a implementação abaixo para prover uma forma de configurar os dados para a célula programaticamente.

``` swift
class TransacaoTableViewCell: UITableViewCell {
    
    // propriedades armazenadas com outlets omitidos
    
    func setup() {
        tituloLabel.text = "Transferência enviada"
        dataLabel.text = "12 SET."
        valorTransacaoLabel.text = "R$ 5.000,00"
        
        // se necessário ao tipo
        interessadoLabel.text = "Rafael Rollo"
        interessadoLabel.isHidden = false
        
        // se necessário ao tipo
        subtipoLabel.text = "Pix"
        subtipoLabel.isHidden = false
    }
    
    override func prepareForReuse() {
        interessadoLabel.text = nil
        interessadoLabel.isHidden = true
        
        subtipoLabel.text = nil
        subtipoLabel.isHidden = true
    }
    
}
```

* Abra o arquivo `ContaViewController` novamente, e altere a implementação do método `tableView(_:cellForRowAt:) -> UITableViewCell` para conter o código abaixo.

``` swift
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(withIdentifier: "TransacaoCell", for: indexPath) as? TransacaoTableViewCell else {
            fatalError("Não foi possível obter célula para a tabela.")
        }
        
        cell.setup()
        return cell
    }
```

Demos mais um passo em direção ao objetivo. Ainda temos dados fixos e não conseguimos ver os ícones para as transações, mas para resolver ambos os problemas precisamos primeiro de representações do modelo. 

> Nota: Você reparou na utilização do método `prepareForReuse()` na implementação de `TransacaoTableViewCell`? Sobrescrever este método pode ser útil em situações onde, ao reutilizar uma célula, é necessário fazer alterações que vão além de apenas o conteúdo textual de elementos. Lembre que vamos alternar a visibilidade de alguns dos labels por tipo de célula. Sem este método podemos ter problemas ao apresentar os dados a partir do reuso. 

### Implementando o modelo

Até este momento do desenvolvimento focamos apenas na construção da view e de sua integração com o controlador. Por este motivo, comportamentos fundamentais para o aplicativo, como exibir o ícone para a transação, ainda não são possíveis. Se você abrir o código do componente de view customizado `TransacaoImageView` vai perceber que sua implementação (atualmente comentada) faz referências à tipos ainda inexistentes do projeto, como o modelo de tipo de transação.

Neste tópico vamos tratar de resolver estes problemas. Começando por modelar a ideia de uma transação.

#### Modelando a Transação

* Clique com o botão direito do mouse na pasta base do projeto no _Project navigator_.
* Em seguida selecione _New Group_ para criar um grupo com pasta, e dê a ele o nome de _Models_.
* Arraste esta pasta abaixo para próximo de `View/`
* Clique com o botão direito do mouse na pasta `Models/` no _Project navigator_.
* Em seguida selecione _New File_ e selecione a opção _Swift File_.
* Dê ao novo arquivo o nome de `Transacao.swift`.
* No novo arquivo swift, adicione a implementação abaixo.

``` swift
struct Transacao {
    let data: Date
    let tipo: String
    let interessado: String?
    let valor: Decimal
    
    init(tipo: String, valor: Decimal, data: Date = .now, interessado: String? = nil) {
        self.data = data
        self.tipo = tipo
        self.interessado = interessado
        self.valor = valor
    }
}
```

Perceba como já temos os elementos principais de uma transação, além de um inicializador customizado que já permite que sejam criadas transações de formas alternativas, seja não obrigando a passagem de uma data (e então é associado `.now`), seja possibilitando a criação sem a informação de interessado, que, como o domínio do problema nos mostra, é opcional.

Uma questão importante nessa versão do modelo é como tratamos o tipo da transação. O `tipo` está definido como string, possibilitando um range muito diverso de valores a serem associados. Recorrendo novamente às ideias do domínio da aplicação, recordamos que o tipo de transação pode assumir apenas um dos valores do conjunto formato por `Transferência enviada`, `Transferência recebida`, `Pagamento efetuado`, `Pagamento da fatura`, `Recarga de celular` ou `Compra no débito`, e que além disso, alguns tipos podem ter subtipos que também são determinados.

Para modelar o domínio com precisão, vamos recorrer ao uso de enumerações. As enums vão nos dar exatamente o poder de limitar os possíveis valores string, e também de exigir os subtipos adequados a partir do uso da feature de _associated types_.

* Altere a implementação presente em `Transacao.swift` para ter o código abaixo.

``` swift
struct Transacao {
    let data: Date
    let tipo: TipoDeTransacao // enum
    let interessado: String?
    let valor: Decimal
    
    init(tipo: TipoDeTransacao, valor: Decimal, data: Date = .now, interessado: String? = nil) {
        self.data = data
        self.tipo = tipo
        self.interessado = interessado
        self.valor = valor
    }
}

enum TipoDeTransacao {
    case transferenciaEnviada(Transferencia)
    case transferenciaRecebida(Transferencia)
    case pagamentoEfetuado(Pagamento)
    case pagamentoDaFatura
    case recargaDeCelular
    case compraNoDebito
}

enum Transferencia: String {
    case doc, ted, pix
}

enum Pagamento: String {
    case boleto, pix
}
```

Com a alteração já conseguimos exercer controle sobre o universo de possibilidades para um `TipoDeTransacao`. Algo a se notar é que, com o uso de tipos associados à enum `TipoDeTransacao`, infelizmente não é possível defini-la como tendo o _raw type_ `String`. Sendo assim, precisamos adicionar propriedades para computar os valores, como por exemplo para a string de título ao invés de usar `.rawValue`.

* Adicione as propriedades armazenadas abaixo à enum `TipoDeTransacao`.

``` swift
enum TipoDeTransacao {
    // cases omitidos

    var titulo: String {
        switch self {
        case .transferenciaEnviada:
            return "Transferência enviada"
        case .transferenciaRecebida:
            return "Transferência recebida"
        case .pagamentoEfetuado:
            return "Pagamento efetuado"
        case .pagamentoDaFatura:
            return "Pagamento da fatura"
        case .recargaDeCelular:
            return "Recarga de celular"
        case .compraNoDebito:
            return "Compra no débito"
        }
    }
    
    var subtipo: String? {
        switch self {
        case .transferenciaEnviada(let tipoDeTransferencia),
                .transferenciaRecebida(let tipoDeTransferencia):
            return tipoDeTransferencia.rawValue.capitalized
        case .pagamentoEfetuado(let tipoDePagamento):
            return tipoDePagamento.rawValue.capitalized
        default:
            return nil
        }
    }
}
```

Perceba que já conseguimos adicionar a representação para o subtipo, considerando-o como opcional em relação ao tipo da transação.

Com a implementação atual, já é possível contar com o suporte da view customizada `TransacaoImageView`.

* Abra o arquivo `TransacaoImageView.swift`, selecione todo o conteúdo comentado da classe e pressione o comando _⌘ command + /_.
* Builde a aplicação novamente utilizando _⌘ command + B_. _(A aplicação deve ser compilada com sucesso)_.

> Nota: Neste momento, já sendo possível para o Interface Builder determinar detalhes sobre o componente que encapsula as regras de visualização para a imagem da transação, caso você abra novamente o storyboard, depois de recarregar o arquivo, provavelmente a prévia no canvas será apresentada de forma mais fiel ao que esperamos para o design.

Nesse momento, ja temos o necessário para configurar células com mais fidelidade ao resultado final.

* Altere a implementação de `TransacaoTableViewCell` para que o método `setup()` receba uma referência de `Transacao` e oriente sua apresentação a partir dos dados da instância.

``` swift
class TransacaoViewCell: UITableViewCell {
    
    // outlets omitidos
    
    func setup(_ transacao: Transacao) {
        transacaoImageView.tipoDeTransacao = transacao.tipo
        tituloLabel.text = transacao.tipo.titulo
        dataLabel.text = transacao.data.description
        valorTransacaoLabel.text = String(describing: transacao.valor)
        
        if let interessado = transacao.interessado {
            interessadoLabel.text = interessado
            interessadoLabel.isHidden = false
        }
        
        if let subtipo = transacao.tipo.subtipo {
            subtipoLabel.text = subtipo
            subtipoLabel.isHidden = false
        }
    }
    
    // prepareForReuse() omitida
}
```

* Agora, para corrigir o problema de compilação de `ContaViewController`, altere a implementação do método `tableView(_:cellForRowAt:) -> UITableViewCell` para conter o código abaixo.

``` swift
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(withIdentifier: "TransacaoCell", for: indexPath) as? TransacaoTableViewCell else {
            fatalError("Não foi possível obter célula para a tabela.")
        }
        
        let transacao = Transacao(tipo: .transferenciaRecebida(.pix), valor: 200, interessado: "Alberto Souza")
        
        cell.setup(transacao)
        return cell
    }
```

Nesse momento, rodando a aplicação temos a seguinte exibição.

<p align="center">
<img alt="Imagem com o exemplo o estado da célula atualmente. Ela aparece já com o layout pronto e ícone visível, porém ainda sem formatações" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-lt2-imitation-celula-sem-formatacao-de-valores.png?raw=true" width="25%"/>
</p>

Estamos quase prontos em relação à exibição dos itens 😃. O único detalhe ainda é que não temos valores formatados adequadamente para a data e o valor. Por este motivo, no caso da data, por exemplo, seu label que tem maior aderência a permanecer com seu tamanho intrínseco cresce para ocupar a maior parte da tela por conta das descrição como string estar extensa demais. Sabemos que nosso layout prevê o uso de um valor para data abreviado. Precisamos cuidar disso.

#### Usando extensions para adicionar implementações úteis aos tipos

Para resolver o problema de formatação podemos usar a implementação de DateFormatter e NumberFormatter para os problemas de data e valor como moeda, respectivamente. No entanto, ao invés de deixar as implementações contextualizadas próximas ao uso (dentro do componente de view, por exemplo), ou mesmo criar classes para conter código útil e prover reuso (como já fizemos antes neste treino) uma estratégia mais interessante é o uso da feature de _Extensions_ da Swift.

> Nota: Perceba que o projeto já utiliza essa estratégia para prover valores com mais fácil acesso na API pública de UIColors.

Com ela vamos adicionar mais detalhes de implementação nos tipos base para prover os valores corretamente formatados para os padrões esperados pelo projeto. É possível adicionar implementação tanto nos componentes que modelam o próprio dado, `Date` e `Decimal`, como nos componentes de formatadores, `DateFormatter` e `NumberFormatter`. Neste guia trabalharemos com os dois para aumentar nosso repertório, mas você pode optar pelo que entender mais agradavél.

* Clique com o botão direito do mouse na pasta `Common/Extensions` no _Project navigator_.
* Em seguida selecione _New File_ e selecione a opção _Swift File_.
* Dê ao novo arquivo o nome de `CustomPatterns+DateFormatter.swift`.
* No novo arquivo swift, adicione a implementação abaixo.

``` swift
extension DateFormatter {
    
    private static var dayPlusAbbreviatedMonthFormatter: DateFormatter = {
        let formatter = DateFormatter()
        formatter.locale = .init(identifier: "pt_BR")
        formatter.dateFormat = "dd MMM"
        
        return formatter
    }()
    
    enum CustomPattern {
        case dayPlusAbbreviatedMonth
        
        var formatter: DateFormatter {
            switch self {
            case .dayPlusAbbreviatedMonth:
                return DateFormatter.dayPlusAbbreviatedMonthFormatter
            }
        }
    }
    
    static func format(date: Date, to customPattern: CustomPattern) -> String {
        return customPattern.formatter.string(from: date).uppercased()
    }
    
}

extension Date {
    typealias FormattedText = String
    
    var asDayPlusAbbreviatedMonth: FormattedText {
        return DateFormatter.format(date: self, to: .dayPlusAbbreviatedMonth)
    }
}

```

Neste momento as APIs públicas de `Date` e `DateFormatter` oferecem as implementações para favorecer o reuso por todo o projeto. Você poderia optar por utilizar `transacao.data.asDayPlusAbbreviatedMonth` para tornar o código mais conciso, ou `DateFormatter.format(date: transacao.data, to: .dayPlusAbbreviatedMonth)` para expressar mais intenção. Por aqui vamos com a segunda.

* Altere a implementação de `TransacaoTableViewCell` para atribuir o valor da string devidamente formatada ao texto do label de data.

``` swift
    // código anterior omitido

    func setup(_ transacao: Transacao) {
        transacaoImageView.tipoDeTransacao = transacao.tipo
        tituloLabel.text = transacao.tipo.titulo
        dataLabel.text = DateFormatter.format(date: transacao.data, to: .dayPlusAbbreviatedMonth)
        
        // demais atribuições
    }

    // código posterior omitido
```

Com a data formatada, passamos ao valor decimal.

* Clique com o botão direito do mouse na pasta `Common/Extensions` no _Project navigator_.
* Em seguida selecione _New File_ e selecione a opção _Swift File_.
* Dê ao novo arquivo o nome de `Currency+NumberFormatter.swift`.
* No novo arquivo swift, adicione a implementação abaixo.

``` swift
extension NumberFormatter {
    
    private static var currencyFormatter: NumberFormatter = {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currency
        formatter.locale = Locale(identifier: "pt_BR")
        return formatter
    }()
    
    static func formatToCurrency(decimal: Decimal) -> String {
        return currencyFormatter.string(from: decimal as NSDecimalNumber)!
    }
    
}

extension Decimal {
    typealias FormattedText = String
    
    var asCurrency: FormattedText {
        return NumberFormatter.formatToCurrency(decimal: self)
    }
}
```

* Agora, no `TransacaoTableViewCell`, altere a implementação para usar o valor decimal formatado.

``` swift
    func setup(_ transacao: Transacao) {
        transacaoImageView.tipoDeTransacao = transacao.tipo
        tituloLabel.text = transacao.tipo.titulo
        dataLabel.text = DateFormatter.format(date: transacao.data, to: .dayPlusAbbreviatedMonth)
        valorTransacaoLabel.text = NumberFormatter.formatToCurrency(decimal: transacao.valor)
        
        // demais atribuições
    }
```

Com isso temos tudo pronto para a exibição dos itens nas células. 🎉

#### Manipulando a lista de Transação através da Conta

Se você rodar agora a aplicação e comparar o resultado obtido com o objetivo da tarefa, claramente conseguirá perceber muitas diferenças. Nossos próximos passos neste tópico já vão reduzir um pouco essa sensação.

A tela que estamos implementando visa apresentar toda a lista de transações recentes de um Conta digital. Adicionalmente, vale mencionar que o ponto desta tela em um fluxo de utilização do app como um todo provavelmente está a frente de alguma tela inicial. Precisamos considerar a ideia de uma seleção prévia da conta que se deseja visualizar, e para isso, precisamos modelar de fato a ideia de um Conta que tem Transações. Vamos a isso!

* Clique com o botão direito do mouse na pasta `Models` no _Project navigator_.
* Em seguida selecione _New File_ e selecione a opção _Swift File_.
* Dê ao novo arquivo o nome de `Conta`.
* No novo arquivo swift, adicione a implementação abaixo.

``` swift
struct Conta {
    let saldo: Decimal
    let montanteInvestido: Decimal
    let rendimentoMensalAtual: Decimal
    var historico: [Transacao]
    
    init(saldoInicial: Decimal, montanteInvestido: Decimal,
         rendimentoMensalAtual: Decimal, historico: [Transacao] = []) {
        self.saldo = saldoInicial
        self.montanteInvestido = montanteInvestido
        self.rendimentoMensalAtual = rendimentoMensalAtual
        self.historico = historico
    }
}
```

Temos uma _struct_ simples contemplando as características da `Conta` e contando com um inicializador customizado que possibilita a criação de uma instância ainda sem uma lista de transações associada. Isso pode ajudar se pensarmos que se deseja carregar as informações de uma Conta em um momento inicial, e somente ao ir para a nossa tela, carregar enfim a lista de transações que pode ser extensa.

Com nossa representação pronta. Podemos então trabalhar para fornecer um modelo de Conta para nosso controlador. A partir dessa referência é que ele irá acessar a instância de conta para pedir suas transações à API e posteriormente atualizar a lista ao receber a resposta.

* Volte ao arquivo `ContaViewController` e altere sua implementação para conter referências para `Conta` e `ContasAPI`.

``` swift
class ContaViewController: UITableViewController {
    
    var contasAPI: ContasAPI?
    var conta: Conta! 
    
    // override func viewDidLoad() { ... }
    
    // demais métodos omitidos
}
```

* Agora, abra o arquivo `SceneDelegate` e descomente as linhas de código próximas ao final do método `scene(_:willConnectTo:options)`. A implementação deve ficar como a que segue.

``` swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        // comentátios gerados pelo template de projeto Xcode.
        guard let _ = (scene as? UIWindowScene) else { return }
        
        let contasAPI = ContasAPI()
        let contaSelecionada = Conta(saldoInicial: 99_999.90, montanteInvestido: 19_999.90, rendimentoMensalAtual: 199.90)

        let controller = (window!.rootViewController as! UINavigationController).topViewController as! ContaViewController
        controller.contasAPI = contasAPI
        controller.conta = contaSelecionada
    }

    // demais métodos omitidos
}
```

> Nota: Aqui novamente utilizamos o `SceneDelegate` como ponto de acesso aos módulos anteriores ao controlador na hierarquia de objetos. Em uma situação de aplicação real provavelmente teríamos essas injeções de dependências feitas de formas diferentes.

Agora que já temos os objetos no contexto do controlador, podemos utilizá-los para obter a lista de transações junto à `ContasAPI` e orientar a construção da tabelas através destes dados, alterando a implementação do _data source_.

* Abra o arquivo `ContasAPI` da pasta `API/`
* Selecione as linhas de código comentadas de sua implementação e, através de _⌘ command + /_, torne-as válidas novamente.
* Agora, altere a implementação do `ContaViewController` para que tenhamos a lista de transações da conta apresentada ao invés dos dados de exemplo. A implementação deve ficar próxima ao código abaixo.

``` swift
class ContaViewController: UITableViewController {
    
    var contasAPI: ContasAPI?
    
    var conta: Conta! {
        didSet { // observa alterações no estado da conta para atualizar a view
            tableView.reloadData()
        }
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        if let contasAPI = contasAPI {
            // carrega histórico de transações para a conta
            conta.historico = contasAPI.carregaHistorico(para: conta)
        }
    }
    
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        // define o número de linhas para a seção através da quantidade de elementos do array
        return conta.historico.count
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(withIdentifier: "TransacaoCell", for: indexPath) as? TransacaoTableViewCell else {
            fatalError("Não foi possível obter célula para a tabela.")
        }
        
        // recupera a transação correta para o determinado index path
        let transacao = conta.historico[indexPath.row]
                
        cell.setup(transacao)
        return cell
    }
    
}
```

A partir desta implementação já temos nossa nossa table view apresentando todos os valores corretamente. Faltam apenas alguns ajustes para finalizar a tarefa e atingir a especificação da tarefa. 🥳

<p align="center">
<img alt="Animação com a demo scrollando por toda a tabela e mostrando a lista completa na tela" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-lt2-imitation-table-view-com-todos-os-itens.gif?raw=true" width="25%"/>
</p>

### Fazendo os ajustes finais na implementação

Agora vamos atacar os dois últimos pontos para atingir o objetivo. Primeiro vamos prover o componente de view para o _header_ da tabela como um todo, e depois vamos prover os elementos que serão utilizados para os _headers_ de seção.

#### Fornecendo uma implementação para `tableViewHeader`.

Seja pelo enunciado desta tarefa ou mesmo pelo primeiro _tour_ pela organização do projeto, você provavelmente percebeu a presença do componente `TableHeaderView`. A implementação do mesmo permaneceu comentada até o momento pra evitar problemas com o _build_, já que ela dependia de componentes da camada do modelo. 

Os detalhes de implementação do componente não são importantes para o escopo da tarefa uma vez que estudaremos aqui o que tange à configuração de um header para a tabela. Poderíamos trabalhar essencialmente com qualquer objeto de UIView e a dinâmica deste tópico seria a mesma. A API pública do componente oferece um _factory method_ `constroi(para:) -> TableHeaderView` que recebe uma `Conta` e devolve o componente devidamente configurado.

Um único detalhe que é interessante notar é que Table View não provê total suporte à Auto Layout para seus componentes de _header view_ e _footer view_ como poderíamos esperar. Então se torna fundamental definir o `frame` para as views que passamos adiante (em alguns casos como nos headers de seção, precisaremos alternativamente informar para a Table View sobre as dimensões de altura destes componentes). Internamente no código dos componentes você pode utilizar os recursos do Auto Layout normalmente. Vamos ao código.

* Antes de qualquer coisa, abra o arquivo `TableHeaderView` e descomente sua implementação através de _⌘ command + /_.
* Altere a implementação do `ContaViewController` para adicionar código que configura o header para a tabela. O código deve se parecer com o que segue.

``` swift
class ContaViewController: UITableViewController {
    
    // propriedades armazenadas omitidas

    override func viewDidLoad() {
        super.viewDidLoad()
        
        // adiciona o header
        tableView.tableHeaderView = TableHeaderView.constroi(para: conta)

        if let contasAPI = contasAPI {
            // carrega histórico de transações para a conta
            conta.historico = contasAPI.carregaHistorico(para: conta)
        }
    }

    // código posterior omitido
```

Como ainda vamos adicionar código responsável por configurar o visual para a tabela, e assim como o código relativo ao carregamento das transações poderiam oferecer maior complexidade em aplicações reais, vamos separar essas duas principais responsabilidades contidas hoje no método `viewDidLoad()`.

* Refatore o código do controlador para contar com as funções `setupViews()` e `carregaHistorico()`.

``` swift
class ContaViewController: UITableViewController {
    
    // propriedades armazenadas omitidas

    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupViews()
        carregaHistorico()
    }

    func setupViews() {
        tableView.tableHeaderView = TableHeaderView.constroi(para: conta)
    }

    func carregaHistorico() {
        guard let contasAPI = contasAPI else { return } 
        conta.historico = contasAPI.carregaHistorico(para: conta)
    }

    // código posterior omitido
```

Com esse código, nossa tabela adiciona a _view_ na seção logo acima do conjunto de valores apresentados no conteúdo _scrollavel_ da lista. Avançamos ainda mais em direção ao objetivo pra essa tela.

#### Fornecendo uma implementação para _headers_ de seção

Nesse tópico, focaremos em algo ainda pendente na visualização da nossa coleção de transações para a Conta. Embora para o escopo da atividade tenhamos uma lista única sem a ideia de separação de seções, o objetivo prevê um título para a única seção. Veremos nos próximos passos uma das possíveis implementações e como o _Table View_ suporta esse tipo de customização.

##### Implementando o código para a view do header de seção

Atualmente, não temos no projeto um componente de view para utilizarmos nos headers de seção, como tivemos para o _table header_. Vamos então implementá-lo.

Até então nossa experiência de desenvolvimento de visualizações foi totalmente apoiado pelo Interface Builder. Através dele, desenhamos nossa composição de componentes por meio de referências no canvas, que por sua vez são conectadas com o código em um segundo momento. Como em qualquer cenário, essa abordagem tem alguns pontos positivos, mas também uma série de pontos negativos. A discussão conceitual sobre qual modelo para visualização utilizar está fora do escopo deste treino, mas desde já podemos conhecer as possibilidades de implementação. Dessa forma já temos uma boa ideia dos estilos de desenvolvimento utilizados e aumentamos repertório.

Nossa view de cabeçalho para seção é bem simples. Ela é composta apenas por um _subview_ do tipo `UILabel`, com algum detalhe específico para o estilo do texto e para o posicionamento do mesmo dentro do box da _view_. Vamos então escrever nosso código.

* Clique com o botão direito do mouse na pasta `View/` no _Project navigator_.
* Em seguida selecione _New Group_ para criar um grupo com pasta, e dê a ele o nome de _Components_.
* Clique com o botão direito do mouse na pasta `Components/` no _Project navigator_.
* Em seguida selecione _New File_ e selecione a opção _Cocoa Touch Class_.
* Na próxima janela, selecione _UITableViewHeaderFooterView_ para o seletor _Subclass of:_.
* Digite como nome da classe "SectionTitle".
* Avance para criar o novo arquivo.
* No novo arquivo swift, adicione a implementação abaixo.

``` swift
class SectionTitle: UITableViewHeaderFooterView {
    
    static var reuseId: String {
        return String(describing: self)
    }
    
    static var heightConstant: CGFloat = 72

    override init(reuseIdentifier: String?) {
        super.init(reuseIdentifier: reuseIdentifier)
        setup()
    }
        
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setup() {
        // código de configuração da view vai aqui
    }

}
```

Fizemos algumas coisas nesse trecho que valem a pena destacar. Primeiro construímos nosso novo arquivo de _view_ dentro de `View/Components/`. Isso porque, como nosso componente é simples o suficiente para poder ser reutilizado no contexto de outras tabelas ou mesmo telas, faz mais sentido viver longe do contexto da tela de Contas.

Depois, construímos nossa classe herdando de `UITableViewHeaderFooterView`. Isso é necessário porque, mesmo com a API de _Table View_ esperando apenas uma `UIView`, como veremos em métodos adiante, queremos contar também com o recurso de [reuso de objetos de _header_ e _footer_](https://developer.apple.com/documentation/uikit/uitableview/1614975-dequeuereusableheaderfooterview) view assim como fizemos para as células. 

Indo adiante, sobre o código, percebemos duas propriedades de tipo para facilitar o acesso ao `reuseIdentifier` ao registrar o componente de `HeaderFooterView` e também a uma constante de altura para informar esta dimensão ao objeto de table view (como discutido no tópico anterior). Além disso, provemos uma maneira de inicializar o componente adequadamente aproveitando a implementação da superclasse. Nesse ponto, atente-se para a obrigatoriedade da presença do `required init?(coder: NSCoder)`, que nesse caso apenas encerra a aplicação já que não prevemos suporte a este componente através do uso pelo Interface Builder.

Para continuar nosso código, precisamos de uma maneira de configurar o conteúdo interno da _view_. Algo parecido com arrastar um novo componente de _label_ para dentro do box da _view_ e configurar suas constraints, mas através de código Swift.

* Adicione implementação ao código de `SectionTitle` para que o código contenha um _label_ devidamente configurado.

``` swift
class SectionTitle: UITableViewHeaderFooterView {
    
    static var reuseId: String {
        return String(describing: self)
    }
    
    static var heightConstant: CGFloat = 72
    
    private lazy var label: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.textColor = .label
        label.font = .systemFont(ofSize: 18, weight: .semibold)
        return label
    }()
    
    override init(reuseIdentifier: String?) {
        super.init(reuseIdentifier: reuseIdentifier)
        setup()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setup() {
        addViews()
        addContraints()
    }
    
    private func addViews() {
        addSubview(label)
    }
    
    private func addContraints() {
        NSLayoutConstraint.activate([
            heightAnchor.constraint(equalToConstant: Self.heightConstant)
        ])
        
        NSLayoutConstraint.activate([
            label.leadingAnchor.constraint(equalTo: self.leadingAnchor, constant: 24),
            label.trailingAnchor.constraint(equalTo: self.trailingAnchor, constant: 24),
            label.bottomAnchor.constraint(equalTo: self.bottomAnchor, constant: -24),
        ])
    }

}
```

Com essa adição, temos uma propriedade que guarda a referência para o label, definida com um bloco de código autoexecutável que configura o label, mas que, pelo uso do modificador `lazy`, só vai ser inicializada no momento onde o label realmente precisar ser carregado para a memória. No mais, a implementação apenas adiciona funções para adicionar as subviews na árvore hierárquica de visualizações e, a partir daí, configurar as constraints para o layout através da API de `NSLayoutConstraint` (uso do Auto Layout normalmente, como faríamos visualmente no Interface Builder).

Por fim, apenas para oferecer meios de configurar o título, vamos adicionar mais uma propriedade na API pública do componente.

``` swift
    // código anterior omitido
    
    // override init(reuseIdentifier: String?) { .. }

    var titulo: String? {
        didSet {
            label.text = titulo
        }
    }
```

Com o observer, já conseguimos assegurar a configuração do valor do _label_ interno assim que o titulo for definido externamente.

##### Orientando a tabela sobre os headers de seção.

Como podemos conter múltiplas seções para a tabela, a configuração dos objetos de _header_ e _footer_ _view_ necessitam ser diferentes do caso de _header_ e _footer_ geral para a _Table View_. Precisamos prover implementação para que a tabela consulte a cada momento onde precisar obter um header para uma seção. Fazemos isso sobrescrevendo um método, dessa vez de `UITableViewDelegate`.

* Adicione implementação para o método `tableView(_:viewForHeaderInSection:) -> UIView?` no `ContaViewController`.

``` swift
    // código anterior omitido

    override func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
        guard let headerView = tableView.dequeueReusableHeaderFooterView(withIdentifier: SectionTitle.reuseId) as? SectionTitle else {
            fatalError("Não foi possível recuperar header view para a tabela")
        }
        
        headerView.titulo = "Histórico"
        return headerView
    }

}
```

Como o contexto da tela exige uma única seção atualmente, podemos inferir seu valor e passar um título fixo para a mesma. Em diferentes situações poderíamos trabalhar com o parâmetro que identifica a seção, `section`, para orientar o valor adequado.

Ainda não estamos prontos para os _headers_ de seção. Se rodarmos a aplicação no momento teremos um `crash`: `LearningTask_6_2/ContaViewController.swift:53: Fatal error: Não foi possível recuperar header view para a tabela`. É impossível obter uma view para o header através da queue da _Table View_ sem antes registrar o `reuseIdentifier` pelos quais serão selecionados.

* No código do `ContaViewController`, adicione implementação ao método `setupViews()` para registrar o identificador.

``` swift
    // código anterior omitido

    func setupViews() {
        tableView.tableHeaderView = TableHeaderView.constroi(para: conta)
        
        tableView.register(SectionTitle.self, forHeaderFooterViewReuseIdentifier: SectionTitle.reuseId)
        tableView.sectionHeaderHeight = SectionTitle.heightConstant
    }

    // código posterior omitido
```

Aproveite aqui para informar também sobre a dimensão de altura do componente de _header_ para a seção. Rodando a aplicação agora já temos 99% do objetivo cumprido.

<p align="center">
<img alt="Imagem com o visual da tela pronta, com a tabela com header geral, header de seção e corpo" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-lt2-imitation-table-view-com-visual-pronto.png?raw=true" width="25%"/>
</p>

#### Adicionando algum feedback visual ao selecionar um elemento da tabela

O 1% restante fica por conta da adição de alguma resposta à seleção dos elementos, se você se recordar dos requisitos para a tarefa. E isso se dá de maneira bastante simples, já que a implementação de fato de alguma regra de negócio em resposta está fora do escopo desta atividade. Mesmo assim, já conseguimos entender como interagir com os eventos do usuário para com a tabela. Precisamos novamente interagir com o protocolo `UITableViewDelegate`, dessa vez fornecendo implementação para o método `tableView(_:didSelectRowAt:)`. Atualmente, quando um elemento é selecionado, a célula exibe um estado selecionado para o elemento, alterando seu background.

* Adicione implementação ao `ContaViewController`, para conter o código abaixo.

``` swift
    // código anterior omitido

    override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
    }

} // fim do código do controlador
```

Aqui contamos apenas com o feedback de desseleção animado padrão da própria _Table View_.

Rode a aplicação e teste nosso comportamento. Temos nosso objetivo cumprido. 🥹

## Conclusão

Fechamos por aqui nossa implementação de um TableViewController com uma série de elementos para customização da UI para o seu aplicativo. Espero que tenha absorvido as ideias e os fundamentos por trás do caminho cognitivo apresentado para a solução.