# Siga os passos para desenvolver a tela de despesas de acordo com o padrão arquitetural

Neste passo a passo construiremos juntos a funcionalidade da tela de adição de despesas em um pseudo-aplicativo de solicitação de reembolsos. Ao final da atividade o aplicativo deve se parecer com a especificação abaixo.

<p align="center">
<img alt="Imagem da especificação de tela" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-lt2-especificacao-alvo.jpg?raw=true" width="80%"/>
</p>

## Finalizando a implementação da _view_

Para que possamos fazer nossas primeiras alterações, vamos primeiro conhecer o estado atual da _view_ da atividade. Abra o arquivo `Main.storyboard` que vive dentro do grupo `View/` do projeto base. Você deve encontrar seu _storyboard_ de acordo com a imagem do Interface Builder abaixo.

<p align="center">
<img alt="Imagem do Interface Builder no estado inicial do projeto" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-lt2-imitation-despesas-view-inicial.png?raw=true" width="80%"/>
</p>

Navegue pelos itens através do _Document Outline_ e perceba que já temos algumas _views_ adicionadas à hierarquia de _views_ do ViewController. Entre elas já temos inclusive a _stack view_ principal que coordena o arranjo vertical e a organização dos grandes blocos de visualização para a tela. Dentro dela já temos uma _stack view_ horizontal contendo os componentes para o cabeçalho, uma _view_ que por hora apenas se expande para manter o espaçamento entre os elementos, e finalmente um botão que pretende registrar o relatório de despesas.

### A seção _Novas entradas_

Agora que já temos uma ideia do ponto onde estamos, vamos começar a implementar a porção de tela com o formulário de novas despesas logo abaixo do cabeçalho.

Consulte a imagem com a especificação da tela e note que nesta área temos alguns padrões bem definidos de posicionamento e espaçamento entre os elementos, tanto verticalmente, quanto horizontalmente, além de um padrão de largura para os campos de texto.

Identificados os padrões, utilizando nosso conhecimento prévio, vamos começar a adicionar os primeiros elementos. Primeiro vamos contar com uma _stack view_ vertical para a área do formulário.

* Clique no ícone _(+)_ superior direito do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"vert"_.
* Utilize o _drag and drop_ para arrastar o elemento de Vertical Stack View para o _Document Outline_, posicionando-o abaixo do `Cabecalho Stack View`.
* Altere o identificador no _Document Outline_ para `Nova Entrada Stack View`.
* Aproveite para alterar a propriedade _Spacing_, no _Attributes Inspector_, para `12` pontos.

Ignore por enquanto algum possível problema apontado pelo Interface Builder, estamos apenas começando nossas alterações e, neste momento, isso é perfeitamente aceitável. Com as próximas alterações já teremos um layout funcional.

Dentro do `Nova Entrada Stack View`, adicione um novo UILabel que servirá de subtítulo para essa seção da UI.

* Clique no ícone _(+)_ superior direito do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"uil"_.
* Utilize o _drag and drop_ para arrastar um elemento de Label para o Document Outline, tomando cuidado para garantir a hierarquia.
* Altere o identificador no _Document Outline_ para `Nova Entrada Label`.

Altere as propriedades visuais do _label_ através do _Attributes Inspector_.

* Altere o valor do label para _"Nova entrada"_.
* Defina o _Font style_ para _System Semibold_.
* Defina o tamanho da fonte para `18`.

Com o título já definido, vamos adiante para adicionar a primeira dupla de _label_ e campo de texto para o formulário.

* Clique no ícone _(+)_ superior direito do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"uil"_.
* Utilize o _drag and drop_ para arrastar um elemento de Label para o Document Outline, abaixo do `Nova Entrada Label`.
* Com o _label_ selecionado, no _Attributes Inspector_ altere sua fonte para `System 14.0`
* Novamente, clique no ícone _(+)_ superior direito do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"text"_.
* Utilize o _drag and drop_ para arrastar um elemento de Text Field para o Document Outline, abaixo do Label anterior.
* Selecione o campo de texto e altere sua fonte para `System 12.0`.
* Também altere o identificador para o _label_ e para o campo de texto para `Label` e `Text Field`.

Para atingir o arranjo adequado para os dois últimos elementos adicionados, vamos contar com uma _stack view_ horizontal.

* Pressionando a tecla _command ⌘_, selecione os dois últimos elementos adicionados.
* Clique no ícone inferior direito do Interface Builder referente à feature _Embed In_, e em seguida selecione a opção _Stack View_ da seção _Embed In View_.
* Selecione a nova _stack view_ que foi adicionada e altere sua propriedade _Axis_ para _Horizontal_ e sua propriedade _Spacing_ para `12`.

Com o arranjo horizontal devidamente organizado, vamos aproveitar para fazer cópias da _stack view_ no mesmo ponto da hierarquia para facilitar a configuração dos demais campos.

* Pressionando a tecla _option ⌥_, utilize o _drag and drop_ no Document Outline para arrastar abaixo uma cópia do Stack View.
* Repita o processo para mais um campo.

Assim, já tendo nossos elementos dispostos, vamos organizar seus valores.

* Selecione a primeira _stack view_ do grupo e altere seu identificador no _Document Outline_ para `Stack View Título`.
* Com o _label_ selecionado, no _Attributes Inspector_ altere seu valor para "Título:".
* Selecione o campo de texto e adicione "Título curto para o item" como valor para o _Placeholder_.
* Repita o procedimento para as outras duas _stack views_ com os valores `Stack View Tipo`, "Tipo:" e "1: Eletrônicos - 2: Escritório", além de `Stack View Valor`, "Valor:", "R$ 100,00", respectivamente.

