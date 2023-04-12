# Documentando a API de módulos reutilizáveis

Ao desenvolver aplicações utilizando View Code trocamos o suporte do Interface Builder para conter a representação da view com todas suas características de layout, por escrever tudo em código. Assim, é comum que o volume de código para lidar com detalhes como, por exemplo, a adição de constraints de Auto Layout aumente consideravelmente. 

Embora as APIs do UIKit framework ofereçam abstrações em diferentes níveis para facilitar a escrita, em alguns cenários pode ser interessante a utilização de técnicas adicionais. Este material apresenta uma proposta de API para simplificar a adição de constraints. No caminho até o final aproveitaremos para conhecer maneiras de se documentar a API de módulos reutilizáveis com o Quick Help.

## Contextualizando

O material teórico que aborda a criação de um projeto com View Code já endereça uma série de pontos sobre boas práticas de implementação, desde o uso de propriedades com carregamento postergado à utilização de padrões de projeto para ajudam a organizar o código de um componente View Code. No entanto ainda é possível que o código destes componentes cresçam consideravelmente acompanhando a curva de complexidade das UIs.

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

O código acima nos traz novamente a ideia de como ficaria o código de um componente utilizando os conceitos utilizados até aqui. 

Analisando o código é possível notar que, entre os diferentes tipos de responsabilidades contextualizadas, uma delas se destaca pelo volume. Mesmo com apenas dois componentes necessários para atingir o layout de exemplo, foi necessário uma boa quantidade de linhas de código para descrever como os elementos do design devem se comportar em relação ao Auto Layout.

Por mais interessante que a API de `NSLayoutConstraint`, com suas âncoras de layout, possa parecer (e comparado a soluções anteriores ela é), podemos considerar que ela ainda proporciona um código bastante verboso. Para cada constraint necessária, é preciso que se invoque o método `constraint(equalTo:)`, por exemplo, para a restrição seja inicializada com seus detalhes.

Como estudado em _Organizando componentes do layout_ - seção deste treino que trata do Auto Layout - lembramos que para que uma _constraint_ seja construída, é preciso que exista a relação entre as propriedades (âncoras) de até dois elementos, além de detalhes adicionais como a constante e/ou um fator multiplicador que atuam sobre o deslocamento base dos elementos.

Essa rápida revisão da anatomia de _constraints_ também nos traz a percepção adicional de complexidade para uma simples linha de código: `greetingLabel.trailingAnchor.constraint(equalTo: containerView.trailingAnchor, constant: -24)`. Com a escalada da complexidade para UIs das aplicações é bastante previsível o aumento da carga cognitiva exercida por códigos como este.

## Analisando alternativas

Como citado na seção acima, a API utilizada anteriormente pode nos levar a implementações mais difíceis de se ler e manter em alguns casos. É necessário que algo seja feito para reduzir essa pressão exercida sobre nossa implementação. 

Por um bom ponto de partida podemos citar a adoção da utilização de componentes de alto nível como os _Stack Views_ sempre que possível. Através de sua utilização, reduzimos em muito a quantidade de _constraints_ de layout que desenvolvedores e desenvolvedoras tem que gerenciar por si mesmos, já que o próprio componente as adiciona e controla automaticamente.

Entretanto, _Stack Views_ representam bem sequências de _views_ organizadas em sequência. Embora possam ter sua aplicação numa vasta gama de layouts diferentes de maior complexidade, ainda vão existir uma série de situações onde serão necessárias as aplicações manuais das restrições de layout, seja por uma quebra nessa ideia sequencial de layout, seja por uma necessidade maior de controle às referências das constraints em tempo de execução.

É ainda necessário que tenhamos uma maneira clara codificar sobre a gestão das _constraints_.

### Uma extensão para adicionar funções de layout utilitárias 

Uma série de diferentes abordagens poderiam ser mencionadas. Desde a criação de classes especializadas ou outros componentes de alto nível, até a adoção de _frameworks_ existentes no mercado, mas neste material vamos manter uma solução que preza pela simplicidade. _Constraints_ de layout são aplicáveis a uma (e qualquer uma) _view_, portanto, uma _extension_ que adicione a elas capacidades neste sentido parece suficiente.

Nosso objetivo é obtermos funções simples, com nomenclatura que expressem de forma clara suas intenções, e que de quebra escondem os detalhes de implementação de `NSLayoutConstraint` evitando uma legibilidade mais custosa e maior propensão a erros lógicos na escrita.

