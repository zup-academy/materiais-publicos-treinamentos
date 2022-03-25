# Mais sobre propriedades

>Esse material teórico foi adaptado da fonte original disponível em https://docs.swift.org/swift-book/LanguageGuide/Properties.html

As propriedades associam valores a uma classe, _struct_ ou _enum_ específica. As propriedades armazenadas _(stored properties)_ armazenam valores constantes e variáveis como parte de uma instância, enquanto as propriedades computadas _(computed properties)_ calculam (em vez de armazenar) um valor. Propriedades computadas são fornecidas por classes, _structs_ e _enums_. As propriedades armazenadas são fornecidas apenas por classes e _structs_.

As propriedades armazenadas e computadas geralmente estão associadas a instâncias de um tipo específico. No entanto, as propriedades também podem ser associadas à propria definição do tipo. Essas propriedades são conhecidas como propriedades de tipo _(type properties)_.

Além disso, você pode definir observadores de propriedade _(property observers)_ para monitorar as alterações no valor de uma propriedade, às quais você pode responder com ações personalizadas. Observadores de propriedade podem ser adicionados a propriedades armazenadas que você mesmo define e também a propriedades que uma subclasse herda de sua superclasse.

Você também pode usar um wrapper de propriedade para reutilizar o código no getter e setter de várias propriedades.

## Propriedades armazenadas _(stored properties)_

Em sua forma mais simples, uma propriedade armazenada é uma constante ou variável que é armazenada como parte de uma instância de uma classe ou _struct_ específica. As propriedades armazenadas podem ser propriedades armazenadas de variáveis ​​(introduzidas pela palavra-chave `var`) ou propriedades armazenadas constantes (introduzidas pela palavra-chave `let`).

Você pode fornecer um valor padrão para uma propriedade armazenada como parte de sua definição. Você também pode definir e modificar o valor inicial de uma propriedade armazenada durante a inicialização. Isso é verdade mesmo para propriedades armazenadas de constantes.

O exemplo abaixo define uma estrutura chamada `IntervaloDeComprimentoFixo`, que descreve um intervalo de inteiros cujo comprimento do intervalo não pode ser alterado após sua criação:

``` swift
struct IntervaloDeComprimentoFixo {
    var valorInicial: Int
    let comprimento: Int
}

var intervaloDeTresItens = IntervaloDeComprimentoFixo(valorInicial: 0, comprimento: 3)
// o intervalo representa os valores inteiros 0, 1 e 2

intervaloDeTresItens.valorInicial = 6
// o intervalo agora representa os valores inteiros 6, 7 e 8
```

As instâncias de `IntervaloDeComprimentoFixo` têm uma propriedade armazenada de variável chamada `valorInicial` e uma propriedade armazenada constante chamada `comprimento`. No exemplo acima, o comprimento é inicializado quando o novo intervalo é criado e não pode ser alterado depois, porque é uma propriedade constante.

### Propriedades armazenadas de instâncias constantes de _struct_ 

Se você criar uma instância de uma _struct_ e atribuir essa instância a uma constante, não poderá modificar as propriedades da instância, mesmo que tenham sido declaradas como propriedades variáveis:

``` swift
let intervaloDeQuatroItens = IntervaloDeComprimentoFixo(valorInicial: 0, comprimento: 4)
// este intervalo representa valores inteiros 0, 1, 2 e 3

intervaloDeQuatroItens.valorInicial = 6
// isso reportará um erro, mesmo que valorInicial seja uma propriedade variável
```

Como `intervaloDeQuatroItens` é declarado como uma constante (com a palavra-chave `let`), não é possível alterar sua propriedade `valorInicial`, mesmo que `valorInicial` tenha sido declarado com a palavra-chave `var`.

Esse comportamento é devido às _structs_ serem _value types_. Quando uma instância de um tipo de valor é marcada como uma constante, todas as suas propriedades também o são.

O mesmo não acontece com as classes, que são _reference types_. Se você atribuir uma instância de um tipo referência a uma constante, ainda poderá alterar as propriedades da variável dessa instância.

## Propriedades computadas _(computed properties)_

