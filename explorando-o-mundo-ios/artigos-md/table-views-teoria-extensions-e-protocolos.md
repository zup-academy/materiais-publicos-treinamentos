# Mais Swift: Protocolos e Extensions

> Esse material teórico foi adaptado das fontes originais disponíveis em: 
>   * https://docs.swift.org/swift-book/LanguageGuide/Protocols.html
>   * https://docs.swift.org/swift-book/LanguageGuide/Extensions.html

Neste material são apresentados mais dois recursos fundamentais da Swift. Suas utilizações isoladas, ou ainda mais, combinadas, podem trazer um bom poder para as estruturas de código com a linguagem.

## Protocolos

Um protocolo define um esquema de métodos, propriedades e outros requisitos utilizados para modelar características que atendem a uma determinada tarefa ou funcionalidade. O protocolo pode então ser adotado por uma classe, _struct_ ou _enum_ para que estes atribuam estas características à sua definição fornecendo para isso uma implementação real desses requisitos. Qualquer tipo que satisfaça os requisitos de um protocolo está em conformidade com esse protocolo. Qualquer tipo que entre em conformidade com um protocolo pode ser então tratado e utilizado em termos do tipo que o protocolo designa (de maneira polimórfica).

Além de especificar os requisitos que os tipos em conformidade devem implementar, você pode estender um protocolo para implementar alguns desses requisitos ou implementar funcionalidades adicionais que os tipos em conformidade podem aproveitar, como uma forma de compartilhar não só uma interface comum, mas também possíveis implementações.

> Nota: Os protocolos da Swift oferecem recursos praticamente equivalentes à ideia de Interfaces da Orientação a Objetos e como elas são implementadas em outras linguagens.

### Sintaxe de protocolos

Você define protocolos de maneira muito semelhante a classes, _structs_ e _enums_:

``` swift
protocol AlgumProtocolo {
    // a definição do protocolo vai aqui
}
```

Os tipos personalizados afirmam que entram em conformidade com um protocolo específico colocando o nome do protocolo após o nome do tipo, separado por dois pontos, como parte de sua definição. Vários protocolos podem ser listados e são separados por vírgulas:

``` swift
struct UmaStruct: PrimeiroProtocolo, SegundoProtocolo {
    // a definição da struct vai aqui e você, provavelmente, terá que fornecer implementações para atender aos requisitos de ambos os protocolos
}
```

Se uma classe tiver uma superclasse, liste o nome da superclasse antes de qualquer protocolo que ela adote, seguido de uma vírgula:

``` swift
class UmaClass: UmaSuperclass, PrimeiroProtocolo, OutroProtocolo {
    // a definição da classe vai aqui
}
```

### Requisitos de propriedade

Um protocolo pode exigir que qualquer tipo em conformidade forneça uma propriedade de instância ou propriedade de tipo com um nome e tipo específicos. O protocolo não especifica se a propriedade deve ser uma propriedade armazenada ou uma propriedade computada, apenas o nome e o tipo da propriedade necessária. O protocolo também especifica se cada propriedade deve ser somente leitura (_gettable_) ou leitura e escrita (_gettable and settable_).

Se um protocolo exigir que uma propriedade seja _gettable_ e _settable_, esse requisito de propriedade não pode ser atendido por uma propriedade armazenada de constante (`let`) ou uma propriedade computada somente leitura. Se o protocolo requer apenas que uma propriedade seja obtida, o requisito pode ser satisfeito por qualquer tipo de propriedade, e é válido que a propriedade também seja configurável se isso for útil para seu próprio código.

Os requisitos de propriedade são sempre declarados como propriedades variáveis, prefixadas com a palavra-chave `var`. As propriedades _gettable_ e _settable_ são indicadas escrevendo `{ get set }` após sua declaração de tipo, e as propriedades _gettable_ são indicadas escrevendo `{ get }`.

``` swift
protocol UmProtocolo {
    var precisaSerSettable: Int { get set }
    var naoPrecisaSerSettable: Int { get }
}
```

Sempre prefixe os requisitos de propriedade **de tipo** com a palavra-chave `static` ao defini-los em um protocolo. Essa regra se aplica mesmo que os requisitos de propriedade de tipo possam ser prefixados com a classe ou palavra-chave `static` quando implementados por uma classe:

