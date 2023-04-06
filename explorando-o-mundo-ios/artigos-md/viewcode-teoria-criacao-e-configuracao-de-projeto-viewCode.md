# Criação e configuração de projeto ViewCode

Ao criar projetos XCode para aplicativos iOS por padrão é requerido que indiquemos nossa escolha para a abordagem quanto à tecnologia de visualização. Entre as opções estão `SwiftUI`, um padrão mais recente para a construção de interfaces, e `Storyboard`, padrão seguido já há alguns anos na plataforma.

<p align="center">
<img alt="Imagem com o modal de configuração de um projeto XCode para aplicativos iOS" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/viewcode-teoria-configuracao-viewcode-imagem-modal-criacao-projetos.png?raw=true" width="70%" />
</p>

Como é possível perceber não há nenhuma opção para um template de projeto que use a prática de View Code. Poderíamos imaginar que a seleção de `SwiftUI` fosse a adequada. No entanto, mesmo que com esta tecnologia seja possível descrever nossa _view_ através de código, ela representa uma abordagem bastante diferente do que vimos até aqui e se encontra fora do escopo deste treino. Com isso, `Storyboard` ainda é a seleção mais adequada para nós.

## Removendo referências ao uso de Storyboards e entendendo as consequências

Você deve se lembrar das inúmeras vezes onde um novo projeto (Storyboard) foi construído. A partir da criação, bastave que se fizesse um build e executasse a aplicação através de um simulador para que tivéssemos nossa _view_ sendo apresentada - até então em branco.

Mesmo sem conteúdo adicionado, o que temos visível nessa situação é justamente o conteúdo da _view_ de um controlador, cuja representação se encontra no arquivo `Main.storyboard`. A esta altura você já sabe isso de cor, nada de novo aqui. No entanto, isto posto, podemos relembrar sobre todo o conhecimento da estrutura de uma aplicação UIKit para entender o que faz com que o conteúdo deste arquivo seja carregado automaticamente.

<p align="center">
<img alt="Imagem com o aplicativo recém criado com sua tela em branco na primeira execução" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/viewcode-teoria-configuracao-viewcode-imagem-app-storyboard-em-branco.png?raw=true" width="30%" />
</p>

Recorrendo ao conhecimento prévio sobre os pontos de código mais próximos da inicialização da aplicação, temos os métodos nos arquivos `AppDelegate.swift` e `SceneDelegate.swift`, que oferecem meios para responder a respeito dos ciclos de vida da aplicação e a eventos do sistema, e a eventos de ciclo de vida que ocorrem no contexto de uma cena, respectivamente. 

Para o nosso propósito nesta explicação, um método já conhecido do arquivo `SceneDelegate.swift` aponta o momento exato onde é mencionado o carregamento automático - e onde se pode inserir código para fazê-lo manualmente intervindo sobre o uso de um Storyboard.

``` swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        // Use this method to optionally configure and attach the UIWindow `window` to the provided UIWindowScene `scene`.
        // If using a storyboard, the `window` property will automatically be initialized and attached to the scene.
        // This delegate does not imply the connecting scene or session are new (see `application:configurationForConnectingSceneSession` instead).
        guard let _ = (scene as? UIWindowScene) else { return }
    }

    // outros métodos
}
```

Duas partes importantes do comentário ressaltam esse sentido: _"If using a storyboard, the `window` property will automatically be initialized and attached to the scene"_, e _"Use this method to optionally configure and attach the UIWindow `window` to the provided UIWindowScene `scene`"_. As afirmações dão conta respectivamente de que (1) caso utilizando um storyboard a propriedade `window` será automaticamente inicializada e adicionada à _window scene_ (implicitamente com o conteúdo do `Main.storyboard` e seu _root view controller_, já voltaremos a isso) e (2) é possível opcionalmente utilizar este espaço para fazer isso através de código.

Por partes, vamos começar pelo item primeiro.

### Referências ao `Main.storyboard`