Além das propriedades armazenadas, classes, _structs_ e _enums_ podem definir propriedades computadas, que na verdade não armazenam um valor. Em vez disso, eles fornecem um _getter_ e um _setter_ opcional para recuperar e definir outras propriedades e valores indiretamente.

``` swift
struct Ponto {
    var x = 0.0, y = 0.0
}

struct Tamanho {
    var largura = 0.0, altura = 0.0
}

struct Retangulo {
    var origem = Ponto()
    var tamanho = Tamanho()
    var centro: Ponto {
        get {
            let centroX = origem.x + (tamanho.largura / 2)
            let centroY = origem.y + (tamanho.altura / 2)
    
            return Ponto(x: centroX, y: centroY)
        }
        set(novoCentro) {
            origem.x = novoCentro.x - (tamanho.largura / 2)
            origem.y = novoCentro.y - (tamanho.altura / 2)
        }
    }
}

var quadrado = Retangulo(
    origem: Ponto(x: 0.0, y: 0.0),
    tamanho: Tamanho(largura: 10.0, altura: 10.0)
)

let centroInicialDoQuadrado = quadrado.centro
// centroInicialDoQuadrado está em (5.0, 5.0)

quadrado.centro = Ponto(x: 15.0, y: 15.0)
print("quadrado.origem agora está em (\(quadrado.origem.x), \(quadrado.origem.y))"))
// Imprime "quadrado.origem está agora em (10.0, 10.0)"
```

Este exemplo define três _structs_ para trabalhar com formas geométricas:

* `Ponto` encapsula as coordenadas x e y de um ponto.
* `Tamanho` encapsula uma largura e uma altura.
* `Retangulo` define um retângulo por um ponto de origem e um tamanho.

A estrutura `Retangulo` também fornece uma propriedade computada chamada `centro`. A posição central atual de um `Retangulo` sempre pode ser determinada a partir de sua origem e tamanho e, portanto, você não precisa armazenar o ponto central como um valor `Ponto` explícito. Em vez disso, `Retangulo` define um _getter_ e _setter_ personalizado para uma variável computada chamada `centro`, para permitir que você trabalhe com o centro do retângulo como se fosse uma propriedade real armazenada.

O exemplo acima cria uma nova variável `Retangulo` chamada `quadrado`. A variável `quadrado` é inicializada com um ponto de origem de (0, 0) e uma largura e altura de 10. Este quadrado é representado pelo quadrado verde claro no diagrama abaixo.

A propriedade `centro` da variável `quadrado` é então acessada através da sintaxe de ponto (`quadrado.centro`), que faz com que o _getter_ de `centro` seja chamado para recuperar o valor da propriedade atual. Em vez de retornar um valor existente, o _getter_ realmente calcula e retorna um novo `Ponto` para representar o centro do quadrado. Como pode ser visto acima, o _getter_ retorna corretamente um ponto central de (5, 5).

A propriedade `centro` é então definida com um novo valor de (15, 15), que move o quadrado para cima e para a direita, para a nova posição mostrada pelo quadrado verde escuro no diagrama abaixo. Definir a propriedade `centro` chama o _setter_ para `centro`, que modifica os valores `x` e `y` da propriedade armazenada `origem` e move o quadrado para sua nova posição.

<p align="center">
<img alt="Ilustração do exemplo acima no plano cartesiano, contendo um quadrado verde um pouco ofuscado que marca a configuração inicial e um segundo quadrado verde de cor sólida que marca a posição de acordo com a configuração final" src="https://docs.swift.org/swift-book/_images/computedProperties_2x.png" width="50%" />
</p>

### Shorthand para Declaração de _Setters_

Se o _setter_ de uma propriedade computada não definir um nome para o novo valor a ser definido, um nome padrão de `newValue` será usado. Aqui está uma versão alternativa da estrutura `Retangulo` que tira proveito dessa notação simplificada:

``` swift
struct RetanguloAlternativo {
    var origem = Ponto()
    var tamanho = Tamanho()
    var centro: Ponto {
        get {
            let centroX = origem.x + (tamanho.largura / 2)
            let centroY = origem.y + (tamanho.altura / 2)
            return Ponto(x: centroX, y: centroY)
        }
        set {
            origem.x = newValue.x - (tamanho.largura / 2)
            origem.y = newValue.y - (tamanho.altura / 2)
        }
    }
}
```