``` swift
protocol OutroProtocolo {
    static var umaPropriedadeDeTipo: Int { get set }
}
```

Aqui está um exemplo de um protocolo com um único requisito de propriedade de instância:

``` swift
protocol CompletudeDeNome {
    var nomeCompleto: String { get }
}
```

O protocolo `CompletudeDeNome` requer que um tipo em conformidade forneça um nome totalmente qualificado. O protocolo não especifica mais nada sobre a natureza do tipo em conformidade - apenas especifica que o tipo deve ser capaz de fornecer um nome completo para si mesmo. O protocolo afirma que qualquer tipo `CompletudeDeNome` deve ter uma propriedade de instância _gettable_ chamada `nomeCompleto`, que é do tipo `String`.

Aqui está um exemplo de uma estrutura simples que adota e está em conformidade com o protocolo `CompletudeDeNome`:

``` swift
struct Pessoa: CompletudeDeNome {
    var nomeCompleto: String
}

let joao = Pessoa(nomeCompleto: "João da Silva")
// joao.nomeCompleto é "João da Silva"
```

Este exemplo define uma estrutura chamada `Pessoa`, que representa uma pessoa com nome qualificado. Ele afirma que adota o protocolo `CompletudeDeNome` como parte da primeira linha de sua definição.

Cada instância de `Pessoa` tem uma única propriedade armazenada chamada `nomeCompleto`, que é do tipo `String`. Isso corresponde ao único requisito do protocolo `CompletudeDeNome` e significa que a `Pessoa` está em conformidade com o protocolo. (Swift relata um erro em tempo de compilação se um requisito de protocolo não for atendido.)

Aqui está uma classe mais complexa, que também adota e está em conformidade com o protocolo `CompletudeDeNome`:

``` swift
class NaveEspacial: CompletudeDeNome {
    var prefixo: String?
    var nome: String

    init(nome: String, prefixo: String? = nil) {
        self.nome = nome
        self.prefixo = prefixo
    }

    var nomeCompleto: String {
        return (prefixo != nil ? prefixo! + " " : "") + nome
    }
}

var ncc1701 = NaveEspacial(nome: "Enterprise", prefixo: "USS")
// ncc1701.nomeCompleto é "USS Enterprise"
```

Essa classe implementa o requisito de propriedade `nomeCompleto` como uma propriedade somente leitura computada para uma nave espacial. Cada instância da classe `NaveEspacial` armazena um nome obrigatório e um prefixo opcional. A propriedade `nomeCompleto` usa o valor do prefixo, se existir, e o anexa ao início do nome para criar um nome completo para a nave espacial.

### Requisitos do método

Os protocolos podem exigir que métodos de instância específicos e métodos de tipo sejam implementados por tipos em conformidade. Esses métodos são escritos como parte da definição do protocolo exatamente da mesma maneira que para métodos normais de instância e tipo, mas sem chaves ou um corpo de método. Parâmetros variáveis ​​são permitidos, sujeitos às mesmas regras dos métodos normais. Os valores padrão, no entanto, não podem ser especificados para parâmetros de método dentro da definição de um protocolo.

Assim como nos requisitos de propriedade de tipo, você sempre prefixa os requisitos de método de tipo com a palavra-chave `static` quando eles são definidos em um protocolo. Isso é verdade mesmo que os requisitos do método de tipo sejam prefixados com a classe ou palavra-chave `static` quando implementados por uma classe:

``` swift
protocol AlgumProtocolo {
    static func algumMetodoDeTipo()
}
```

O exemplo a seguir define um protocolo com um requisito único de método de instância:

``` swift
protocol GeradorDeNumeroRandomico {
    func random() -> Double
}
```

Este protocolo, `GeradorDeNumeroRandomico`, requer que qualquer tipo em conformidade tenha um método de instância chamado `random`, que retorna um valor `Double` sempre que é chamado. Embora não seja especificado como parte do protocolo, supõe-se que esse valor seja um número de `0.0` até (mas não incluindo) `1.0`.

O protocolo `GeradorDeNumeroRandomico` não faz suposições sobre como cada número aleatório será gerado - ele simplesmente exige que o gerador forneça uma maneira padrão de gerar um novo número aleatório.

Aqui está uma implementação de uma classe que adota e está em conformidade com o protocolo `GeradorDeNumeroRandomico`. Esta classe implementa um algoritmo gerador de números pseudo-aleatórios conhecido como gerador congruencial linear:

``` swift
class GeradorCongruencialLinear: GeradorDeNumeroRandomico {
    var ultimoAleatorio = 42.0
    let m = 139968.0
    let a = 3877.0
    let c = 29573.0

    func random() -> Double {
        ultimoAleatorio = ((ultimoAleatorio * a + c)
            .truncatingRemainder(dividingBy: m))
        return ultimoAleatorio / m
    }
}

let gerador = GeradorCongruencialLinear()

print("Aqui está um número aleatório: \(gerador.random())")
// Imprime "Aqui está um número aleatório: 0.3746499199817101"

print("E outro: \(gerador.random())")
// Imprime "E outro: 0.729023776863283"
```

### Requisitos do método com _mutating_

Às vezes, é necessário que um método altere o estado da instância à qual pertence. Para métodos de instância em _value types_ (ou seja, _structs_ e _enums_), você coloca a palavra-chave `mutating` antes da palavra-chave `func` de um método para indicar que o método tem permissão para mutar a instância à qual pertence e quaisquer propriedades dessa instância.

Se você definir um requisito de método de instância de protocolo destinado a alterar instâncias de qualquer tipo que adote o protocolo, marque o método com a palavra-chave `mutating` como parte da definição do protocolo. Isso permite que _structs_ e _enums_ adotem o protocolo e satisfaçam esse requisito de método.

> Nota: Se você marcar um requisito de método de instância de protocolo como `mutating`, não será necessário escrever a palavra-chave `mutating` ao escrever uma implementação desse método para uma classe. A palavra-chave `mutating` é usada apenas por _structs_ e _enums_.

O exemplo abaixo define um protocolo chamado `Togglable`, que define um requisito de método de instância único chamado `toggle`. Como o próprio nome sugere, o método `toggle()` destina-se a alternar ou inverter o estado de qualquer tipo em conformidade, geralmente modificando uma propriedade desse tipo.

O método `toggle()` é marcado com a palavra-chave `mutating` como parte da definição do protocolo `Togglable`, para indicar que se espera que o método altere o estado de uma instância em conformidade quando for chamado:

``` swift
protocol Togglable {
    mutating func toggle()
}
```

Se você implementar o protocolo `Togglable` para uma _struct_ ou _enum_, essa _struct_ ou _enum_ pode estar em conformidade com o protocolo fornecendo uma implementação do método `toggle()` que também é marcado como _mutating_.

O exemplo abaixo define uma _enum_ chamada `Interruptor`. Essa _enum_ alterna entre dois estados, indicados pelos casos de _enum_ `.desligado` e `.ligado`. A implementação de alternância da _enum_ é marcada como mutante, para corresponder aos requisitos do protocolo `Togglable`:

``` swift
enum Interruptor: Togglable {
    case desligado, ligado
    
    mutating func toggle() {
        switch self {
        case .desligado:
            self = .ligado
        case .ligado:
            self = .desligado
        }
    }
}

var interruptorDeLuz = Interruptor.desligado

interruptorDeLuz.toggle()
// interruptorDeLuz agora é igual a .ligado
```

### Requisitos do inicializador

Os protocolos podem exigir que inicializadores específicos sejam implementados por tipos em conformidade. Você escreve esses inicializadores como parte da definição do protocolo exatamente da mesma maneira que para inicializadores normais, mas sem chaves ou um corpo inicializador:

``` swift
protocol UmProtocolo {
    init(umParametro: Int)
}
```

#### Requisitos do inicializador de protocolo em implementações de classe

Você pode implementar um requisito de inicializador em uma classe como um inicializador designado ou um inicializador de conveniência. Em ambos os casos, você deve marcar a implementação do inicializador com o modificador `required`:

``` swift
class UmaClasse: UmProtocolo {
    required init(umParametro: Int) {
        // implementação do inicializador vai aqui
    }
}
```

O uso do modificador `required` garante que você forneça uma implementação explícita ou herdada do requisito do inicializador em todas as subclasses da classe em conformidade, de modo que elas também estejam em conformidade com o protocolo.

> Nota: Você não precisa marcar implementações de inicializadores de protocolo com o modificador `required` em classes marcadas com o modificador `final`, porque as classes finais não podem ter subclasses.