Perceba agora, que com a diferença dos tamanhos de texto para os _labels_, perdemos a característica de largura compatível entre os campos de texto do form, requerida pela especificação de design. Para corrigir o problema e garantir a característica vamos adicionar _constraints_ do Auto Layout para igualar as larguras.

* Pressionando a tecla _command ⌘_, selecione os campos de texto.
* Clique no ícone inferior direito _Add New Constraints_.
* Marque o _check box_ que indica a _constraint_ Equal Widths.
* Clique no botão _Add 2 Constraints_.

Um último ponto muito importante ainda antes de avançar para a representação do botão é fazer os devidos ajustes nos campos de texto para informações numéricas. Da forma padrão como estamos, ao selecionar os campos de texto para tipo e valor, o usuário acessa o teclado virtual na sua forma padrão, o que pode aumentar o esforço necessário ao usuário para inserir valores adequados, assim como dificultar o processo de validação sendo possível a inserção de caracteres alfanuméricos e especiais. Para garantir uma melhor experiência, vamos então ajustar os teclados para esses campos.

* Selecione o campo de texto para o tipo de despesa, e no _Attributes Inspector_ selecione a opção _Number Pad_ para a propriedade _Keyboard Type_.
* Selecione o campo de texto para o valor da despesa, e para a mesma propriedade, selecione a opção _Decimal Pad_.

Por fim, podemos ir adiante para o botão que adiciona novas entradas no relatório. 

Perceba que o botão deve ser colocado no canto direito da tela, tendo sua largura apenas com base no seu tamanho intrínseco. Isso pode parecer confuso se pensarmos que a propriedade _Alignment_ do `Nova Entrada Stack View`, configurada como _Fill_, força as _arranged subviews_ a preencherem todo o espaço na direção (horizontal) contrária ao seu eixo (vertical) - você pode testar isso adicionando um novo Filled Button abaixo da última _stack_ horizontal nesse momento. 

Para conseguir alcançar o layout desejado mesmo com esse comportamento, que é esperado, podemos usar uma tática interessante. É possível envolver o botão com uma nova _stack view_ vertical, invertendo apenas nesse ponto específico da hierarquia o valor para a propriedade _Alignment_.

* Clique no ícone _(+)_ superior direito do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"uib"_.
* Utilize o _drag and drop_ para arrastar um elemento de Filled Button para o Document Outline, abaixo da `Stack View Valor`.
* Com o botão selecionado no _Document Outline_, altere seu identificador para `Botao Adicionar`, clique no ícone inferior direito referente à feature _Embed In_, e em seguida selecione a opção _Stack View_ da seção _Embed In View_.
* Para a nova _stack view_, altere seu identificador para `Stack View Submit` e altere também sua propriedade _Alignment_ para _Trailing_.

Relembrando dos conceitos de Auto Layout com _stack views_ não é difícil perceber o que ocorreu neste momento. A `Stack View Submit` se estende para preencher o espaço horizontal, contendo o efeito da característica de alinhamento da _stack view_ superior. Isso faz com que o botão não sofra _stretching_ na sua largura internamente e mantenha então seu tamanho intrínseco. Seu posicionamento se dá junto à âncora de _trailing_ de sua _superview_.

Fechando a seção basta definir as propriedades para o estilo do botão.

* Selecione o botão no _Document Outline_, em seguida navegue para o _Attributes Inspector_.
* Certifique-se de que a propriedade _Style_ esteja definida para _Filled_.
* Altere o valor textual do botão para "Adicionar".
* Clique no seletor de fonte e altere a propriedade _Font_ para _System_ e a propriedade _Size_ para `12`.
* Altere também a cor na propriedade _Background_ para _Downriver_.
* No seletor de imagem, digite "plus" e selecione o ícone de adição _(+)_.
* Logo abaixo, para a propriedade _Placement_, selecione a opção _Trailing_, e para a propriedade _Padding_, `4`.
* Para a propriedade _Corner Style_, você pode definir a opção _Capsule_.
* Ainda com o botão selecionado, navegue agora para o _Size Inspector_ e selecione a opção _Small_ para a propriedade _Size_.

Nesse momento devemos ter algo como mostra a imagem abaixo.

<p align="center">
<img alt="Imagem do Interface Builder com a seção do formulário construída" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-lt2-imitation-view-secao-formulario.png?raw=true" width="80%"/>
</p>

Não se preocupe se algo ainda parecer estranho em relação às alturas dos campos de texto nessa prévia, provavelmente seja apenas o Interface Builder confuso em relação aos cálculos no canvas. Você pode rodar sua aplicação nesse momento para se certificar de que tudo vai bem.

### A seção _Relatório parcial_

Com as _views_ que possibilitam a entrada de novas despesas já dispostas, precisamos agora construir a porção de tela onde apresentaremos as despesas já adicionadas ao relatório.

Consulte novamente a imagem com a especificação da tela e note que nesta outra área também temos alguns padrões bem definidos de posicionamento e espaçamento entre os elementos. Podemos contar novamente com as _stack views_ para tirar proveito da facilidade que elas oferecem.

* Clique no ícone _(+)_ superior direito do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"vert"_.
* Utilize o _drag and drop_ para arrastar um elemento de Vertical Stack View para o _Document Outline_, posicionando-o abaixo do `Nova Entrada Stack View`.
* Altere o identificador no _Document Outline_ para `Despesas Stack View`.
* Aproveite para alterar a propriedade _Spacing_, no _Attributes Inspector_, para `12` pontos.
* Nesse momento, certifique-se de remover aquela View em branco que veio junto com o projeto.