### Shorthand para Declaração de _Getters_

Se todo o corpo de um _getter_ for uma única expressão, o _getter_ retornará implicitamente essa expressão. Aqui está uma outra versão da estrutura `Retangulo` que aproveita ambas as notações simplificadas para _getters_ e _setters_:

``` swift
struct RetanguloCompacto {
    var origem = Ponto()
    var tamanho = Tamanho()
    var centro: Ponto {
        get {
            Ponto(x: origem.x + (tamanho.largura / 2),
                  y: origem.y + (tamanho.altura / 2))
        }
        set {
            origem.x = newValue.x - (tamanho.largura / 2)
            origem.y = newValue.y - (tamanho.altura / 2)
        }
    }
}
```

Omitir o retorno de um _getter_ segue as mesmas regras que omitir o retorno de uma função, conforme descrito no material teórico sobre Funções com retorno implícito.

### Propriedades computadas somente leitura

Uma propriedade computada com um _getter_ mas sem nenhum _setter_ é conhecida como uma propriedade computada somente leitura. Uma propriedade computada somente leitura sempre retorna um valor e pode ser acessada por meio da sintaxe de ponto, mas não pode ser definida com um valor diferente.

> Nota: Você deve declarar propriedades computadas — incluindo propriedades computadas somente leitura — como propriedades variáveis ​​com a palavra-chave `var`, porque seu valor não é fixo. A palavra-chave `let` é usada apenas para propriedades constantes, para indicar que seus valores não podem ser alterados depois de definidos como parte da inicialização da instância.

Você pode simplificar a declaração de uma propriedade computada somente leitura removendo a palavra-chave `get` e suas chaves:

``` swift
struct Cuboide {
    var largura = 0.0, altura = 0.0, profundidade = 0.0
    var volume: Double {
        return  largura * altura * profundidade
    }
}

let quatroPorCincoPorDois = Cuboide(largura: 4.0, altura: 5.0, profundidade: 2.0)
print("o volume de quatroPorCincoPorDois é \(quatroPorCincoPorDois.volume)")
// Imprime "o volume de quatroPorCincoPorDois é 40.0"
```

Este exemplo define uma nova estrutura chamada `Cuboide`, que representa uma caixa retangular 3D com propriedades de largura, altura e profundidade. Essa _struct_ também possui uma propriedade computada somente leitura chamada `volume`, que calcula e retorna o volume atual do cuboide. Não faz sentido que o volume seja configurável, porque seria ambíguo quanto a quais valores de largura, altura e profundidade devem ser usados ​​para um valor de volume específico. No entanto, é útil para um Cuboide fornecer uma propriedade computada somente leitura para permitir que usuários externos descubram seu volume calculado atual.

## Observadores de propriedade _(property observers)_

Os observadores de propriedade _(property observers)_ observam e respondem a mudanças no valor de uma propriedade. Os observadores de propriedade são chamados sempre que o valor de uma propriedade é definido, mesmo que o novo valor seja igual ao valor atual da propriedade.

Você pode adicionar observadores de propriedade nos seguintes locais:

* Propriedades armazenadas que você define
* Propriedades armazenadas que você herda
* Propriedades computadas que você herda

Para uma propriedade herdada, você adiciona um observador de propriedade substituindo essa propriedade em uma subclasse. Para uma propriedade computada que você define, use o _setter_ da propriedade para observar e responder a alterações de valor, em vez de tentar criar um observador.

Você tem a opção de definir um ou ambos os observadores em uma propriedade:

* willSet é chamado imediatamente antes do valor ser armazenado.
* didSet é chamado imediatamente após o novo valor ser armazenado.

Se você implementar um observador `willSet`, ele passará o novo valor da propriedade como um parâmetro constante. Você pode especificar um nome para este parâmetro como parte de sua implementação `willSet`. Se você não escrever o nome do parâmetro e os parênteses em sua implementação, o parâmetro será disponibilizado com um nome de parâmetro padrão de `newValue`.