Você deve ter notado a menção sobre a implicitude do carregamento da window levar em conta o conteúdo do `Main.storyboard`. Embora isso possa ser verdade considerando apenas o comentário no método local, não é bem ao certo verdade ao se considerar o projeto como um todo. Já pensou sobre como o projeto sabe que um arquivo de nome, específicamente, `Main.storyboard` precisa ser carregado do _bundle_ da aplicação? Colocando em prática nossa capacidade de dedução, podemos supor que esta descrição se encontra em algum ponto do projeto.

#### O Info.plist

Como uma série de outras informações descritivas sobre o projeto, a que procuramos vive no conteúdo do arquivo `Info.plist`. Como o próprio nome nos deixa supor, ele é apenas um arquivo com uma lista de propriedades, contento informações gerais sobre o projeto.

Acessando o conteúdo deste arquivo conseguimos localizar duas chaves principais de valores relacionados com a configuração padrão que aponta para o storyboard. São elas _"Main storyboard file base name"_ e _"Application Scene Manifest > Scene Configuration > Application Session Role > Item 0 (Default Configuration) > Storyboard Name"_.

> Nota: Caso você esteja realizando este treino com uma versão de XCode igual a 14 (ou mais recente) é possível que não encontre as chaves citadas dentro do Info.plist. Neste caso, você pode localicar as mesmas informação abaixo da seção _Info > Custom iOS Target Properties_, ao clicar item do projeto base no _Project navigator_.

Ambas as informações para estas chaves devem ser removidas para que seja possível a configuração para View Code, uma vez que, ao não encontrar o arquivo a aplicação encerraria o processo em erro duranto o _startup_ da aplicação.

#### Carregando a `window` manualmente

Com as referidas remoções, o próximo passo é focado na segunda etapa mencionada acima: utilizar o método `scene(_:willConnectTo:options:)` para restaurar o estado funcional da aplicação com o carregamento da `window` manualmente.

O método `scene(_:willConnectTo:options:)` através de um parâmetro, já adiciona ao seu contexto de execução o objeto relativo à _window scene_, mencionada nos comentários. Na implementação padrão, do método existe apenas um _optional binding_ fazendo a checagem de não-nulidade para o mesmo, mas podemos adiciona contexto na sequência para termos nosso objetivo alcançado. Podemos obter as _bounds_ para o device e construir a `window` - o que era feito automaticamente.

``` swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        // Use this method to optionally configure and attach the UIWindow `window` to the provided UIWindowScene `scene`.
        // If using a storyboard, the `window` property will automatically be initialized and attached to the scene.
        // This delegate does not imply the connecting scene or session are new (see `application:configurationForConnectingSceneSession` instead).
        guard let scene = (scene as? UIWindowScene) else { return }
        
        let window = UIWindow(frame: scene.coordinateSpace.bounds)
        window.windowScene = scene
        self.window = window
    }
    // outros métodos
}
```

> Nota: Neste momento você já poderia se livrar do arquivo `Main.storyboard`. Ele já não mais será necessário.

Caso você tente essa sequência de passos até aqui perceberá que algo de errado ainda acontece.

<p align="center">
<img alt="Imagem com o aplicativo configurado através de código, porém com a tela preta, indicando que nenhuma janela foi apresentada corretamente" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/viewcode-teoria-configuracao-viewcode-imagem-app-viewcode-tela-preta.png?raw=true" width="30%" />
</p>

Isso ocorre porque até o momento nossa configuração apenas indicou a construção de uma `window` com seus limites definidos. O código que roda automaticamente quando da utilização de Storyboards vai além disso. Ele configura o controlador principal indicado pelo Storyboard (_Root View Controller_) como `rootViewController` da `window` construída.

``` swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        // Use this method to optionally configure and attach the UIWindow `window` to the provided UIWindowScene `scene`.
        // If using a storyboard, the `window` property will automatically be initialized and attached to the scene.
        // This delegate does not imply the connecting scene or session are new (see `application:configurationForConnectingSceneSession` instead).
        guard let scene = (scene as? UIWindowScene) else { return }
        
        let window = UIWindow(frame: scene.coordinateSpace.bounds)
        window.windowScene = scene
        self.window = window
        
        window.rootViewController = ViewController() // controlador base do template de projeto
        window.makeKeyAndVisible()
    }

    // outros métodos
}
```