Possivelmente as ambiguidades entre os tamanhos dos componentes gerenciados estão dificultando o preenchimento da _stack view_ principal pelo seu eixo vertical. Então vamos fornecer o ajuste necessário para que a _stack view_ saiba se orientar a respeito.

* Com a referência de `Despesas Stack View` selecionada no _Document Outline_, acesse a seção do _Size Inspector_.
* Na parte inferior da seção, mais precisamente no item _Content Hugging Priority_, diminua o valor referente a direção vertical para `249`.

Com o valor para _content hugging priority_ reduzido, maior a probabilidade de que seja a _stack view_ com o conteúdo do relatório que se estenda para preencher algum espaço restante da diferença para os tamanhos intrínsecos dos componentes - lembre-se da analogia da tensão de um elástico.

Com o contêiner da seção pronto, nos concentram os em adicionar os elementos necessários. Começando pelo _label_.

* Pressionando a tecla _option ⌥_, clique sobre o `Nova Entrada Label` e arraste uma cópia do mesmo para dentro da nova _stack view_, aproveitando o estilo predefinido.
* Selecione a referência copiada no _Document Outline_ e aproveite para alterar seu identificador para `Relatorio Label`
* Altere também o valor textual do _label_ para "Relatório Parcial", no _Attributes Inspector_.

Além do _label_ que serve de título, vamos precisar de uma _view_ para a lista de despesas, e de uma _stack view_ horizontal para os _labels_ que vão mostrar o valor total.

* Clique no ícone _(+)_ superior direito do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"uiv"_.
* Utilize o _drag and drop_ para arrastar um elemento de View para o _Document Outline_, posicionando-o abaixo do `Relatorio Label`.
* Altere o identificador da nova _view_ para `Lista de Despesas` e, com ela selecionada, navegue até o _Identity Inspector_.
* Na seção _Custom Class_, altere o valor de _Class_ para `ListaDeDespesasView`, usando o suporte do Interface Builder para auto completar o valor.
* Na seção _Size Inspector_, altere o valor da propriedade _Content Hugging Priority_ referente à direção vertical para `249`, permitindo assim que a lista se expanda para preencher o espaço vazio, caso exista.
* Caso queira, para que seja fácil visualizar o elemento no Canvas, altere o _background color_ para _System Secondary Background Color_.

Em seguida, vamos aos _labels_.

* Clique no ícone _(+)_ superior direito do Interface Builder ou use o atalho _⌘⇧ + L_.
* No campo de busca digite _"uil"_.
* Utilize o _drag and drop_ para arrastar um elemento de Label para o _Document Outline_, posicionando-o abaixo da `ListaDeDespesasView`.
* Pressionando a tecla _option ⌥_, clique no novo _label_ e arraste para baixo para criar um cópia no mesmo nível hierárquico.
* Altere os identificadores de ambos os _labels_ para `Total Label` e `Valor Total Label`, respectivamente.
* Selecione ambos os _labels_, clique no ícone inferior direito referente a feature _Embed In_, e em seguida selecione _Stack View_ na seção _Embed In View_.
* Altere o identificador da nova _stack view_ para `Totais Stack View`.
* Selecione o _stack view_ e, no _Attributes Inspector_, altere sua propriedade _Axis_ para _Horizontal_, assim como altere sua propriedade _Spacing_ para `4`.
* Selecione primeiro _label_, `Total Label`, e através do _Size Inspector_, altere sua _content hugging priority_ referente à direção horizontal para `250`.

Vamos ajustar o estilo do primeiro _label_.

* Selecione o `Total Label` e, no _Attributes Inspector_, altere seu valor textual para "Total:".
* Altere também a cor da fonte para _Secondary Label Color_ e a fonte para `System 14.0`.
* Para a propriedade de _Alignment_ do texto, selecione _Right_.

Agora vamos ajustar o _label_ restante, referente ao valor total. Neste momento o _label_ apresentará um valor monetário fictício. Resolveremos isso durante a implementação.

* Selecione o `Valor Total Label` e, no _Attributes Inspector_, altere seu valor textual para "R$ 100,00".
* Altere também a fonte para `System 18.0`.

Com todos os ajustes feitos, devemos ter nosso Interface Builder de acordo com a imagem abaixo.

<p align="center">
<img alt="Imagem do Interface Builder no estado final da implementação da view" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-lt2-imitation-despesas-view-final.png?raw=true" width="80%"/>
</p>

## Iniciando a codificação do View Controller

### Organizando o projeto

Agora é hora de começar a preparar o View Controller para que seja possível gerenciar a funcionalidade. Antes de tudo precisamos seguir nossa convenção para organização de grupos e pastas também na camada de controle, assim como ter um nome mais adequado para o controlador.

* Abra o arquivo View Controller no editor de código, selecione o identificador `ViewController` à frente da palavra reservada `class`, e com um clique com o botão direito, selecione _Refactor > Rename_.
* Substitua o nome para `RelatorioDeDespesasViewController`. _(perceba que tanto o código, quanto o nome do arquivo e referência ao mesmo dentro storyboard foram substitúidos corretamente)_
* Em seguida, no _Project Navigator_, clique com o direito sobre a pasta com o nome do projeto (a mais interna, que tem o ícone adequado de pasta), e selecione _New Group_.
* Para o novo grupo, agora em estado de edição, dê o nome de `Scenes`. Arraste a referência da pasta para mais próximo das já existentes _Models/_ e _View/_.
* Como não é necessário criar grupos internos, já que temos uma única cena na aplicação, apenas arraste o arquivo `RelatorioDeDespesasViewController.swift` para dentro do grupo `Scenes`.

