>Esse material teórico foi atualizado tendo como base as fontes originais em:
>   - https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/Outlet.html
>   - https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/TargetAction.html
>   - https://developer.apple.com/documentation/uikit/uialertcontroller

# Entendendo IBOutlets

Um _outlet_ é uma propriedade armazenada que é anotada com `@IBOutlet` e cujo valor você pode definir graficamente em um arquivo Storyboard (ou arquivo interface builder equivalente). Você declara um _outlet_ em uma classe e faz uma conexão entre o _outlet_ e outro objeto no storyboard. Quando a _view_ referente a um _View Controller_ no arquivo Storyboard é carregada, a conexão é estabelecida.

<p align="center">
<img alt="uma imagem com um diagrama ilustrando a relação entre o outlet como propriedade armazenada de um View Controller e o objeto View declarado no Interface Builder conectado ao mesmo" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-outlets-imagem-vc-outlet.jpg?raw=true" width="60%" />
</p>

Você define um _outlet_ como uma propriedade armazenada com a anotação `@IBOutlet`:

``` swift
@IBOutlet var loginTextField: UITextField!
```

A anotação `IBOutlet` é usado apenas pelo Xcode, para determinar quando uma propriedade é um _outlet_, não tendo valor real para a semântica do código da classe em si.

Por meio de um _outlet_, um objeto em seu código pode obter uma referência a um objeto definido em um arquivo Storyboard e, em seguida, pode ser carregado tendo como base as definições desse arquivo. O objeto que contém um _outlet_ geralmente é um objeto _View Controller_ customizado. Você frequentemente define _outlets_ para poder enviar mensagens para objetos de _View_ do UIKit framework. 

Um _outlet_ pode ser especialmente útil para evitar o trabalho de manipulação das referências das _subviews_ da _root view_ (em _View Controllers_) em uma hierarquia complexa. Provendo uma referência direta a um objeto _View_ muitos níveis abaixo na árvore de componentes de uma tela, um _outlet_ facilita o código necessário para que um _View Controller_ configure dinamicamente o estado inicial correto para um objeto _TextField_. Como no exemplo ilustrado acima, é possível de forma direta solicitar que um campo de texto seja focado a partir do carregamento da _view_ de uma tela.

## Conectando IBOutlets

O XCode oferece alguns níveis de suporte para criar e conectar _outlets_ entre seu Interface Builder e o editor de código. Nesta seção veremos duas possibilidades. Param ambos os exemplos utilizaremos um aplicativo de exemplo simples contendo apenas um `UILabel` que se destina a exibir uma mensagem de boas vindas. Apenas as contraints básicas de posicionamento foram adicionadas e a fonte ajustada, como é possível perceber na imagem abaixo:

<p align="center">
<img alt="uma imagem com o app de exemplo em seu estado inicial" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-outlets-projeto-exemplo.png?raw=true" width="60%" />
</p>

### Conectando do View Controller para os objetos no Canvas

A primeira forma de conectar suas _views_ com código é usando a referência do _controller_ que gerencia a tela, visível na barra de menus no topo do objeto `View Controller` do canvas.

Primeiro é necessário criar a propriedade armazenada que guardará a referência no código de sua classe `ViewController`:

``` swift
import UIKit

class ViewController: UIViewController {
    
    @IBOutlet var welcomeLabel: UILabel! // <--

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }

}
```

A partir desse ponto, voltando-se para o Interface Builder, é possível usar o assistente do XCode para, pressionando a tecla _Control_, clicar e arrastar o ponteiro do mouse do ícone que indica o _View Controller_ até o componente de _label_.

<p align="center">
<img alt="um gif animado com a demonstração do suporte do xcode para conexão através do ViewController" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-outlets-conectando-do-vc.gif?raw=true" width="60%" />
</p>

Perceba que ao soltar o ponteiro do mouse dentro dos limites do componente de _label_, o XCode abre uma caixa de diálogo com as opções de propriedade para conexão. Basta então selecionar o nome da propriedade armazenada que criamos (`welcomeLabel`) e temos nossa conexão completa.