O código adicionado ao _snippet_ acima é bastante simples. Sabemos que a implementação de base para o _controller_, cuja representação vivia no _storyboard_, é a presente na classe `ViewController`, provida pelo template do projeto. Desta forma, o código acima simplesmente cria um objeto de mesmo controlador e passa sua referência à `window` como _root view controller_ de toda a aplicação. A partir deste ponto, este controlador será o _entrypoint_ do fluxo da aplicação.

A invocação adicional apenas diz para que a `window` ganhe foco e se faça visível.

#### O que muda nos controladores

Mais uma vez, se você seguiu as etapas à risca até aqui, a aplicação continua ainda num estado inconsistente e apresentando uma tela preta.

Por mais que já conseguimos orientar qual o controlador principal e fazer com a que `window` se faça visível, precisamos nos perguntar outra coisa. Como será que uma representação storyboard de controlador era instanciada e tinha sua hierarquia de _views_ carregada e exibida na tela?

``` swift
class ViewController: UIViewController {
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
}
```

Você deve se lembrar de ter visto o código acima antes. É ele a chave para o entendimento do passo que ainda falta. Os controladores, quando com sua _view_ representados através de _storyboards_, tem sua instanciação realizada através do `required init?(coder: NSCoder)`. Este inicializador por padrão utiliza uma implementação de base que, em tempo de execução, interpreta o código de marcação do arquivo do _storyboard_, busca a representação do controlador, e instancia os objetos adequados para sua hierarquia de views. Todo esse último ponto não existe na nossa solução atual, afinal: que hierarquia de _views_? Neste momento temos puro código Swift.

``` swift
import UIKit

class ViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }
    
}

```

## Escrevendo com _View Code_

Com o final da seção acima, permanecemos com uma missão em aberto. Como representar a _view_ deste _View Controller_? Em qual momento fazer o carregamento dos objetos de `UIView` necessários?

A resposta para a primeira pergunta pode ser facilmente deduzida: através de código Swift. Em algum ponto do framework de UIKit existe código que orienta a criação de cada tipo de view da _object library_, precisamos neste momento escrevê-los por nós mesmos. A segunda pergunta pode não ser tão óbvia para um _view coder_ de primeira viagem, mas logo será. A resposta é o código do ciclo de vida dos controladores responsável exatamente por isso: `loadView()`.

``` swift
import UIKit

class ViewController: UIViewController {
    
    override func loadView() {
        super.loadView()
        self.view.backgroundColor = .systemBackground
        
        let view = UIView()
        view.translatesAutoresizingMaskIntoConstraints = false
    
        self.view.addSubview(view)
        
        NSLayoutConstraint.activate([
            view.leadingAnchor.constraint(equalTo: self.view.safeAreaLayoutGuide.leadingAnchor),
            view.topAnchor.constraint(equalTo: self.view.safeAreaLayoutGuide.topAnchor),
            view.trailingAnchor.constraint(equalTo: self.view.safeAreaLayoutGuide.trailingAnchor),
            view.bottomAnchor.constraint(equalTo: self.view.safeAreaLayoutGuide.bottomAnchor),
        ])
        
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.textAlignment = .center
        label.font = .systemFont(ofSize: 18, weight: .bold)
        label.text = "Hello World!"
        
        view.addSubview(label)
        
        NSLayoutConstraint.activate([
            label.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 24),
            label.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -24),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }
    
}
```

Através da implementação do método `loadView()` é possível utilizar as APIs dos módulos já conhecidos do UIKit framework, para que assim tenhamos nossas _views_ carregadas e adequadamente apresentadas na tela do aplicativo. Assim temos nossa primeira implementação com _View Code_.

<p align="center">
<img alt="Imagem com o aplicativo desenvolvido com View Code completamente funcional, apresentando um label com o conteúdo 'Hello World!' sobre o fundo branco" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/viewcode-teoria-configuracao-viewcode-imagem-app-funcional.png?raw=true" width="30%" />
</p>