Pronto, temos nossos grupos devidamente organizados e com alguma convenção sendo seguida.

### Conectando _outlets_ e _actions_

Já temos quase tudo pronto para começar a codificar nossa solução. Precisamos apenas das devidas ligações entre as views representadas no arquivo `Main.storyboard` e nosso código Swift. Faremos isso agora.

<p align="center">
<img alt="Animação da conexão dos outlets para a lista de despesas e label com valor total" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-lt2-imitation-outlets.gif?raw=true" width="80%"/>
</p>

Além dos _outlets_ para a lista de despesas e para o _label_ com valor total, **adicione também outros para os campos de texto do formulário de novas entradas e para o botão de registro de relatório**. Ao final das conexões você deve ter para seu controlador um código parecido com o que segue:

``` swift
import UIKit

class RelatorioDeDespesasViewController: UIViewController {
    
    @IBOutlet weak var tituloTextField: UITextField!
    @IBOutlet weak var tipoTextField: UITextField!
    @IBOutlet weak var valorTextField: UITextField!
    
    @IBOutlet weak var listaDeDespesasView: ListaDeDespesasView!
    @IBOutlet weak var valorTotalLabel: UILabel!
    
    @IBOutlet weak var registrarButton: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }

}
```

> Dica: Posicionar as propriedades armazenadas conectadas via `@IBOutlet` no código na mesma ordem que os elementos de _view_ são dispostos no `.storyboard` e seguindo blocos lógicos de acordo com a UI pode facilitar o entendimento da relação código-desenho da view.

Com os _outlets_ configurados, falta apenas conectar as _actions_ para que possamos focar apenas em código Swift. Vamos a isso.

<p align="center">
<img alt="Animação da conexão das actions para os botões e criação de code snippet para função" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-lt2-imitation-actions.gif?raw=true" width="80%"/>
</p>


Você deve criar conexões para ter os _snippets_ de código das funções de acordo com o código a seguir:

``` swift
import UIKit

class RelatorioDeDespesasViewController: UIViewController {
    
    // código anterior omitido ... 
    
    @IBAction func botaoAdicionarDespesaPressionado(_ sender: UIButton) {
        // em breve código aqui
    }
    
    @IBAction func botaoRegistrarDespesasPressionado(_ sender: UIButton) {
        print("vai para uma tela de revisão...")
    }

}

```

Pronto! Nosso _controller_ está pronto para darmos seguimento a implementação. Para isso precisamos conhecer nosso modelo.

## Conhecendo o domínio

A partir do ponto onde estamos já é possível responder a eventos gerados pelo usuário através da UI, e receber os dados gerados pelos mesmos. Mas antes de escrevermos qualquer código adicional, primeiro é necessário fazer algumas perguntas.

1. O que eu quero fazer quando o usuário toca no botão _Adicionar +_
1. Quais tipos de validação devem ser feitas na interação?
1. O que os dados provenientes do formulário representam? E como planejar o trabalho com essa representação na aplicação?
1. O que acontece com o estado da aplicação assim que uma nova despesa é informada?

Respondendo a essas perguntas conhecemos um pouco mais sobre o domínio do problema que a funcionalidade se propõe a resolver, e assim, podemos modelar de acordo.

### Revisando o caso de uso para a funcionalidade

Nossa funcionalidade tem por objetivo principal registrar dados de **despesas**, através de um **relatório de despesas** em um pseudo-aplicativo de solicitação de reembolsos para itens do seu home office. 

Por meio de um formulário simples o usuário pode incluir uma despesa informando seu título, tipo e valor. Além das propriedades auto explicativas, o tipo de despesa tem uma característica mais específica: ele pode assumir uma das opções entre **Eletrônicos** e **Escritório**, que são representados pelos valores `1` e `2`, respectivamente. O tipo precisa também fornecer uma representação textual adequada para a apresentação na lista de despesas.

Através do evento gerado pelo toque do botão de adição de novas despesas, se faz necessário garantir que os dados enviados por ele estejam de acordo com a representação de uma despesa. Sendo assim, é necessário que os dados sejam validados de maneira prévia. 

O **relatório de despesas** que a funcionalidade visa registrar possui uma data de criação, um valor total e as despesas que são adicionadas. Para conter alguns detalhes de implementação que ainda vamos estudar mais a frente, e facilitar o escopo desta tarefa, já existe uma implementação para uma entidade do modelo representando o conjunto de despesas adicionadas a um relatório, a estrutura `Despesas`.

### Modelando o domínio

Com os requisitos e detalhes de funcionamento claros, já é possível modelar no entorno do módulo fornecido (`Despesas`) buscando uma representação virtual do problema da vida real. Algumas entidades importantes para o problema já são possíveis de serem reconhecidas, como: `Relatório de Despesas`, `Despesas` (que representa o conjunto, encapsulando detalhes de implementação) e `Despesa`. É possível favorecer também a relação com o tipo de despesa, dadas as suas necessidades um pouco peculiares.

<p align="center">
<img alt="Imagem com modelo de domínio para o problema" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-lt2-imitation-modelo-de-dominio.jpg?raw=true" width="80%"/>
</p>

## Implementando o modelo

> Nota: Este pode ser um bom momento para descomentar o conteúdo fornecido nos arquivos `RelatorioDeDespesas.swift` e `ListaDeDespesasView.swift`. Por um momento é possível que a aplicação não consiga compilar com sucesso, mas isso se dá justamente pela ausência de modelos criados nos passos a seguir. 🙂