A partir desse ponto, é possível interagir com o objeto no código do `ViewController`. Podemos definir dinamicamente a mensagem que a app deve apresentar ao usuário, por exemplo, com o código abaixo:

``` swift
import UIKit

class ViewController: UIViewController {
    
    @IBOutlet var welcomeLabel: UILabel!

    override func viewDidLoad() {
        super.viewDidLoad()
        
        welcomeLabel.text = "Olá, Rafael!" // <--
    }

}
```

Adicionamos a instrução `welcomeLabel.text = "Olá, Rafael!"`, que altera a propriedade de texto do _label_ de boas vindas, ao método `viewDidLoad()` que tem sua execução quando do carregamento da _view_ deste _View Controller_. Assim, fica assegurado que quando a app renderiza na tela já temos a informação atualizada.

<p align="center">
<img alt="imagem com o app exemplo funcionando atraves da funcionalidade implementada com o outlet" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-outlets-outlet-aplicado.png?raw=true" width="320" />
</p>

Perceba que nesse momento é possível visualizar as conexões estabelecidas entre o Interface Builder e o código da aplicação através do inspetor de conexões (_Connection inspector_).

<p align="center">
<img alt="imagem com o inspector de connections mostrando as conexões atuais entre o interface builder e o código" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-outlets-connections-inpector.png?raw=true" width="60%" />
</p>

>Nota: Um cuidado especial é necessário ao gerenciar as conexões. Um erro comum é remover uma propriedade armazenada conectada ao Interface Builder através de um _outlet_ apenas no código, fazendo que a aplicação pare por conta de erros ao tentar carregar as _views_ do Storyboard. Caso precise remover uma propriedade configurada como _outlet_, ou mesmo mudar seu nome, certifique-se de remover a conexão do Interface Builder através do _Connections inspector_, e caso seja necessário, refazê-la após alguma refatoração de código.

### Conectando do Interface Builder para o código fonte

Conectar objetos desenhados no Interface Builder a seus arquivos de código é uma tarefa corriqueira ao desenvolver as funcionalidades de um aplicativo. E justamente por se repetir tanto, imagina-se que em algum momento esse trabalho fique automático. E todo trabalho automatizado se torna menos propenso a erros quando provido pelas ferramenta que compõem o ambiente de desenvolvimento.

Você deve ter notado no exemplo acima que parte do trabalho ainda ficou a seu cargo: escrever o código que define a propriedade armazenada para o _outlet_. Até mesmo essa parte do trabalho pode ser executada pelo XCode quando conectando nossas _views_ no Storyboard com o código.

Para que seja possível aprender essa forma ainda mais assistida de trabalho, voltemos ao estado inicial do projeto exemplo, sem qualquer conexão criada no IB ou alteração no código fonte.

<p align="center">
<img alt="uma imagem com o app de exemplo em seu estado inicial" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-outlets-projeto-exemplo.png?raw=true" width="60%" />
</p>

Ao invés de dessa vez começar alterando o código fonte, vamos trabalhar diretamente sobre o Interface Builder. Com a janela do IB aberta, com a tela principal em foco, podemos clicar no ícone do canto superior direito referente a adição de editores à direita (_Add Editor on Right_). Na sequência, no novo editor aberto, clicamos no ícone referente à navegação para itens relacionados (_Navigate to Related Items_) no canto superior esquerdo dessa janela, e em seguida selecionamos _Automatic (1) > `ViewController.swift`_, para abrir o _View Controller_ que gerencia a tela. 

<p align="center">
<img alt="um gif animado com a demonstração de como abrir o editor auxiliar com o codigo relativo à tela atual do Interface Builder" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-outlets-editor-auxiliar.gif?raw=true" width="60%" />
</p>

Desta forma, temos lado a lado no XCode o Interface Builder e o editor de código com o arquivo adequado sendo apresentado. Podemos assim usar o mesmo recurso de, pressionando a tela _Control_, clicar e arrastar. Porém dessa vez o faremos do objeto da _view_ no canvas para o ponto do código onde colocaríamos a propriedade armazenada.