Da mesma forma, se você implementar um observador `didSet`, ele receberá um parâmetro constante contendo o valor da propriedade antiga. Você pode nomear o parâmetro ou usar o nome de parâmetro padrão de `oldValue`. Se você atribuir um valor a uma propriedade dentro de seu próprio observador `didSet`, o novo valor atribuído substituirá aquele que acabou de ser definido.

> Nota: Os observadores `willSet` e `didSet` das propriedades da superclasse são chamados quando uma propriedade é definida em um inicializador de subclasse, após o inicializador da superclasse ter sido chamado. Eles não são chamados enquanto uma classe está definindo suas próprias propriedades, antes que o inicializador da superclasse seja chamado.

Aqui está um exemplo de `willSet` e `didSet` em ação. O exemplo abaixo define uma nova classe chamada `ContadorDePassos`, que rastreia o número total de passos que uma pessoa dá enquanto caminha. Essa classe pode ser usada com dados de entrada de um pedômetro ou outro contador de passos para acompanhar o exercício de uma pessoa durante sua rotina diária.

``` swift
class ContadorDePassos {
    var passosTotais: Int = 0 {
        willSet(novoPassosTotais) {
            print("Prestes a definir passosTotais para \(novoPassosTotais)")
        }
        didSet {
            if passosTotais > oldValue {
                print("Adicionados \(passosTotais - oldValue) passos")
            }
        }
    }
}

let contadorDePassos = ContadorDePassos()
contadorDePassos.passosTotais = 200
// Prestes a definir passosTotais para 200
// Adicionados 200 passos

contadorDePassos.passosTotais = 360
// Prestes a definir passosTotais para 360
// Adicionados 160 passos

contadorDePassos.passosTotais = 896
// Prestes a definir passosTotais para 896
// Adicionados 536 passos
```

A classe `ContadorDePassos` declara uma propriedade `passosTotais` do tipo `Int`. Esta é uma propriedade armazenada com observadores `willSet` e `didSet`.

Os observadores `willSet` e `didSet` para `passosTotais` são chamados sempre que a propriedade recebe um novo valor. Isso é verdade mesmo se o novo valor for igual ao valor atual.

O observador `willSet` deste exemplo usa um nome de parâmetro personalizado de `novoPassosTotais` para o próximo novo valor. Neste exemplo, ele simplesmente imprime o valor que está prestes a ser definido.

O observador `didSet` é chamado depois que o valor de `passosTotais` é atualizado. Ele compara o novo valor de `passosTotais` com o valor antigo. Se o número total de passos aumentou, uma mensagem será impressa para indicar quantos novos passos foram realizadas. O observador `didSet` não fornece um nome de parâmetro personalizado para o valor antigo, e o nome padrão de `oldValue` é usado.

> Nota: Se você passar uma propriedade que tem observadores para uma função como parâmetro de entrada, os observadores `willSet` e `didSet` serão sempre chamados. Isso ocorre devido ao modelo de memória copy-in copy-out para parâmetros de entrada e saída: O valor é sempre gravado de volta na propriedade no final da função.

## Wrappers de propriedades _(property wrappers)_

Um _property wrapper_ (wrapper de propriedade) adiciona uma camada de separação entre o código que gerencia como uma propriedade é armazenada e o código que define uma propriedade. Por exemplo, se você tiver propriedades que forneçam verificações de segurança ou armazenem seus dados subjacentes em um banco de dados, será necessário escrever o código relativo a essa funcionalidade em cada propriedade. Ao usar um _property wrapper_, você escreve o código de gerenciamento uma vez ao definir o _wrapper_ e, em seguida, reutiliza esse código de gerenciamento aplicando-o a várias propriedades.

Para definir um _property wrapper_, você cria uma _struct_, _enum_ ou classe que define uma propriedade `wrappedValue`. No código abaixo, a estrutura `DozeOuMenos` garante que o valor que ela envolve sempre contenha um número menor ou igual a 12. Se você pedir para armazenar um número maior, ele armazenará `12`.

``` swift
@propertyWrapper
struct DozeOuMenos {
    private var numero = 0
    
    var wrappedValue: Int {
        get { return numero }
        set { numero = min(newValue, 12) }
    }
}
```

O _setter_ garante que os novos valores sejam menores ou iguais a 12 e o _getter_ retorna o valor armazenado.