Agora que temos tudo bem definido, podemos prosseguir com a implementação para o modelo conceitual produzido anteriormente. Começando por `RelatorioDeDespesas`, abra o arquivo que vive no grupo `Models/`, e perceba que já existe a implementação de parte do modelo. Os detalhes de implementação do código existente não são importantes para o escopo desta atividade, então concentre-se em expressar a ideia do relatório apenas.

Adicione a seguinte implementação para o modelo:

``` swift
struct RelatorioDeDespesas {
    let dataDeCriacao: Date = Date.now
    private(set) var despesas: Despesas = Despesas()
    private(set) var valorTotal: Decimal = 0
    
    mutating func adiciona(_ despesa: Despesa) {
        despesas.adiciona(despesa)
        valorTotal = despesas.total
    }
}

// struct Despesas { ... }
```

Perceba que nossa implementação já adiciona a `dataDeCriação` e `valorTotal`, assim como estabelece a relação entre o relatório e o conjunto de despesas, à medida que uma instância de `RelatorioDeDespesas` é criado. Assim, a cada vez que a tela de relatótio for apresentada, fica simples imaginar o que fazer para se ter o modelo sobre o qual incidem as alterações do usuário.

A seguir, vamos tratar de resolver o problema de compilação causado atualmente pela ausência de uma representação de `Despesa`.

Adicione a seguinte implementação ainda no arquivo `RelatorioDeDespesas.swift`:

``` swift
// struct RelatorioDeDespesas { ... }

// struct Despesas { ... }

struct Despesa {
    let titulo: String
    let tipo: String
    let valor: Decimal
}
```

Assim temos quase tudo pronto, exceto por ainda não representar adequadamente as particularidades do `tipo` de despesa de acordo com o domínio. Como visto, o tipo de despesa pode assumir dois possíveis valores bem definidos, representados a partir de inteiros e com representação textual. Essa parte específica do problema é uma excelente candidata ao uso de Enums no modelo.

``` swift
struct Despesa {
    let titulo: String
    let tipo: Tipo // <--- tipo agora é um Tipo (enum type) e não String
    let valor: Decimal
    
    enum Tipo: Int {
        case eletronicos = 1
        case escritorio = 2
    
        var text: String {
            switch self {
            case .eletronicos:
                return "Eletrônicos"
                
            case .escritorio:
                return "Escritório"
            }
        }
    }
}
```

Como o tipo da despesa no nosso caso de uso é intrínsecamente ligado à própria despesa, para efeitos práticos, modelamos como um tipo interno à `Despesa`. Referências a ele podem ser feitas a partir de `Despesa.Tipo`.

## Finalizando a implementação: O View Controller como um agente intermediário

Neste ponto da atividade temos representações para a View e para o Modelo do domínio da aplicação, no entanto, embora a lógica base para adição de despesas e as regras para apresentação estejam estabelecidas, a aplicação ainda não é capaz de cumprir seu propósito. Isso pode ser uma boa ilustração de como esses dois grandes grupos de objetos do padrão arquitetural MVC, segundo a visão da plataforma iOS, são dependentes do View Controller atuando como um agente intermediário.

Para todas as interações entre o que ocorre na fronteira com o usuário (view, UI) e com o coração do aplicativo (modelo, dados, lógica) o View Controller é responsável por ser o mediador.

<p align="center">
<img alt="Imagem com ideia de implementação MVC da plataforma iOS" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-lt2-ideia-mvc.jpg?raw=true" width="80%"/>
</p>

Já temos todo o necessário para começar a mediar as relações entre as views e o modelo.

### Validando a entrada de uma nova despesa

Embora algumas abordagens possam levar as validações para a implementação do modelo, por questões de facilidade de implementação vamos convencionar nossa solução levando essa responsabilidade para o código mais próximo da borda externa da aplicação.

No código do View Controller, adicione a seguinte implementação:

``` swift
import UIKit

class RelatorioDeDespesasViewController: UIViewController {
    
    typealias MensagemDeErro = String
    
    // @IBOutlet propriedades armazenadas aqui
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }

    func exibeAlerta(para mensagemDeErro: MensagemDeErro?) {
        let alert = UIAlertController(
            title: "Erro",
            message: mensagemDeErro ?? "Verifique os dados informados e tente novamente.",
            preferredStyle: .alert)
        
        alert.addAction(UIAlertAction(title: "OK", style: .cancel, handler: nil))
        
        self.present(alert, animated: true, completion: nil)
    }

    func formularioEhValido() -> (Bool, MensagemDeErro?) {
        if let titulo = tituloTextField.text, titulo.isEmpty {
            return (false, "Título inválido")
        }
        
        guard let tipoComoTexto = tipoTextField.text,
              let tipo = Int(tipoComoTexto),
              let _ = Despesa.Tipo(rawValue: tipo) else {
            return (false, "Tipo de despesa inválido")
        }
         
        guard let valorEmTexto = valorTextField.text,
              let _ = Converter.paraDecimal(string: valorEmTexto) else {
            return (false, "Valor inválido")
        }
            
        return (true, nil)
    }
    
    @IBAction func botaoAdicionarDespesaPressionado(_ sender: UIButton) {
        switch formularioEhValido() {
        
        case (false, let mensagem):
            exibeAlerta(para: mensagem)
        
        default:
            //adicionaDespesa()
        }
    }
    
    // @IBAction func botaoRegistrarDespesasPressionado(_ sender: UIButton) { ... }
       
}
```

> Nota: Caso a complexidade do código para a validação adicione dificuldades no entendimento do fluxo base do controlador, pode ser interessante a construção de componentes específicos para conter essa responsabilidade. Isso pode trazer grandes vantagens à implementação.