> Nota: Implementar toda a _view_ com _View Code_ pode requerir bastante conhecimento sobre as APIs de diversas classes que até o momento utilizáva-mos apenas indiretamente pela abstração visual provida pelo Interface Builder. É muito recomendável que você estude bem as APIs dessas classes para que seja possível atingir o mesmo resultado que já é capaz através do Storyboard com código puro. Nessa troca de contexto você vai notar que nada de tão grave muda na abordagem. Essencialmente, apenas faremos escrevendo instruções Swift o que fazíamos através da manipulação de objetos com o suporte do Interface Builder.

### Possíveis ajustes de implementação

Você deve ter percebido que a implementação do controlador para um simples exemplo de _Hello World!_ com _View Code_ fez com que o código para o mesmo se estendesse bastante. Mesmo com uma composição super simples de objetos e constraints de Auto Layout o conteúdo do método `loadView()` disparou, e como deve estar presumindo, quanto mais nossa tela se tornar complexa, mais esse efeito tende a acontecer.

Essa é mesmo uma verdade, com a prática de _View Code_ trouxemos toda a parte do conteúdo que vivia nos `Storyboards` para um determinada tela para a implementação base em código Swift. Embora não exista muitos meios para que evitemos isso, podemos apostar em melhorar nossa implementação para que o código se torne mais gerenciável.

#### _lazy vars_ para a separação dos objetos de UIView em propriedades contextualizadas

``` swift
import UIKit

class ViewController: UIViewController {
    // MARK: - Subviews
    
    private var greetingLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.textAlignment = .center
        label.font = .systemFont(ofSize: 18, weight: .bold)
        label.text = "Hello World!"
        return label
    }()
    
    private var containerView: UIView = {
        let view = UIView()
        view.translatesAutoresizingMaskIntoConstraints = false
        return view
    }()
    
    // MARK: - Lifecycle
    override func loadView() {
        super.loadView()
        view.backgroundColor = .systemBackground
    
        view.addSubview(containerView)
        
        NSLayoutConstraint.activate([
            containerView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor),
            containerView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            containerView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor),
            containerView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor),
        ])
        
        view.addSubview(greetingLabel)
        
        NSLayoutConstraint.activate([
            greetingLabel.leadingAnchor.constraint(equalTo: containerView.leadingAnchor, constant: 24),
            greetingLabel.trailingAnchor.constraint(equalTo: containerView.trailingAnchor, constant: -24),
            greetingLabel.centerYAnchor.constraint(equalTo: containerView.centerYAnchor)
        ])
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }
    
}
```

Uma proposta de solução para o problema mencionado é separar as responsabilidades designadas para o método `loadView()` em partes menores. Começando por separar o objetos de `UIView` de que dependemos em propriedades para o _View Controller_. Essa dica é especialmente útil também para quando queremos controlar o estado destes objetos ao longo do ciclo de vida do controlador ou mesmo de outras _views_, além de ajudar a concentrar os objetos da hierarquia de views em um ponto específico do código.

Através de um bloco de código autoexecutável, podemos já configurar o objeto com as características desejáveis para o layout da tela.

``` swift
    // MARK: - Subviews
    
    private var greetingLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.textAlignment = .center
        label.font = .systemFont(ofSize: 18, weight: .bold)
        label.text = "Hello World!"
        return label
    }()
```

Um ponto de atenção sobre o exemplo de código acima é que ele pode esconder um problema de implementação. Da forma como definido o bloco que configura os objetos, a execução (instanciação) de cada um deles se dá quando da instanciação do próprio _View Controller_, o que no geral não é recomendável. O peso de instanciar os objetos deveria ser mantido mais próximo do ponto onde todo o carregamento da view e cálculo de layout se dá. Em outras palavras devemos levar essa construção para mais próximo do momento onde utilizamos de fato o objeto: `loadView()`.

Mas como fazer para que eu tenha a execução no momento do `loadView()` sem desfazer todo o trabalho e voltar este código para dentro do método?

A resposta é postergar deliberadamente o carregamento. Já sabemos que isso é possível através das _lazy properties_.

``` swift
class ViewController: UIViewController {
    // MARK: - Subviews

    private lazy var greetingLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.textAlignment = .center
        label.font = .systemFont(ofSize: 18, weight: .bold)
        label.text = "Hello World!"
        return label
    }()
    
    private lazy var containerView: UIView = {
        let view = UIView()
        view.translatesAutoresizingMaskIntoConstraints = false
        return view
    }()
    
    // resto do código omitido
```