Podemos supor a criação de funções para as mais variadas situações onde já tenhamos lidado com a adição de uma _constraint_. Neste momento, você pode fazer um exercício mental para se lembrar das mais diferentes maneiras e até mesmo tentar encontrar padrões entre elas. 

Podemos refletir por exemplo sobre as diferentes _constraints_ que podemos aplicar ao tentar fazer com que uma _view_ seja contida dentro dos limites de uma segunda, provavelmente em nível mais alto da hierarquia.

``` swift
//
//  LayoutHelpers+UIView.swift
//

extension UIView {

    func constrainToEdges(of view: UIView) {
        let constraints = [
            self.topAnchor.constraint(equalTo: view.topAnchor),
            self.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            self.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            self.trailingAnchor.constraint(equalTo: view.trailingAnchor),
        ]
        
        NSLayoutConstraint.activate(constraints)
    }

}

```

Ao exemplo de código acima, podemos ainda adicionar capacidades para definir espaçamentos internos, como margens, em qualquer nível de detalhamento. Por exemplo, fazendo uma distinção entre margens aplicadas às âncoras do eixo _x_ e _y_.

``` swift
    func constrainToEdges(of view: UIView,
                     verticalMargins: CGFloat = .zero,
                     horizontalMargins: CGFloat = .zero) {
        let constraints = [
            self.topAnchor.constraint(equalTo: view.topAnchor, constant: verticalMargins),
            self.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: horizontalMargins),
            self.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -verticalMargins),
            self.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -horizontalMargins),
        ]
        
        NSLayoutConstraint.activate(constraints)
    }
```

Com o exemplo anterior já podemos entender a que nível de suporte somos atendidos por esta API ao supor uma aplicação.

``` swift
    // código anterior omitido

    func addLayoutConstraints() {
        containerView.constrainToEdges(of: view)
    }
}
```

Ou ainda, antecipando a correção de um possível bug, contando com uma função especializada na adição de constraints às âncoras de _layout_ do _layout guide_ de _safe area_:

``` swift
    func constrainToEdges(of view: UIView,
                          verticalMargins: CGFloat = .zero,
                          horizontalMargins: CGFloat = .zero) {
        let constraints = [
            self.topAnchor.constraint(equalTo: view.topAnchor, constant: verticalMargins),
            self.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: horizontalMargins),
            self.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -verticalMargins),
            self.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -horizontalMargins),
        ]
        
        NSLayoutConstraint.activate(constraints)
    }

    func constrainToSafeEdges(of view: UIView,
                              verticalMargins: CGFloat = 0,
                              horizontalMargins: CGFloat = 0) {
        let constraints = [
            self.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: verticalMargins),
            self.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor, constant: horizontalMargins),
            self.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -verticalMargins),
            self.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor, constant: -horizontalMargins),
        ]
        
        NSLayoutConstraint.activate(constraints)
    }
```

``` swift
    // código anterior omitido

    func addLayoutConstraints() {
        containerView.constrainToSafeEdges(of: view)
    }
}
```

Só até aqui já é possível perceber como ganhamos bastante com essa troca. Isolamos todo o detalhe de implementação de `NSLayoutConstraint` nessa _extension_ e oferecemos aos desenvolvedores uma maneira sucinta de se obter o resultado esperado nestas situações. Além disso, erros comuns como executar a função a partir de outra referência de âncora, inverter âncoras entre possíveis elementos ou esquecer do detalhe de utilizar o operador `-` para casos de âncoras específicas do sistema de coordenadas de layout, agora são facilmente evitados.

Indo além, podemos refletir sobre mais formas de _constraints_ para outras situações específicas. Em certos momentos, precisamos posicionar um elemento em relação a outro, seja logo abaixo, acima ou ao centro, noutros ainda dimensionar o elemento dada uma constante ou relação com outro elemento. Para todos os casos, podemos prever uma implementação de mesma natureza.

``` swift
    @discardableResult
    func anchorBelow(of view: UIView, withMargin margin: CGFloat = .zero) -> NSLayoutConstraint {
        let constraint = self.topAnchor.constraint(equalTo: view.bottomAnchor, constant: margin)
        constraint.isActive = true
        return constraint
    }

    func anchorToCenter(of view: UIView, withOffset point: CGPoint = .zero) {
        let constraints = [
            self.centerXAnchor.constraint(equalTo: view.centerXAnchor, constant: point.x),
            self.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: point.y)
        ]
        
        NSLayoutConstraint.activate(constraints)
    }

    @discardableResult
    func constrainHeight(to constant: CGFloat) -> NSLayoutConstraint {
        let constraint = self.heightAnchor.constraint(equalToConstant: constant)
        constraint.isActive = true
        return constraint
    }

    func constrainSize(to size: CGSize) {
        let constraints = [
            self.widthAnchor.constraint(equalToConstant: size.width),
            self.heightAnchor.constraint(equalToConstant: size.height)
        ]
        
        NSLayoutConstraint.activate(constraints)
    }
```