> Nota: A declaração para `numero` no exemplo acima marca a variável como privada, o que garante que `numero` seja usado apenas na implementação de `DozeOuMenos`. O código escrito em qualquer outro lugar acessa o valor usando o _getter_ e o _setter_ para `wrappedValue` e não pode usar o número diretamente.

Você aplica um _wrapper_ a uma propriedade escrevendo o nome do _wrapper_ antes da propriedade como um atributo. Aqui está uma _struct_ que armazena um retângulo que usa o _property wrapper_ `DozeOuMenos` para garantir que suas dimensões sejam sempre 12 ou menos:

``` swift
struct PequenoRetangulo {
    @DozeOuMenos var height: Int
    @DozeOuMenos var largura: Int
}

var retangulo = PequenoRetangulo()
print(retangulo.altura)
// Imprime "0"

retangulo.altura = 10
print(retangulo.altura)
// Imprime "10"

retangulo.altura = 24
print(retangulo.altura)
// Imprime "12"
```

As propriedades `altura` e `largura` obtêm seus valores iniciais da definição de `DozeOuMenos`, que define `DozeOuMenos.numero` como zero. O _setter_ em `DozeOuMenos` trata `10` como um valor válido, portanto, armazenar o número `10` em `retangulo.altura` continua conforme escrito. No entanto, `24` é maior do que `DozeOuMenos` permite, então tentar armazenar 24 acaba configurando `retangulo.altura` para `12`, o maior valor permitido.

Quando você aplica um _wrapper_ a uma propriedade, o compilador sintetiza o código que fornece armazenamento para o _wrapper_ e o código que fornece acesso à propriedade por meio do _wrapper_. (O _property wrapper_ é responsável por armazenar o valor encapsulado, portanto, não há código sintetizado para isso.) Você poderia escrever um código que usasse o comportamento de um _property wrapper_, sem tirar vantagem da sintaxe de atributo especial. Por exemplo, aqui está uma versão de `PequenoRetangulo` da listagem de código anterior que envolve suas propriedades na estrutura `DozeOuMenos` explicitamente, em vez de escrever `@DozeOuMenos` como um atributo:

``` swift
struct PequenoRetangulo {
    private var _altura = DozeOuMenos()
    private var _largura = DozeOuMenos()
    
    var altura: Int {
        get { return _altura.wrappedValue }
        set { _altura.wrappedValue = newValue }
    }
    
    var largura: Int {
        get { return _largura.wrappedValue }
        set { _largura.wrappedValue = newValue }
    }
}
```

As propriedades `altura` e `largura` armazenam uma instância do _property wrapper_, `DozeOuMenos`. O _getter_ e o _setter_ para `altura` e `largura` envolvem o acesso à propriedade `wrappedValue`.

### Configurando valores iniciais para _wrapped properties_

O código nos exemplos acima define o valor inicial para a propriedade encapsulada dando a `numero` um valor inicial na definição de `DozeOuMenos`. O código que usa esse _property wrapper_ não pode especificar um valor inicial diferente para uma propriedade que é encapsulada por `DozeOuMenos` — por exemplo, a definição de `PequenoRetangulo` não pode fornecer valores iniciais de `altura` ou `largura`. Para dar suporte à configuração de um valor inicial ou outra personalização, o _property wrapper_ precisa adicionar um inicializador. Aqui está uma versão expandida do `DozeOuMenos` chamada `NumeroPequeno` que define os inicializadores que definem o _wrapped value_ e o valor máximo:

``` swift
@propertyWrapper
struct NumeroPequeno {
    private var maximo: Int
    private var numero: Int

    var wrappedValue: Int {
        get { return numero }
        set { numero = min(newValue, maximo) }
    }

    init() {
        maximo = 12
        numero = 0
    }

    init(wrappedValue: Int) {
        maximo = 12
        numero = min(wrappedValue, maximo)
    }

    init(wrappedValue: Int, maximo: Int) {
        self.maximo = maximo
        numero = min(wrappedValue, maximo)
    }
}
```

A definição de `NumeroPequeno` inclui três inicializadores — `init()`, `init(wrappedValue:)` e `init(wrappedValue:maximo:)` — que os exemplos abaixo usam para definir o _wrapped value_ e o valor máximo.