#### _Template method_ para garantir uma distribuição coordenada

Com os últimos ajustes o código já tem uma melhora considerável, mas ainda é possível perceber a implementação de `loadView()` sobrecarregada. Se pensarmos a respeito das responsabilidaes autocontidas podemos chegar a (1) customização da aparência da root view, (2) construção da hierarquia de visualizações; e (3) adição das constraints de layout adequadas. É ainda muita coisa para um único método.

Podemos pensar na separação por estes grandes grupos de responsabilidades, mas com um cuidado, a ordem de execução entre a adição de uma _subview_ e a adição de constraints que possam relacionar esta _subview_ à outras _views_ deve ser respeitada, sob risco de erro em tempo de execução. Uma maneira de reduzir o problema é garantir de primeiro se faça toda a composição hierarquica e só depois de adicionem as constraints.

Pensando sobre o contexto acima e em uma possível solução podemos ser levados a um problema clássico. Quebrar um problema em partes menores e garantir uma ordem de execução coordenada é exatamente o problema clássico que o _design pattern_ _Template method_ resolve.

``` swift
protocol ViewCode {
    func customizeAppearance()
    func addSubviews()
    func addLayoutConstraints()
}

extension ViewCode {
    
    /// Orchestrates the ViewCode component configuration.
    /// A template method that performs a set of operations in a predefined sequence using the
    /// implementations provided by the class that conforms to the ViewCode protocol.
    ///
    func setup() {
        customizeAppearance()
        addSubviews()
        addLayoutConstraints()
    }
    
    func customizeAppearance() {
        // Nothing happens. Just to unenforce an implementation if you don't need one.
    }
    
    func addSubviews() {
        // Nothing happens. Just to unenforce an implementation if you don't need one.
    }
    
    func addLayoutConstraints() {
        // Nothing happens. Just to unenforce an implementation if you don't need one.
    }
    
}

class ViewController: UIViewController {
    // MARK: - Subviews

    private lazy var greetingLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.textAlignment = .center
        label.font = .systemFont(ofSize: 18, weight: .bold)
        label.text = "Hello World!"
        return label
    }()

    private lazy var containerView: UIView = {
        let view = UIView()
        view.translatesAutoresizingMaskIntoConstraints = false
        return view
    }()

    // MARK: - Lifecycle
    override func loadView() {
        super.loadView()
        setup()
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }

}

extension ViewController: ViewCode {
    
    func customizeAppearance() {
        view.backgroundColor = .systemBackground
    }
    
    func addSubviews() {
        view.addSubview(containerView)
        view.addSubview(greetingLabel)
    }
    
    func addLayoutConstraints() {
        NSLayoutConstraint.activate([
            containerView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor),
            containerView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            containerView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor),
            containerView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor),
        ])

        NSLayoutConstraint.activate([
            greetingLabel.leadingAnchor.constraint(equalTo: containerView.leadingAnchor, constant: 24),
            greetingLabel.trailingAnchor.constraint(equalTo: containerView.trailingAnchor, constant: -24),
            greetingLabel.centerYAnchor.constraint(equalTo: containerView.centerYAnchor)
        ])
    }
    
}
```

Através de um protocolo com os métodos relativos a cada parte do processo somada a uma extensão de protocolo que adiciona a implementação padrão (o _template method_ `setup()`) para o objeto que se conforme ao mesmo, temos uma solução elegante e confiável que torna o código bastante mais fácil de se entender.

## Conclusão

Este material apresenta uma forma de implementar uma aplicação com _View Code_ bastante eficaz para a maior parte dos problemas. Ainda assim, em alguns casos esta implementação ainda pode impor um carga considerável de código sobre módulos complexos não sendo assim incomum de perceber a adoção de outras técnicas de engenharia ou mesmo uso de _libs_ para facilitar o processo em alguns projetos. No entanto, tais usos estarão fora do contexto das implementações deste treino. Caso tenha se interessado, é recomendável pesquisar sobre o tema na comunidade.
