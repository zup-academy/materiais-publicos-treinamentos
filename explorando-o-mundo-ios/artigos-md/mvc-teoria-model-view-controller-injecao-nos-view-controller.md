# Injeção de Dependências via Construtores ou Propriedades nos View Controllers

Este material teórico complementa a leitura do anterior em [Model-View-Controller (MVC)](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/artigos-md/mvc-teoria-model-view-controller.md) de modo a aprimorar a implementação da aplicação exemplo utilizada do jogo de dados.

## Entendendo a necessidade da aplicação do padrão

Na leitura anterior cujo link segue logo acima, você deve ter reparado que a aplicação, a partir da separação de módulos por camada, contou com uma dependência explícita do controlador para o modelo - existe uma nota destacando o assunto.

``` swift
class JogoDeDadosViewController: UIViewController {

    // dependências da camada de visualização
    @IBOutlet weak var valorAnteriorLabel: UILabel!
    @IBOutlet weak var valorAtualLabel: UILabel!
    
    // dependência da camada de modelo
    var jogo: JogoDeDados = .init() { // ou var jogo = JogoDeDados()
        didSet {
            atualizaView(para: jogo)
        }
    }

    // demais implementações omitidas
}
```

Conforme as diretrizes para o design de uma aplicação MVC pela visão da plataforma iOS, temos que um objeto de controlador não deve depender de um objeto do modelo. No entanto a própria documentação destaca - _"although, like views, this may be necessary if it's a custom controller class"_ - indicando que pode ser difícil dada a natureza customizável de alguns controladores. Nosso controlador `JogoDeDadosViewController` tem por responsabilidade coordenar um fluxo de ações base para o negócio, cheio de detalhes específicos e por natureza indica a dependência do modelo `JogoDeDados`. Assim, pela implementação anterior, que focava em favorecer a separação de camadas, resolvemos esta relação com `var jogo: JogoDeDados = .init()`.

Por mais necessária que se faça a dependência, e que portanto, a diretriz para esse caso possa ser desconsiderava, é importante destacar que existem possíveis problemas na implementação anterior. Na atual situação, nossa classe não apenas depende do modelo, como também é responsável por satisfazer a própria dependência. Não temos como fugir do acoplamento entre os módulos, mas devemos evitar que esse acoplamento traga ainda mais risco para a implementação.

Podemos destacar dois aspectos principais pelos quais a abordagem de resolução local da dependência deveria ser evitada:

* Acopla o controlador aos detalhes sobre a inicialização de uma instância do modelo: Caso o modelo tenha sua interface de inicialização alterada o código do controlador quebra;
* Torna o controlador menos flexível e testável: Não existe uma forma de substituir a instância de `JogoDeDados` utilizada durante o ciclo de vida da tela, seja para favorecer que por razões de fluxo de utilização se possa receber uma instância de um controlador anterior (por exemplo, que configuraria uma partida com número de turno e jogadores, numa evolução do domínio), seja para favorecer a utilização de mocks caso seja necessário testar o comportamento em isolamento.

A aplicação do design pattern de injeção de dependências pode ajudar a implementação do controlador.

## Injeção da Dependência e os inicializadores de um `UIViewController`