Bastante código foi introduzido aqui, no entanto, você já deve estar familiarizado com ele dadas as atividades realizadas anteriormente no nosso treino. Vamos então focar nos pontos específicos que merecem destaque.

``` swift
    func formularioEhValido() -> (Bool, MensagemDeErro?) {
        // checagem do título   
        
        guard let tipoComoTexto = tipoTextField.text,
              let tipo = Int(tipoComoTexto),
              let _ = Despesa.Tipo(rawValue: tipo) else {
            return (false, "Tipo de despesa inválido")
        }
         
        guard let valorEmTexto = valorTextField.text,
              let _ = Converter.paraDecimal(string: valorEmTexto) else {
            return (false, "Valor inválido")
        }
            
        return (true, nil)
    }
```

Parte do trabalho de garantir que o dado informado esteja dentro do esperado começa na _view_, com as configurações adequadas dos _inputs_. No entanto, como já é esperado, recebemos as informações no controlador como texto. Assim, parte do trabalho de validar é, num contexto controlado, fazer o _parse_ dos dados buscando adequa-los ao modelo. É exatamente o que o código destaca acima faz.

Inicializamos um tipo de despesa, representado pela enum `Despesa.Tipo`, a partir do número inteiro informado pelo usuário, previamente convertido através da string em `tipoTextField.text`. Assim como no caso anterior, o decimal necessário para representar o valor da despesa é construído através da string eventualmente em `valorTextField.text`. Em qualquer dos casos onde haja falha, temos um erro de validação sendo informado ao usuário, ao invés do prosseguimento da adição.

### Coordenando a adição de uma despesa

Caso não ocorra qualquer falha ao processar o evento gerado pelo botão podemos dar prosseguimento à lógica. Com o sucesso das operação de adequação assegurados pela etapa de validação podemos agora trabalhar de forma mais simples, instanciando uma nova despesa utilizando os inicializadores fornecidos automaticamente pelas `structs` do Swift.

Adicione código à implementação do seu View Controller para chegar ao resultado abaixo.

``` swift
import UIKit

class RelatorioDeDespesasViewController: UIViewController {
    
    typealias MensagemDeErro = String
    
    // @IBOutlet propriedades armazenadas aqui
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }

    // func formularioEhValido() -> (Bool, MensagemDeErro?) { ... }
    
    @IBAction func botaoAdicionarDespesaPressionado(_ sender: UIButton) {
        switch formularioEhValido() {
        
        case (false, let mensagem):
            exibeAlerta(para: mensagem)
        
        default:
            adicionaDespesa()
        }
    }

    func adicionaDespesa() {
        let codigo = Int(tipoTextField.text!)!
        let tipo = Despesa.Tipo(rawValue: Int(codigo))!
        
        let despesa = Despesa(titulo: tituloTextField.text!,
                              tipo: tipo,
                              valor: Converter.paraDecimal(string: valorTextField.text!)!)
        
        // relatorio.adiciona(despesa) já é possível
    }
    
    // @IBAction func botaoRegistrarDespesasPressionado(_ sender: UIButton) { ... }
    
}
```

Avançamos com este código para o ponto onde já seria possível garantir que o relatório adicione a nova despesa. Mas qual relatório? Ainda não temos no nosso View Controller nenhuma forma de manter o estado de um relatório de despesas.

### Gerenciando o relatório de despesas

Precisamos agora garantir que o View Controller tenha uma instância de `RelatorioDeDespesas` onde vão incidir as ações de adição do usuário.

Poderíamos supor em um primeiro momento que o View Controller de relatório de despesas, a partir do momento de sua criação, inicialize um novo `RelatorioDeEmpresas` e guarde sua referência utilizando uma propriedade armazenada.

``` swift
class RelatorioDeDespesasViewController: UIViewController {
    
    // typealias
    // @IBOutlet propriedades armazenadas aqui

    var relatorioDeDespesas = RelatorioDeDespesas()

```

Mas por esta abordagem podemos impor ao View Controller a responsabilidade de satisfazer parte das próprias dependências, aumentando ainda mais seu acoplamento com outros módulos da aplicação. 

Embora já não seja lá muito fácil, ou mesmo usual, a abordagem de escrever testes unitários para o comportamento do controlador, imagine que fossemos por este caminho e gostaríamos de faze-lo. Como garantir que o comportamento seja verificado em isolamento? =/ Todo o comportamento depende da implementação de `RelatorioDeDespesas` e não há uma forma de mudar isso, como usando mocks, por exemplo.

Indo nessa direção, poderíamos supor então a utilização do padrão _Dependency Injection_ através de inicializadores para prover a instância de forma mais flexível, podendo assim facilmente passar outro objeto em seu lugar em situações como a citada acima.

``` swift
class RelatorioDeDespesasViewController: UIViewController {
    
    // typealias
    // @IBOutlet propriedades armazenadas aqui

    var relatorioDeDespesas: RelatorioDeDespesas

    init(relatorioDeDespesas: RelatorioDeDespesas) {
        self.relatorioDeDespesas = relatorioDeDespesas
        super.init(nibName: nil, bundle: nil)
    }
    
    // código posterior omitido
```

Porém a inicialização do View Controller ocorre automaticamente através do carregamento do arquivo `Main.storyboard`, que por decisão de design da plataforma para as aplicações que o utilizam, requer o uso de um inicializador específico cuja assinatura não pode ser alterada. Então somos ainda obrigados a satisfazer a dependência internamente, na _branch_ de código que a aplicação executa no dispositivo dos usuários. =/