Quando você aplica um _wrapper_ a uma propriedade e não especifica um valor inicial, o Swift usa o inicializador `init()` para configurar o _wrapper_. Por exemplo:

``` swift
struct RetanguloZero {
    @NumeroPequeno var altura: Int
    @NumeroPequeno var largura: Int
}

var retanguloZero = RetanguloZero()
print(retanguloZero.altura, retanguloZero.largura)
// Imprime "0 0"
```

As instâncias de `NumeroPequeno` que envolvem `altura` e `largura` são criadas chamando `NumeroPequeno()`. O código dentro desse inicializador define o valor encapsulado inicial e o valor máximo inicial, usando os valores padrão de zero e `12`. O _property wrapper_ ainda fornece todos os valores iniciais, como o exemplo anterior que usou `DozeOuMenos` em `PequenoRetangulo`. Ao contrário desse exemplo, `NumeroPequeno` também suporta a escrita desses valores iniciais como parte da declaração da propriedade.

Quando você especifica um valor inicial para a propriedade, o Swift usa o inicializador `init(wrappedValue:)` para configurar o _wrapper_. Por exemplo:

``` swift
struct RetanguloUnitario {
    @NumeroPequeno var altura: Int = 1
    @NumeroPequeno var largura: Int = 1
}

var retanguloUnitario = RetanguloUnitario()
print(retanguloUnitario.altura, retanguloUnitario.largura)
// Imprime "1 1"
```

Quando você escreve `= 1` em uma propriedade com um _wrapper_, isso é traduzido em uma chamada para o inicializador `init(wrappedValue:)`. As instâncias de `NumeroPequeno` que envolvem `altura` e `largura` são criadas chamando `NumeroPequeno(wrappedValue: 1)`. O inicializador usa o _wrapped value_ especificado aqui e usa o valor máximo padrão de `12`.

Quando você escreve argumentos entre parênteses após o atributo customizado, o Swift usa o inicializador que aceita esses argumentos para configurar o _wrapper_. Por exemplo, se você fornecer um valor inicial e um valor máximo, o Swift usará o inicializador `init(wrappedValue:maximo:)`:

``` swift
struct RetanguloEstreito {
    @NumeroPequeno(wrappedValue: 2, maximo: 5) var altura: Int
    @NumeroPequeno(wrappedValue: 3, maximo: 4) var largura: Int
}

var retanguloEstreito = RetanguloEstreito()
print(retanguloEstreito.altura, retanguloEstreito.largura)
// Imprime "2 3"

retanguloEstreito.altura = 100
retanguloEstreito.largura = 100
print(retanguloEstreito.altura, retanguloEstreito.largura)
// Imprime "5 4"
```

A instância de `NumeroPequeno` que agrupa a `altura` é criada chamando `NumeroPequeno(wrappedValue: 2, maximo: 5)` e a instância que agrupa a `largura` é criada chamando `NumeroPequeno(wrappedValue: 3, maximo: 4)`.

Ao incluir argumentos no _property wrapper_, você pode configurar o estado inicial no _wrapper_ ou passar outras opções para o _wrapper_ quando ele for criado. Essa sintaxe é a maneira mais geral de usar um _property wrapper_. Você pode fornecer quaisquer argumentos necessários para o atributo e eles são passados ​​para o inicializador.

Ao incluir argumentos de _property wrapper_, você também pode especificar um valor inicial usando atribuição. O Swift trata a atribuição como um argumento `wrappedValue` e usa o inicializador que aceita os argumentos que você inclui. Por exemplo:

``` swift
struct RetanguloMisto {
    @NumeroPequeno var altura: Int = 1
    @NumeroPequeno(maximo: 9) var largura: Int = 2
}

var retanguloMisto = RetanguloMisto()
print(retanguloMisto.altura)
// Imprime "1"

retanguloMisto.altura = 20
print(retanguloMisto.altura)
// Imprime "12"
```

A instância de `NumeroPequeno` que agrupa a `altura` é criada chamando `NumeroPequeno(wrappedValue: 1)`, que usa o valor máximo padrão de `12`. A instância que agrupa a `largura` é criada chamando `NumeroPequeno(wrappedValue: 2, maximo: 9)`.