<p align="center">
<img alt="um gif animado com a demonstração de como usar o click and drag do Interface Builder para o código" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-outlets-click-drag.gif?raw=true" width="60%" />
</p>

Perceba que ao soltar o mouse no editor de código o XCode abre a caixa de diálogo auxiliar para adição de _outlets_. Basta então que selecionemos as configurações desejadas (no geral aceitando o padrão sugerido) e que digitemos o nome adequado para a variável propriedade armazenada que será criada, nesse caso `welcomeLabel`. Um clique em _Connect_ e _voilà_, temos o _outlet_ devidamente adicionado ao código e com a conexão com o Interface Builder configurada. 

``` swift
import UIKit

class ViewController: UIViewController {
    
    @IBOutlet weak var welcomeLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()

        welcomeLabel.text = "Olá, Rafael!"
    }

}
```

Podemos prosseguir com passo final do exemplo anterior, utilizando a referência para configurar a mensagem, como o exemplo de código acima mostra.

>Nota: Você deve ter notado a palavra-chave `weak` na declaração da propriedade armazenada. Não se preocupe, no momento ela ainda não é nosso tema de estudo. Mais adiante teremos informações a respeito do que ela representa. Por hora podemos apenas seguir utilizando-a.

# Entendendo IBActions

Uma _action_ é uma função que é anotada com `@IBAction` e cuja implementação responde a um evento gerado por alguma _view_ que tem as capacidades de um controle, como por exemplo um botão, um slider e outras possíveis _views_. Você pode implementar essa função no código de um controlador e definir graficamente uma conexão a um controle adicionado no arquivo Storyboard (ou arquivo interface builder equivalente). Quando a _view_ for carregada e posteriormente o objeto cuja ação foi conectada receber alguma interação do usuário, automaticamente a função anotada com `@IBAction` será chamada como resposta ao evento, executando algum comportamento customizado.

<p align="center">
<img alt="uma imagem com um diagrama ilustrando a relação entre a action com a execução da resposta do controlador" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-actions-diagrama-evento-botao-vc.jpg?raw=true" width="60%" />
</p>

Um método declarado como _action_ deve ter sua assinatura respeitando alguns padrões. O UIKit framework permite alguma variação de assinatura, mas no geral elas são semelhantes à seguinte:

``` swift
@IBAction func controleAcionado(_ sender: UIControl) {
    // implementação da resposta do evento vai aqui
}
```

Assim como no caso dos _outlets_, a anotação `IBAction` é usada apenas pelo Xcode, para determinar quando um método é uma _action_, não tendo valor real para a semântica do código da função em si.

Para que um método _action_ seja disponibilizado no Interface Builder como conexão, primeiro você deve declará-lo no código do arquivo que controla a porção da sua UI onde o componente está inserido. No nosso exemplo simples, no próprio _View Controller_.

O parâmetro `sender` é o objeto controle que foi acionado e envia a mensagem para a ação. Ao responder a uma mensagem de ação, você pode consultar o `sender` para obter mais informações sobre o contexto do evento que aciona a mensagem de ação.

## Conectando IBActions

Para conectar ações entre um objeto no Interface Builder e o código do controlador relacionado, é possível utilizar o suporte avançado do XCode. Assim como nos casos dos _outlets_, com ele é possível solicitar a conexão e a criação do método no código, evitando erros na assinatura da função.

Com o editor auxiliar à direita, é possível pressionar a tecla _Control_ e clicar no componente de controle desejado no Interface Builder, como por exemplo um botão, e arrastar traçando uma linha em direção ao código do editor ao lado. Ao soltar o ponteiro dentro dos limites do código, o XCode abre uma caixa de diálogo solicitando a configuração adequada para o método da ação. A partir do formulário você pode informar por exemplo o nome adequado para o método, assim como o tipo de parâmetro para o `sender`, e o tipo de evento relacionado.