``` swift
class RelatorioDeDespesasViewController: UIViewController {
    
    // typealias
    // @IBOutlet propriedades armazenadas aqui

    var relatorioDeDespesas: RelatorioDeDespesas

    init(relatorioDeDespesas: RelatorioDeDespesas = RelatorioDeDespesas()) {
        self.relatorioDeDespesas = relatorioDeDespesas
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) { // utilizado pelo UIKit a partir do carregamento do storyboard
        self.relatorioDeDespesas = RelatorioDeDespesas()
        super.init(coder: coder)
    }
    
    // código posterior omitido
```

O código acima já seria suficiente para suprir a necessidade de trocar a implementação de `RelatorioDeDespesas` para o View Controller em um possível teste, mas ainda apresenta algumas dificuldades. Imagine que nosso modelo evolua, e sua inicialização se torne mais complexa, necessitando por exemplo de outro objeto como parâmetro. O que acontece com o código do View Controller? =/

Isso mesmo que você deve ter imaginado. Se o View Controller satisfaz a própria dependência do `RelatorioDeDespesas`, ele acaba de ser penalizado por isso, e deve também suprir as necessidades que o mesmo possa requerer. Por esse raciocínio também não vamos seguir por esta abordagem. Vamos usar ainda a ideia de receber a injeção da dependência, mas não através de inicializadores.

``` swift
class RelatorioDeDespesasViewController: UIViewController {
    
    // typealias
    // @IBOutlet propriedades armazenadas aqui

    var relatorioDeDespesas: RelatorioDeDespesas?
    
    // código posterior omitido
```

Perceba a alteração do código. Para que possamos trabalhar com a propriedade armazenada sem o compilador exercer controle e nos obrigar a inicialização, podemos utilizar um _optional type_ para gerenciar a referência de `RelatorioDeDespesas`.

Mas de onde virá então a referência para a instância? A resposta é: "Isso não é da conta do View Controller!". Ele precisa contar com a referência, depende dela, mas apenas isso. Ele não toma controle sobre sua criação e delega para cima essa responsabilidade. Qualquer que seja o módulo da aplicação que queira contar com seus serviços agora é responsável por prover a instância e injetar essa dependência através da propriedade para o correto funcionamento. A melhor forma de resolver o problema de código nesse ponto, é considerar que o problema não é dele. 

Para dar mais clareza então sobre quem vai se responsabilizar por isso, podemos pensar um pouco mais sobre o contexto de utilização deste pseudo-aplicativo.

Estamos apenas desenvolvendo uma das telas de um aplicativo de solicitação de reembolsos para itens de home office. Exercitando nossa criatividade, podemos supor que antes dessa tela ganhar foco, possivelmente temos uma tela principal com todos os relatórios gerados anteriormente, com seus devidos status de execução, etc e etc. Dessa tela anterior no fluxo de utilização - ou de qualquer outra - provavelmente deve vir um evento de transição para a nossa tela. E é exatamente para quem gerencia este ponto do código, que estamos delegando a responsabilidade de satisfazer a nossa dependência.

No contexto do nosso projeto de desafio, esse ponto anterior do fluxo de utilização não existe. Ou pelo menos, não como a gente imaginou nesse exercício de criatividade. Conforme já conhecemos estudando View Controllers anteriormente, existe um módulo responsável por determinar como a aplicação inicializa as cenas que serão dispostas na janela do aplicativo, e portando é quem participa do processo de construção do View Controller e disponibilização do mesmo como `rootViewController`: o `SceneDelegate`.

Para o nosso propósito ele será suficiente. Adicione implementação ao seu projeto para conter o seguinte código:

``` swift
// SceneDelegate.swift
import UIKit

class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        // Use this method to optionally configure and attach the UIWindow `window` to the provided UIWindowScene `scene`.
        // If using a storyboard, the `window` property will automatically be initialized and attached to the scene.
        // This delegate does not imply the connecting scene or session are new (see `application:configurationForConnectingSceneSession` instead).
        guard let _ = (scene as? UIWindowScene) else { return }
        
        let relatorio = RelatorioDeDespesas()
        
        let relatorioViewController = window!.rootViewController as! RelatorioDespesasViewController
        relatorioViewController.relatorioDeDespesas = relatorio
    }

    // código posterior omitido
```

E assim já podemos atualizar nosso View Controller para garantir a adição da despesa ao formulário.

``` swift
import UIKit

class RelatorioDeDespesasViewController: UIViewController {

    // codigo anterior omitido

    var relatorioDeDespesas: RelatorioDeDespesas?
    
    // código omitido
    
    @IBAction func botaoAdicionarDespesaPressionado(_ sender: UIButton) {
        switch formularioEhValido() {
        
        case (false, let mensagem):
            exibeAlerta(para: mensagem)
        
        default:
            adicionaDespesa()
        }
    }

    func adicionaDespesa() {
        let codigo = Int(tipoTextField.text!)!
        let tipo = Despesa.Tipo(rawValue: Int(codigo))!
        
        let despesa = Despesa(titulo: tituloTextField.text!,
                              tipo: tipo,
                              valor: Converter.paraDecimal(string: valorTextField.text!)!)
        
        relatorioDeDespesas?.adiciona(despesa)
    }
    
    // @IBAction func botaoRegistrarDespesasPressionado(_ sender: UIButton) { ... }

}
```

Com as alterações feitas até esse ponto já é possível utilizar a aplicação. Rode a aplicação em seu simulador e teste o comportamento.

Nossas despesas estão sendo adicionadas - você pode adicionar um `print(despesas)` no código do modelo para se certificar. Mas por que nada se reflete na tela?