> Nota: Ainda que entrar em detalhes sobre o design pattern em si não seja o foco deste material teórico, mas sim sua aplicação como solução para o problema mencionado no código do controlador, caso necessite de uma revisão sobre o tema, você pode encontrar informações úteis clicando [aqui](https://pt.wikipedia.org/wiki/Inje%C3%A7%C3%A3o_de_depend%C3%AAncia), [aqui](https://www.youtube.com/watch?v=evhskJG1kvY) e [aqui](https://en.wikipedia.org/wiki/Dependency_injection).

Como a seção anterior destaca, a resolução local da dependência representa um possível problema. Ao invés de permitir que o código que inicializa o modelo seja parte da implementação do controlador podemos seguir por um caminho inverso: delegar a responsabilidade de inicializar e injetar a dependência para algum módulo acima do controlador. Dessa forma, continuamos dependentes, mas com um acoplamento mais baixo. 

Vamos implementar inicialmente a resolução através da estratégia _constructor injection_, onde pelo inicializador do _View Controller_ informaremos sobre a necessidade da injeção do `JogoDeDados`.

``` swift
class JogoDeDadosViewController: UIViewController {

    @IBOutlet weak var valorAnteriorLabel: UILabel!
    @IBOutlet weak var valorAtualLabel: UILabel!
    
    var jogo: JogoDeDados {
        didSet {
            atualizaView(para: jogo)
        }
    }
    
    init(jogo: JogoDeDados) { // é necessário a injeção na construção do objeto controlador
        self.jogo = jogo
        super.init(nibName: nil, bundle: nil)
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        atualizaView(para: jogo)
    }

    // demais implementações omitidas
}
```

Com o código acima, incorremos em um erro de compilação. `'required' initializer 'init(coder:)' must be provided by subclass of 'UIViewController'`

O erro informa que ao indicar um inicializador específico para o controlador - com `init(jogo:)` - sobrescrevendo a forma padrão de inicialização para o controlador, somos obrigados a implementar uma forma adicional, requerida por decisão do framework, de inicialização -  `required init?(coder: NSCoder)`.

Ao recorrer à documentação não temos muita informação adicional sobre a forma de construção, mas a saber, algo importante para a implementação atual, este é o inicializador utilizado pelo framework para criar o controlador cuja representação é definida pelo `Main.storyboard`. Ou seja, o inicializador idealizado para indicar a injeção da dependência, por padrão, não será utilizado, a menos que alteramos a forma como a aplicação é carregada, rompendo com algumas decisões do projeto com a _view_ baseada em _storyboards_ e invocando o _init_ adequado à mão.

Outro ponto é que a assinatura do inicializador, como você já deve imaginar, não suporta alterações, caso você esteja se questionando sobre alternativas. Tentar se adaptar ao seu uso tende a nos levar de volta ao problema anterior:

``` swift
    init(jogo: JogoDeDados) {
        self.jogo = jogo
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        self.jogo = JogoDeDados() // 😫
        super.init(coder: coder)
    }
```

## Injeção da Dependência via propriedades de _View Controllers_

Como já sabemos, os projetos deste treino utilizam as decisões do framework de UIKit e seus _storyboards_ como base para a implementação da camada de visualização. Sendo assim, precisamos procurar uma forma alternativa de trabalhar questões como a apresentada acima. Com o framework não sendo muito convidativo para o uso da estratégia de _constructor injection_, seguiremos então para outra: _setter injection_ (que pode ser inferido a partir das _properties_ do Swift).

O primeiro passo nesse sentido é remover os inicializadores idealizados na seção anterior. Mas isso de antemão já nos traz outro problema. Com a declaração da propriedade armazenada sendo feita atraves do _statement_ `var jogo: JogoDeDados`, o compilador exerce seu controle sobre a segurança da implementação e nos impede de obter um build para o aplicativo. O motivo já é conhecido: é preciso garantir que a propriedade seja inicializada de alguma forma, evitando que o objeto inicie seu ciclo de vida com `nil` para esta referência.

O problema já indica um caminho alternativo. É possível trabalhar com um _optional_ de `JogoDeDados` como propriedade armazenada e indicar a injeção da dependência a partir do _setter_ para a propriedade.

``` swift
class JogoDeDadosViewController: UIViewController {

    @IBOutlet weak var valorAnteriorLabel: UILabel!
    @IBOutlet weak var valorAtualLabel: UILabel!
    
    var jogo: JogoDeDados? { // optional type
        didSet {
            atualizaView(para: jogo)
        }
    }

    // demais implementações omitidas
```

Importante notar que a partir desse ponto, a implementação restante sofre o impacto de lidar com a possível presença para o valor.

``` swift
class JogoDeDadosViewController: UIViewController {

    @IBOutlet weak var valorAnteriorLabel: UILabel!
    @IBOutlet weak var valorAtualLabel: UILabel!
    
    var jogo: JogoDeDados? {
        didSet {
            guard let jogo = jogo else { return } // optional binding
            atualizaView(para: jogo)
        }
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        if let jogo = jogo { // optinal binding
            atualizaView(para: jogo)
        }
    }

    @IBAction func botaoPlayPressionado(_ sender: UIButton) {
        jogo?.executa() // optional chaining
    }
    
    func atualizaView(para jogo: JogoDeDados) {
        valorAnteriorLabel.text = String(describing: jogo.valorDoTurnoAnterior)
        valorAtualLabel.text = String(describing: jogo.valorDoTurnoAtual)
    }
}
```

Agora conseguimos reduzir o problema para o acoplamento gerado pela dependência. O controlador apenas descreve o uso do módulo do qual depende, no entando ele obriga que outro módulo terceiro que responsabilize pela injeção.

## Injetando a dependência um nível acima

Ao ler sobre o design pattern de injeção de depencências é comum notar as referências à outros aspectos que rodeiam sua aplicação em geral no desenvolvimento de software. O desenvolvimento em uma série de grandes plataformas do mercado é por anos pautado por exemplo pelo uso de frameworks que tem como único trabalho inverter o controle sobre as dependências (e outras coisas) fornecendo contêineres gerenciadores do ciclo de vida de objetos e das injeções das dependências.

Ainda que sejam possíveis aplicações parecidas no desenvolvimento iOS, utilizando libs ou mesmo escrevendo código para gerenciar um conjunto de objetos e fornecer suas referências, elas não são tão presentes como em outras plataformas. Assim, nos projetos deste treino, vamos tirar proveito das vantagens para a implementação dos módulos clientes (que dependem de certos objetos, como o nosso controlador) mas não nos preocuparemos muito com módulos injetores. No geral, vamos delegar para um módulo acima a responsabilidade de prover a dependência, mantendo a solução simples o suficiente.

No contexto do nosso projeto de exemplo, como aplicação de única tela, pode não ser tão visível qual é o módulo acima do controlador na hierarquia. Mas conforme já visto em [Entendendo _View Controllers_](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/artigos-md/primeiros-comportamentos-teoria-view-controllers.md) anteriormente, existe um módulo responsável por determinar como a aplicação inicializa as cenas que serão dispostas na janela do aplicativo: o `SceneDelegate`. A implementação deste módulo é o ponto mais próximo que conseguimos chegar atualmente da instância do View Controller, já após sua inicialização. Podemos adicionar código à função `scene(_:willConnectTo:options:)` para acessar a referência em `rootViewController` e efetivar a injeção da dependência.

``` swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        // Use this method to optionally configure and attach the UIWindow `window` to the provided UIWindowScene `scene`.
        // If using a storyboard, the `window` property will automatically be initialized and attached to the scene.
        // This delegate does not imply the connecting scene or session are new (see `application:configurationForConnectingSceneSession` instead).
        guard let _ = (scene as? UIWindowScene) else { return }
        
        let jogo = JogoDeDados()
        
        let viewController = window!.rootViewController as! JogoDeDadosViewController
        viewController.jogo = jogo
    }
```

> Nota: Em situações de aplicativos mais complexos com múltiplas telas e fluxo de utilização baseado em navegações e outras estratégias, é comum que este tipo de código esteja presente em outro grupo de objetos que tem por responsabilidade conter o conhecimento sobre estes fluxos e o papel de cada view controller nele. Em situações ainda mais complexas, estes objetos podem se apoior no uso de outros módulos gerenciadores de dependências para garantir um boa separação de responsabilidades.

Por questões específicas de projetos como o do exemplo esse código atualmente apresenta um problema em tempo de execução. O fluxo de atualização de views, disparado automaticamente pela implementação do observador do modelo no view controller, está sendo executado. Ao injetar a dependência em `viewController.jogo = jogo` disparamos sua execução. Neste momento, o controlador ainda **não tem** sua _view_ carregada e por sua vez suas conexões satisfeitas. O fluxo de atualização de views tenta acessar referências nulas e então temos um _crash_.

Para evitar situações como a que segue, é possível utilizar a propriedade `isViewLoaded` de `UIViewController` para evitar a execução anterior ao momento do carregamento da _view_.

``` swift
    var jogo: JogoDeDados? {
        didSet {
            guard isViewLoaded, let jogo = jogo else { return }
            atualizaView(para: jogo)
        }
    }
```

Com a adição desta verificação adicional, temos nossos requisitos satisfeitos com uma gestão mais adequado do acoplamento entre os módulos.