## Type Properties _(Propriedades de Tipo)_

Propriedades de instância são propriedades que pertencem a uma instância de um tipo específico. Toda vez que você cria uma nova instância desse tipo, ela tem seu próprio conjunto de valores de propriedade, separado de qualquer outra instância.

Você também pode definir propriedades que pertencem à própria definição do tipo, e não a qualquer instância desse tipo. Sempre haverá apenas uma cópia dessas propriedades, não importa quantas instâncias desse tipo você crie. Esses tipos de propriedades são chamados de _type properties_ (ou propriedades de tipo).

As propriedades de tipo são úteis para definir valores que são universais para todas as instâncias de um tipo específico, como uma propriedade constante que todas as instâncias podem usar (como uma constante estática em C) ou uma propriedade variável que armazena um valor global para todas as instâncias desse tipo (como uma variável estática em C).

As propriedades armazenadas de tipo podem ser variáveis ​​ou constantes. Propriedades computadas de tipo são sempre declaradas como propriedades variáveis, da mesma forma que propriedades computadas de instância.

> Nota: Ao contrário das propriedades armazenadas de instância, você deve sempre atribuir um valor padrão às propriedades armazenadas de tipo . Isso ocorre porque o próprio tipo não possui um inicializador que possa atribuir um valor a uma propriedade armazenada de tipo no momento da inicialização.
>
>As propriedades armazenadas de tipo são inicializadas de forma _lazy_ em seu primeiro acesso. Eles são garantidos para serem inicializados apenas uma vez, mesmo quando acessados ​​por várias threads simultaneamente, e não precisam ser marcados com o modificador `lazy`.

### Sintaxe das propriedades de tipo

Em C e Objective-C, você define constantes e variáveis ​​estáticas associadas a um tipo como variáveis ​​estáticas globais. No Swift, no entanto, as propriedades de tipo são escritas como parte da definição do tipo, dentro das chaves externas do tipo, e cada propriedade de tipo é explicitamente delimitada para o tipo que ela suporta.

Você define as propriedades de tipo com a palavra-chave `static`. Para propriedades computadas de tipo em classes, você pode usar a palavra-chave `class` para permitir que as subclasses substituam a implementação da superclasse. O exemplo abaixo mostra a sintaxe para propriedades de tipo armazenadas e computadas:

``` swift
struct UmaStruct {
    static var propriedadeArmazenadaDeTipo = "Algum valor"
    static var propriedadeComputadaDeTipo: Int {
        return 1
    }
}

print(UmaStruct.propriedadeArmazenadaDeTipo)
// Imprime "Algum valor"

print(UmaStruct.propriedadeComputadaDeTipo)
// Imprime "1"

enum UmaEnum {
    static var propriedadeArmazenadaDeTipo = "Algum valor"
    static var propriedadeComputadaDeTipo: Int {
        return 6
    }
}

print(UmaEnum.propriedadeArmazenadaDeTipo)
// Imprime "Algum valor"

print(UmaEnum.propriedadeComputadaDeTipo)
// Imprime "6"

class UmaClasse {
    static var propriedadeArmazenadaDeTipo = "Algum valor"
    static var propriedadeComputadaDeTipo: Int {
        return 27
    }
    class var propriedadeComputadaDeTipoSobrescrevivel: Int {
        return 107
    }
}

print(UmaClasse.propriedadeArmazenadaDeTipo)
// Imprime "Algum valor"

print(UmaClasse.propriedadeComputadaDeTipo)
// Imprime "27"

print(UmaClasse.propriedadeComputadaDeTipoSobrescrevivel)
// Imprime "107"
```

> Nota: Os exemplos de propriedades computadas de tipo acima são para propriedades computadas de tipo somente leitura, mas você também pode definir propriedades computadas de tipo de leitura/escrita com a mesma sintaxe das propriedades computadas de instância.

> Nota: Assim como para as propriedade a palavra-chave `static` também é aplicável para funções. Uma `static func` segue a mesma linha de raciocínio e se refere a um comportamento adicionado ao tipo, sendo comum à todas as intâncias. Por este motivo, em uma `static func` é impossível fazer referência à propriedades e métodos de instância, por não ter acesso a uma em seu contexto de execução.