### Atualizando a View com as alterações observadas no Modelo

Até o momento a resposta mais precisa para a pergunta que permaneceu aberta na seção anterior é: "porque não implementamos! 🤷🏻‍♂️". Então vamos adicionar o código responsável por atualizar as _views_ necessárias para uma nova versão ou estado de um relatório de despesas.

``` swift
import UIKit

class RelatorioDeDespesasViewController: UIViewController {

    // codigo anterior omitido

    func adicionaDespesa() {
        let codigo = Int(tipoTextField.text!)!
        let tipo = Despesa.Tipo(rawValue: Int(codigo))!
        
        let despesa = Despesa(titulo: tituloTextField.text!,
                              tipo: tipo,
                              valor: Converter.paraDecimal(string: valorTextField.text!)!)
        
        relatorioDeDespesas?.adiciona(despesa)
    }
    
    func atualizaViews(para relatorio: RelatorioDeDespesas) {
        listaDeDespesasView.atualiza(relatorio.despesas)
        
        valorTotalLabel.text = Formatter.paraMoeda(decimal: relatorio.valorTotal)
        registrarButton.isEnabled = relatorio.valorTotal > 0
    }
    
    // @IBAction func botaoRegistrarDespesasPressionado(_ sender: UIButton) { ... }

}
```

Já temos a função que realiza o trabalho necessário. Basta então que a invoquemos no ponto certo do código e _voilá_. Poderíamos supor:

``` swift
    func adicionaDespesa() {
        // código omitido
        
        relatorioDeDespesas?.adiciona(despesa)
        atualizaViews(para: relatorioDeDespesas) // mas não compila
    }
```

O compilador nos impediria por conta da necessidade do _unwrapping_ do opcional em `relatorioDeDespesas`. Poderíamos seguir por este caminho. Mas já imagine outras alterações no modelo provenientes de outros possíveis eventos. Em todos as funções que potencialmente alteram o modelo se faria necessário a adição desse tipo de código. Parece muito trabalho. 

Por outro lado poderíamos supor receber um opcional `relatorioDeDespesas` como parâmetro em `atualizaViews(relatorio:)`, afrouxando sua interface. Mas isso por si só já não faria sentido, dado que não se espera atualizar as _views_ em uma situação onde não exista um relatório de despesas. =/

O que pretendemos então aqui, é garantir a integridade do dado ao invocarmos a função `atualizaViews`, mas fazer isso de forma mais prática. Podemos então lançar mão da feature de _property observers_ do Swift.

Usando um _property observer_ para `relatorioDeDespesas`, com uma clausula `didSet` podemos isolar o trabalho necessário para responder a alguma alteração de estado do objeto observado, e de quebra ganhamos como bônus sua invocação automática. Com essa abordagem menos imperativa, mais ao estilo reativa, facilitamos, e muito, a implementação do View Controller, suavizando a responsabilidade das funções que atualizam o modelo.

``` swift
class RelatorioDeDespesasViewController: UIViewController {
    
    // código anterior omitido
    
    var relatorioDeDespesas: RelatorioDeDespesas? {
        didSet {
            guard isViewLoaded, let relatorioDeDespesas = relatorioDeDespesas else { return }
            atualizaViews(para: relatorioDeDespesas)
        }
    }

    // código posterior omitido
```

Repare na verificação adicional garantida pela propriedade `isViewLoaded`. Ela se faz necessária porque, de outra forma, a inicialização do controlador falharia. O _property observer_ dispararia uma atualização de view assim que o código do `SceneDelegate` executasse `relatorioViewController.relatorioDeDespesas = relatorio` - num momento onde a `view` para o View Controller ainda não teria sido carregada. Dessa forma o _binding_ dos _outlets_ ainda não teria ocorrido, e uma instrução como `listaDeDespesasView.atualiza(relatorio.despesas)` não seria possível.

> Nota: Uma curiosidade é que poderíamos imaginar uma checagem análoga com `guard let _ = view else { return }` para assegurar que a `view: View!` para o ViewController exista. No entanto, a documentação de View Controller deixa claro que isso deve ser evitado, a menos que você intencionalmente queira fazer com que a `view` esteja na memória. Acessar a propridade `view` de um View Controller quando esta for `nil` faz com que automaticamente o controlador dispare a chamada para `loadView()` forçando o carregamento da _root view_ para a memória. No geral, é recomendado não interferir no ciclo de vida programado e deixar as coisas seguirem seu curso normalmente.

Fechando nossa atividade, adicione uma chamada para `atualizaViews` na implementação de `viewDidLoad`. Assim asseguramos que a aplicação tenha seu estado inicial correto quando do carregamento da tela, evitando que os valores da representação da _view_ no arquivo _storyboard_ sejam usados como padrão.

``` swift
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Do any additional setup after loading the view.
        if let relatorioDeDespesas = relatorioDeDespesas {
            atualizaViews(para: relatorioDeDespesas)
        }
    }
```

Pronto! Neste momento nossa implementação está completa. Fique à vontade para rodar a aplicação e testar seu comportamento.

## Conclusão

Conseguimos concluir nosso trabalho passando por todos os pontos do desenvolvimento de uma tela. Espero que tenha curtido navegar por este caminho cognitivo e construir a funcionalidade entendendo os aspectos da arquitetura padrão de um projeto iOS. Existem pontos positivos e negativos no design proposto pelo padrão, assim como algumas alternativas a ele propostas pela comunidade, mas já temos um bom ponto de partida para pensar sobre nossas soluções e como as decisões de design podem impactar a qualidade do código proposto do dia-a-dia do desenvolvimento.