<p align="center">
<img alt="um gif animado com a demonstração do suporte do xcode para conexão de actions direto do Interface Builder para o código do controller" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-actions-conectando-actions.gif?raw=true" width="60%" />
</p>

Assim podemos adicionar a lógica desejada para a resposta ao evento gerado pelo usuário:

``` swift
import UIKit

class ViewController: UIViewController {
    
    @IBOutlet weak var welcomeLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()

        welcomeLabel.text = "Olá, Rafael!"
    }

    @IBAction func botaoEntrarPressionado(_ sender: UIButton) {
        print("botão pressionado")
    }

}
```

# UIAlertController

Como visto nos materiais teóricos, uma das recomendações de design sobre _View Controllers_ é sempre reutilizar um _View Controller_ caso já exista uma implementação padrão na plataforma que atenda as necessidades. E existe uma infinidade de _View Controllers_ já prontos para vários casos de uso no UIKit framework. Desde controladores com implementações para gerenciar listas ou coleções de valores, carregar arquivos do rolo da câmera, até gerenciar arquivos do iCloud e interagir com o hardware do dispositivo para registrar fotos ou vídeos. 

Alguns dos citados acima veremos algumas seções a frente no treinamento. Mas já é possível conhecer um controlador, e talvez o mais simples, para resolver o problema de apresentação do código acima. Você deve ter notado que utilizamos a simples e já conhecida função `print(_:separator:terminator:)` para imprimir um valor respondendo à ação do usuário. A implementação pode ser suficiente para completar o fluxo de resposta apenas em tempo de desenvolvimento, já que imprimindo na saída padrão do sistema, apenas temos acesso ao dado no console do XCode. É necessário um mecanismo capaz de interagir com a tela do usuário durante a execução do aplicativo.

Caso já tenha utilizado aplicativos iOS, você deve estar familiarizado com uma das formas mais simples e diretas de comunicar algo a um usuário, que é o uso de alertas interativos. Como mencionado acima, evitando a necessidade de programar algo customizado pra esse fim, o UIKit framework já te oferece uma solução padrão: o `UIAlertController`.

`UIAlertController` é uma subclasse de `View Controller` e já conta com uma API clara e de fácil utilização. Use esta classe para configurar alertas com a mensagem que você deseja exibir e as ações a serem escolhidas pelo usuário. Após configurar o `UIAlertController` com as ações e o estilo desejado, você pode apresentá-lo usando o método `present(_:animated:completion:)` através da instância do _View Controller_ atual. O UIKit exibe os alertas como um modal sobre o conteúdo do seu aplicativo.

Como visto acima, além de exibir uma mensagem para um usuário, você pode associar ações ao seu `UIAlertController` para fornecer ao usuário uma maneira de responder. Para cada ação que você adiciona usando o método `addAction(_:)`, o `UIAlertController` configura um botão com os detalhes da ação. Quando o usuário toca nesse botão, o controlador de alerta executa o bloco que você forneceu ao criar o objeto de ação. O código abaixo ilustra a utilização:

``` swift
    @IBAction func botaoEntrarPressionado(_ sender: UIButton) {
        let alert = UIAlertController(title: "Estamos prontos", message: "Vamos começar a desenvolver nossas primeiras funcionalidades", preferredStyle: .alert)
        
        alert.addAction(UIAlertAction(title: "Ok", style: .default, handler: nil))
        
        self.present(alert, animated: true, completion: nil)
    }
```

Com o código acima, ao clicar em um botão _Entrar_, a aplicação exibe um alerta com o título e a mensagem informados, além de um botão para a ação adicionada. A `UIAlertAction` adicionada está configurada com um texto de confirmação e estilo padrão, além de um `handler` nulo (_`nil`_), o que indica que nenhuma função foi fornecida para que algum código fosse executado quando da seleção do usuário. Dessa forma, ao clicar no botão _Ok_ o alert simplesmente é fechado.

<p align="center">
<img alt="um gif animado com a demonstração do uso de um alerta simples pelo usuário" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-alertas-demo-alerta-simples.gif?raw=true" width="320" />
</p>