Se uma subclasse substituir um inicializador designado em uma superclasse e também implementar um requisito de inicializador de um protocolo, marque a implementação do inicializador com os modificadores `required` e de `override`:

``` swift
protocol UmProtocolo {
    init()
}

class UmaSuperclasse {
    init() {
        // implementação do inicializador vai aqui
    }
}

class UmaSubclasse: UmaSuperclasse, UmProtocolo {
    // "required" da implementação de UmProtocolo; "override" de UmaSuperclasse
    required override init() {
        // implementação do inicializador vai aqui
    }
}
```

### Protocolos como tipos

Os protocolos não implementam nenhuma funcionalidade por si mesmos. No entanto, você pode usar protocolos como tipos completos em seu código. Usar um protocolo como tipo às vezes é chamado de tipo existencial, que vem da frase “existe um tipo T tal que T está em conformidade com o protocolo” (pouco importa, desta perspectiva, de qual tipo específico T é).

Você pode usar um protocolo em muitos lugares onde outros tipos são permitidos, incluindo:

* Como um tipo de parâmetro ou tipo de retorno em uma função, método ou inicializador
* Como o tipo de uma constante, variável ou propriedade
* Como o tipo de itens em um _array_

> Nota: Como os protocolos são tipos, comece seus nomes com uma letra maiúscula (como GeradorDeNumeroRandomico) para corresponder aos nomes de outros tipos em Swift (como Int, String e Double).

Aqui está um exemplo de um protocolo usado como um tipo:

``` swift
class Dado {
    let lados: Int
    let gerador: GeradorDeNumeroRandomico

    init(lados: Int, gerador: GeradorDeNumeroRandomico) {
        self.lados = lados
        self.gerador = gerador
    }

    func lanca() -> Int {
        return Int(gerador.random() * Double(lados)) + 1
    }
}
```

Este exemplo define uma nova classe chamada `Dado`, que representa um dado de `n` faces para uso em um jogo de tabuleiro. Instâncias de `Dado` têm uma propriedade inteira chamada `lados`, que representa quantos lados eles têm, e uma propriedade chamada `gerador`, que fornece um gerador de números aleatórios para criar valores para a rolagem de dados.

A propriedade do gerador é do tipo `GeradorDeNumeroRandomico`. Portanto, você pode configurá-lo para uma instância de qualquer tipo que adote o protocolo `GeradorDeNumeroRandomico`. Nada mais é exigido da instância que você atribui a essa propriedade, exceto que a instância deve adotar o protocolo `GeradorDeNumeroRandomico`. Como seu tipo é `GeradorDeNumeroRandomico`, o código dentro da classe `Dado` só pode interagir com o gerador de maneiras que se aplicam a todos os geradores que estejam em conformidade com esse protocolo. Isso significa que ele não pode usar nenhum método ou propriedade definido pelo tipo específico subjacente do gerador. No entanto, você pode fazer o _casting_ de um tipo de protocolo para um tipo específico da mesma forma que pode fazer o _casting_ de uma superclasse para uma subclasse como em `let controller = window!.rootViewController as! AutoresViewController`.

`Dado` também tem um inicializador, para configurar seu estado inicial. Este inicializador possui um parâmetro chamado `gerador`, que também é do tipo `GeradorDeNumeroRandomico`. Você pode passar um valor de qualquer tipo compatível para este parâmetro ao inicializar uma nova instância de `Dado`.

`Dado` fornece um método de instância, `lanca()`, que retorna um valor inteiro entre 1 e o número de lados do dado. Esse método chama o método `random()` do gerador para criar um novo número aleatório entre `0.0` e `1.0` e usa esse número aleatório para criar um valor de lançamento de dados dentro do intervalo correto. Como o gerador é conhecido por adotar `GeradorDeNumeroRandomico`, é garantido que ele tenha um método `random()` para chamar.

Veja como a classe `Dado` pode ser usada para criar um dado de seis lados com uma instância de `GeradorCongruencialLinear` como seu gerador de números aleatórios:

``` swift
var d6 = Dado(lados: 6, gerador: GeradorCongruencialLinear())

for _ in 1...5 {
    print("O lançamento de dados aleatório é \(d6.lanca())")
}

// O lançamento de dados aleatório é 3
// O lançamento de dados aleatório é 5
// O lançamento de dados aleatório é 4
// O lançamento de dados aleatório é 5
// O lançamento de dados aleatório é 4
```