A quantidade de funções utilitárias acompanha diretamente a quantidade de situações diferentes que o dia-a-dia escrevendo layouts exigir.

> Nota: Ao final deste material teórico você encontra uma implementação sugerida que adiciona mais situações de layout possíveis.

## Facilitando o uso da API utilitária

Até este momento caminhamos no sentido de escrever e refatorar código buscando um melhor implementação. No entanto, existe um esforço de trabalho não-funcional que também pode adicionar muito à API de módulos tão reutilizáveis quanto o último.

Imagine a situação hipotética de se receber uma nova pessoa no time de desenvolvimento de um projeto que funciona com a solução proposta até aqui. Naturalmente, podemos designar alguém da equipe para auxiliar o momento de entrada, apresentar o contexto da aplicação e seus principais pontos, mas e quando essa pessoa for interagir com o código dessa _extension_? Será que ela terá a mesma visão que construímos até agora? Com quanto esforço ela conseguirá construir isso?

Uma boa saída para estas situações é oferecer documentação interativa para suas APIs, assim todo o contexto que poderia passar implicitamente pela interpretação de uma terceira pessoa pode ser adicionado de forma bastante pontual.

### Utilizando o Quick Help do XCode

Escrever documentação pode ser algo não muito atraente para a maior parte das pessoas desenvolvedores. Independente do quanto você concorda ou não com essa última afirmação os desenvolvedores das plataformas de desenvolvimento sabem disso. Por isso eles trabalham para prover formas o mais automatizadas possível para que você consiga atingir o resultado com o menor esforço, e também para tornar o resultado além de útil, agradável, com uma experiência interativa.

A documentação com o Quick Help é constituída de notas objetivas com suporte à notação Markdown que podem ser adicionadas a tipos, propriedades, funções, _enum cases_ e praticamente qualquer outro membro no seu código. Depois que a documentação é adicionada aos módulos, o Xcode pode exibir as notas com o conteúdo informativo.

O Xcode pode apresentar o conteúdo da documentação com o Quick Help através de um _tooltip_ ao clicar com o direito sobre um membro e selecionar _Show Quick Help_, ou com o atalho _Option ⌥_ + clique equivalente.

<p align="center">
<img alt="Imagem com tooltip exibindo a documentação interativa do Quick Help" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/viewcode-teoria-documentacao-quickhelp-tooltip.png?raw=true" width="70%" />
</p>

O Xcode também pode apresentar o conteúdo da documentação com o Quick Help através do Quick Help Inspector, acessível pelo painer lateral direito, ou através do atalho _Option ⌥ + Command ⌘ + 3_.

<p align="center">
<img alt="Imagem com o inspector exibindo a documentação interativa do Quick Help" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/viewcode-teoria-documentacao-quickhelp-inspector.png?raw=true" width="30%" />
</p>

#### Usando notações Quick Help e _callouts_

Já tivemos uma mostra sobre as capacidades da documentação interativa acima, nos resta agora entender como habilitar o recurso. Usando uma notação iniciada por `///` é possível adicionar o conteúdo informativo que será utilizado para documentar pontos específicos da implementação.

``` swift
/// An extension to add some helper functions that provide layout capabilities in a clealy way.
extension UIView {
    
    /// An utility to directly access the `safeAreaLayoutGuide` anchors.
    var safeTopAnchor: NSLayoutYAxisAnchor {
        return safeAreaLayoutGuide.topAnchor
    }

    // codigo omitido
```

É possível ainda utilizar mais recursos da notação para que seja atingida uma documentação organizada com separação adequada de seções, como foi possível perceber nas imagens de exemplo acima. Através dos chamados _callouts_ podemos agrupar informações acerca da descrição de um membro, dos exemplos de utilização em determinada situação de código, e dependendo da natureza do membro, usando como exemplo um método, é possível ainda agrupar informações sobre os parâmetros, tipo de retorno e informações sobre lançamento de erros.