### Herança de protocolo

Um protocolo pode herdar um ou mais outros protocolos e pode adicionar requisitos adicionais aos requisitos que herda. A sintaxe para herança de protocolo é semelhante à sintaxe para herança de classe, mas com a opção de listar vários protocolos herdados, separados por vírgulas:

``` swift
protocol ProtocoloHerdando: UmProtocolo, OutroProtocolo {
    // a definição do protocolo vai aqui
}
```

Aqui está um exemplo de implementação com um protocolo que herda requisitos de outro protocolo:

``` swift
protocol RepresentavelPorTexto {
    var descricaoTextual: String { get }
}

protocol RepresentavelPorTextoFormatado: RepresentavelPorTexto {
    var descricaoTextualFormatada: String { get }
}
```

Este exemplo define um protocolo, `RepresentavelPorTextoFormatado`, que herda de outro, `RepresentavelPorTexto`. Qualquer coisa que adote `RepresentavelPorTextoFormatado` deve atender a todos os requisitos impostos por `RepresentavelPorTexto`, além dos requisitos adicionais impostos por `RepresentavelPorTextoFormatado`. Neste exemplo, `RepresentavelPorTextoFormatado` descreve um único requisito adicional para fornecer uma propriedade _gettable_ chamada `descricaoTextualFormatada` que retorna uma String.

### Protocolos somente de classe

Você pode limitar a adoção do protocolo a tipos de classe (e não _structs_ ou _enums_) adicionando o protocolo `AnyObject` à lista de herança de um protocolo.

``` swift
protocol UmProtocoloSomentoDeClasse: AnyObject, AlgumOutroProtocoloHerdado {
    // definição de protocolo somente de classe vai aqui
}
```

No exemplo acima, `UmProtocoloSomentoDeClasse` só pode ser adotado por tipos de classe. É um erro em tempo de compilação escrever uma _struct_ ou definição de _enum_ que tenta adotar `UmProtocoloSomentoDeClasse`.

> Nota: Use um protocolo somente de classe quando o comportamento definido pelos requisitos desse protocolo assumem ou exigem que um tipo em conformidade tenha semântica de referência em vez de semântica de valor.

## Extensions

As extensões adicionam novas funcionalidades a uma classe, _struct_, _enum_ ou tipo de protocolo existente. Isso inclui a capacidade de estender tipos para os quais você não tem acesso ao código-fonte original (algo conhecido como modelagem retroativa). As extensões são semelhantes às categorias em Objective-C. (Ao contrário das categorias Objective-C, as extensões Swift não têm nomes.)

Extensões da Swift podem:

* Adicionar propriedades computadas de instância e propriedades de tipo
* Definir métodos de instância e métodos de tipo
* Fornecer novos inicializadores
* Definir e utilizar novos tipos aninhados
* Fazer com que um tipo existente entre em conformidade com um protocolo

No Swift, você pode até estender um protocolo para fornecer implementações de seus requisitos ou adicionar funcionalidades que os tipos em conformidade podem aproveitar.

> Nota: As extensões podem adicionar novas funcionalidades a um tipo, mas não podem sobrescrever a funcionalidade existente.

### Sintaxe das _extensions_

Declare extensões com a palavra-chave `extension`:

``` swift
extension AlgumTipo {
    // novas funcionalidades para adicionar a AlgumTipo vão aqui
}
```

Uma extensão pode estender um tipo existente para que adote um ou mais protocolos. Para adicionar conformidade de protocolo, você escreve os nomes dos protocolos da mesma forma que os escreve para uma classe ou _struct_:

``` swift
extension AlgumTipo: AlgumProtocolo, OutroProtocolo {
    // implementação de requisitos dos protocolos vão aqui
}
```

> Nota: Se você definir uma extensão para adicionar nova funcionalidade a um tipo existente, a nova funcionalidade estará disponível em todas as instâncias existentes desse tipo, mesmo que tenham sido criadas antes da definição da extensão.

### Propriedades computadas

As extensões podem adicionar propriedades computadas de instância e propriedades de tipo a tipos existentes. Este exemplo adiciona cinco propriedades computadas de instância ao tipo `Double` integrado do Swift, para fornecer suporte básico para trabalhar com unidades de distância:

``` swift
extension Double {
    var km: Double { return self * 1_000.0 }
    var m: Double { return self }
    var cm: Double { return self / 100.0 }
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.28084 }
}

let umaPolegada = 25.4.mm
print("Uma polegada é \(umaPolegada) metros")
// Imprime "Uma polegada é 0,0254 metros"

let tresPes = 3.ft
print("Três pés é \(tresPes) metros")
// Imprime "Três pés é 0.914399970739201 metros"
```

Essas propriedades computadas expressam que um valor `Double` deve ser considerado como uma certa unidade de comprimento. Embora sejam implementadas como propriedades computadas, os nomes dessas propriedades podem ser anexados a um valor literal de ponto flutuante com sintaxe de ponto, como forma de usar esse valor literal para realizar conversões de distância.

Neste exemplo, um valor `Double` de `1.0` é considerado para representar “um metro”. É por isso que a propriedade computada `m` retorna `self` — a expressão `1.m` é considerada para calcular um valor `Double` de `1.0`.

Outras unidades requerem que alguma conversão seja expressa como um valor medido em metros. Um quilômetro é igual a 1.000 metros, portanto, a propriedade computada `km` multiplica o valor por `1_000.00` para converter em um número expresso em metros. Da mesma forma, existem `3.28084` pés em um metro e, portanto, a propriedade computada `ft` divide o valor `Double` subjacente por `3.28084`, para convertê-lo de pés para metros.

Essas propriedades são propriedades computadas somente leitura e, portanto, são expressas sem a palavra-chave get, por brevidade. Seu valor de retorno é do tipo `Double` e pode ser usado em cálculos matemáticos onde quer que um `Double` seja aceito:

``` swift
let umaMaratona = 42.km + 195.m
print("Uma maratona tem \(umaMaratona) metros de comprimento")
// Imprime "Uma maratona tem 42.195,0 metros de comprimento"
```

> Nota: As extensões podem adicionar novas propriedades computadas, mas não podem adicionar propriedades armazenadas ou adicionar observadores de propriedades a propriedades existentes.

### Inicializadores

As extensões podem adicionar novos inicializadores a tipos existentes. Isso permite estender outros tipos para aceitar seus próprios tipos personalizados como parâmetros de inicialização ou fornecer opções de inicialização adicionais que não foram incluídas como parte da implementação original.

As extensões podem adicionar novos inicializadores de conveniência a uma classe, mas não podem adicionar novos inicializadores ou desinicializadores designados a uma classe. Inicializadores e desinicializadores designados sempre devem ser fornecidos pela implementação da classe original.

Se você usar uma extensão para adicionar um inicializador a um _value type_ que fornece valores padrão para todas as suas propriedades armazenadas e não define nenhum inicializador personalizado, você pode chamar o inicializador padrão e o inicializador auto gerado para esse _value type_ de dentro do inicializador da sua extensão . Este não seria o caso se você tivesse escrito o inicializador como parte da implementação original do _value type_.

Se você usar uma extensão para adicionar um inicializador a uma _struct_ que foi declarada em outro módulo, o novo inicializador não poderá acessar `self` até chamar um inicializador do módulo de definição.

O exemplo abaixo define uma estrutura `Retangulo` personalizada para representar um retângulo geométrico. O exemplo também define duas _structs_ de suporte chamadas `Tamanho` e `Ponto`, ambas fornecendo valores padrão de `0.0` para todas as suas propriedades:

``` swift
struct Tamanho {
    var largura = 0.0, altura = 0.0
}

struct Ponto {
    var x = 0.0, y = 0.0
}

struct Retangulo {
    var origem = Ponto()
    var tamanho = Tamanho()
}
```

Como a estrutura `Retangulo` fornece valores padrão para todas as suas propriedades, ela recebe um inicializador padrão e um inicializador gerado automaticamente (_memberwise initializer_). Esses inicializadores podem ser usados ​​para criar novas instâncias de `Retangulo`:

``` swift
let retanguloPadrao = Retangulo()
let retanguloMemberwise = Retangulo(
    origem: Ponto(x: 2.0, y: 2.0),
    tamanho: Tamanho(largura: 5.0, altura: 5.0)
)
```

Você pode estender a _struct_ `Retangulo` para fornecer um inicializador adicional que tenha um ponto central e tamanho específicos:

``` swift
extension Retangulo {
    init(centro: Ponto, tamanho: Tamanho) {
        let origemX = centro.x - (tamanho.largura / 2)
        let origemY = centro.y - (tamanho.altura / 2)
        self.init(origem: Ponto(x: origemX, y: origemY), tamanho: tamanho)
    }
}
```

Esse novo inicializador começa calculando um ponto de origem apropriado com base no ponto central fornecido e no valor de tamanho. O inicializador então chama o inicializador automático de membros da _struct_ `init(origem:tamanho:)`, que armazena os novos valores de origem e tamanho nas propriedades apropriadas:

``` swift
let retanguloCentral = Retangulo(
    centro: Ponto(x: 4.0, y: 4.0),
    tamanho: Tamanho(largura: 3.0, altura: 3.0)
)
// a origem do retanguloCentral é (2.5, 2.5) e seu tamanho é (3.0, 3.0)
```

> Nota: Se você fornecer uma extensão a um novo inicializador, ainda será responsável por garantir que cada instância seja totalmente inicializada assim que o inicializador for concluído.

### Métodos

As extensões podem adicionar novos métodos de instância e métodos de tipo a tipos existentes. O exemplo a seguir adiciona um novo método de instância chamado repetições ao tipo `Int`:

``` swift
extension Int {
    func repeticoes(tarefa: () -> Void) {
        for _ in 0..<self {
            tarefa()
        }
    }
}
```

O método `repeticoes(tarefa:)` recebe um único argumento do tipo `() -> Void`, que indica uma função que não possui parâmetros e não retorna um valor.

Depois de definir essa extensão, você pode chamar o método `repeticoes(tarefa:)` em qualquer número inteiro para executar uma tarefa muitas vezes:

``` swift
3.repeticoes {
    print("Olá!")
}

// Olá!
// Olá!
// Olá!
```

### Métodos de instância com _mutating_

Os métodos de instância adicionados com uma extensão também podem alterar o estado (ou mutar) da própria instância. Os métodos de _struct_ e _enum_ que modificam a si mesmo ou suas propriedades devem marcar o método de instância como _mutating_, assim como os métodos mutantes de uma implementação original.

O exemplo abaixo adiciona um novo método chamado `quadrado` ao tipo `Int` da Swift, que eleva o valor original ao quadrado:

``` swift
extension Int {
    mutating func quadrado() {
        self = self * self
    }
}

var algumInt = 3
algumInt.quadrado()

// algumInt agora é 9
```

### Tipos aninhados

As extensões podem adicionar novos tipos aninhados a classes, _structs_ e _enums_ existentes:

``` swift
extension Int {
    enum Tipo {
        case negativo, zero, positivo
    }

    var tipo: Tipo {
        switch self {
        case 0:
            return .zero
        case let x where x > 0:
            return .positivo
        default:
            return .negativo
        }
    }
}
```

Este exemplo adiciona uma nova _enum_ aninhada a `Int`. Essa _enum_, chamada `Tipo`, expressa o tipo de número que um determinado inteiro representa. Especificamente, ele expressa se o número é negativo, zero ou positivo.

Este exemplo também adiciona uma nova propriedade de instância computada a `Int`, chamada `tipo`, que retorna o caso de _enum_ `Tipo` apropriado para esse inteiro.

A _enum_ aninhada agora pode ser usada com qualquer valor `Int`:

``` swift
func imprimeTiposDosInts(_ numeros: [Int]) {
    for numero in numeros {
        switch numero.tipo {
        case .negativo:
            print("- ", terminator: "")
        case .zero:
            print("0 ", terminator: "")
        case .positivo:
            print("+ ", terminator: "")
        }
    }
    print("")
}

imprimeTiposDosInts([3, 19, -27, 0, -6, 0, 7])
// Imprime "+ + - 0 - 0 + "
```

Essa função, `imprimeTiposDosInts(_:)`, recebe um _array_ de entrada de valores `Int` e itera sobre esses valores por sua vez. Para cada inteiro no _array_, a função considera a propriedade `tipo` computada para esse inteiro e imprime uma descrição apropriada.

> Nota: já se sabe que `numero.tipo` é do tipo `Int.Tipo`. Por causa disso, todos os valores para `Int.Tipo` nas cláusulas `case` podem ser escritos em forma abreviada dentro do `switch`, como `.negativo` em vez de `Int.Tipo.negativo`.