``` swift
    /// Positions your view code component centering on the target view
    ///
    /// - Parameters:
    ///     - view: The target view to anchor centering.
    ///     - point: The point containing the x and y axis offset to apply as the constant of the centerX and centerY anchor constraints.
    /// - Returns: The activated layout constraints reference.
    /// - Example: Passing offset to `anchorToCenter(of:withOffset:)`.
    ///
    ///         let point = Point(x: 10, y: 10)
    ///         view.anchorToCenter(of: anotherView, withOffset: point)
    ///
    @discardableResult
    func anchorToCenter(of view: UIView, withOffset point: CGPoint = .zero) -> [NSLayoutConstraint]  {
        let constraints = [
            self.centerXAnchor.constraint(equalTo: view.centerXAnchor, constant: point.x),
            self.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: point.y)
        ]
        
        NSLayoutConstraint.activate(constraints)
        return constraints
    }
```

O código acima utiliza a notação e seus _callouts_ para obter o resultado que vimos previamente. Navegando pelo código da biblioteca padrão da Swift, é possível perceber como os próprios desenvolvedores utilizam seus recursos de maneira estensiva para obter uma API bastante clara para os futuros desenvolvedores que tiverem contato.

``` swift
/// A type representing an error value that can be thrown.
///
/// Any type that declares conformance to the `Error` protocol can be used to
/// represent an error in Swift's error handling system. Because the `Error`
/// protocol has no requirements of its own, you can declare conformance on
/// any custom type you create.
///
/// Using Enumerations as Errors
/// ============================
///
/// Swift's enumerations are well suited to represent simple errors. Create an
/// enumeration that conforms to the `Error` protocol with a case for each
/// possible error. If there are additional details about the error that could
/// be helpful for recovery, use associated values to include that
/// information.
///
/// The following example shows an `IntParsingError` enumeration that captures
/// two different kinds of errors that can occur when parsing an integer from
/// a string: overflow, where the value represented by the string is too large
/// for the integer data type, and invalid input, where nonnumeric characters
/// are found within the input.
///
///     enum IntParsingError: Error {
///         case overflow
///         case invalidInput(Character)
///     }
///
/// The `invalidInput` case includes the invalid character as an associated
/// value.
///
/// The next code sample shows a possible extension to the `Int` type that
/// parses the integer value of a `String` instance, throwing an error when
/// there is a problem during parsing.
///
///     extension Int {
///         init(validating input: String) throws {
///             // ...
///             let c = _nextCharacter(from: input)
///             if !_isValid(c) {
///                 throw IntParsingError.invalidInput(c)
///             }
///             // ...
///         }
///     }
///
/// When calling the new `Int` initializer within a `do` statement, you can use
/// pattern matching to match specific cases of your custom error type and
/// access their associated values, as in the example below.
///
///     do {
///         let price = try Int(validating: "$100")
///     } catch IntParsingError.invalidInput(let invalid) {
///         print("Invalid character: '\(invalid)'")
///     } catch IntParsingError.overflow {
///         print("Overflow error")
///     } catch {
///         print("Other error")
///     }
///     // Prints "Invalid character: '$'"
///
/// Including More Data in Errors
/// =============================
///
/// Sometimes you may want different error states to include the same common
/// data, such as the position in a file or some of your application's state.
/// When you do, use a structure to represent errors. The following example
/// uses a structure to represent an error when parsing an XML document,
/// including the line and column numbers where the error occurred:
///
///     struct XMLParsingError: Error {
///         enum ErrorKind {
///             case invalidCharacter
///             case mismatchedTag
///             case internalError
///         }
///
///         let line: Int
///         let column: Int
///         let kind: ErrorKind
///     }
///
///     func parse(_ source: String) throws -> XMLDoc {
///         // ...
///         throw XMLParsingError(line: 19, column: 5, kind: .mismatchedTag)
///         // ...
///     }
///
/// Once again, use pattern matching to conditionally catch errors. Here's how
/// you can catch any `XMLParsingError` errors thrown by the `parse(_:)`
/// function:
///
///     do {
///         let xmlDoc = try parse(myXMLData)
///     } catch let e as XMLParsingError {
///         print("Parsing error: \(e.kind) [\(e.line):\(e.column)]")
///     } catch {
///         print("Other error: \(error)")
///     }
///     // Prints "Parsing error: mismatchedTag [19:5]"
public protocol Error : Sendable {
}
```

O trecho de código acima é extraído do módulo `Swift.Misc` e exemplifica o ponto mencionado. É possível perceber a riqueza de detalhes da documentação oferecida. É uma ótima prática sua utilização em módulos altamente reutilizáveis dentro de um projeto, ou mesmo através de vários.

> Nota: Como mencionado em nota anterior, é possível obter implementação de apoio para o layout através do link abaixo:
>
>   [Gist - LayoutHelpers+UIView.swift](https://gist.github.com/rafaelrollozup/8a029d53fbc4ce4525a0b35e40d91431)
