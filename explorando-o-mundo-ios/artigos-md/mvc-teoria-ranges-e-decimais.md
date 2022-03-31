# Outros operadores e tipos: Ranges e Decimais

> Esse material teórico foi adaptado da fonte original disponível em: 
>   * https://docs.swift.org/swift-book/LanguageGuide/BasicOperators.html
>   * https://developer.apple.com/documentation/swift/range
>   * https://developer.apple.com/documentation/swift/closedrange
>   * https://developer.apple.com/documentation/foundation/decimal
>   * https://developer.apple.com/documentation/foundation/nsdecimalnumber

## Operadores de Range

O Swift inclui vários operadores de _range_, que são atalhos para expressar um intervalo de valores.

### Operador de Range Fechado _(Closed Range)_

O operador de _range_ fechado (`a...b`) define um intervalo que vai de `a` a `b` e **inclui** os valores `a` e `b`. O valor de `a` não deve ser maior que `b`.

O operador de _range_ fechado é útil ao iterar em um intervalo no qual você deseja que todos os valores sejam usados, como em um laço de repetição, por exemplo:

``` swift
for index in 1...5 {
    print("\(index) vezes 5 é \(index * 5)")
}

// 1 vezes 5 é 5
// 2 vezes 5 é 10
// 3 vezes 5 é 15
// 4 vezes 5 é 20
// 5 vezes 5 é 25
```

> Nota: Obteremos mais informações sobre como iterar sobre coleções mais à frente.

### Operador de Range Semi-aberto _(Half-Open Range)_

O operador de _range_ semi-aberto `(a..<b)` define um intervalo que vai de `a` a `b`, mas **não inclui** `b`. Se diz semi-aberto porque contém seu valor inicial, mas não seu valor final. Assim como no operador de _range_ fechado, o valor de `a` não deve ser maior que `b`. Se o valor de `a` for igual a `b`, o _range_ resultante será vazio.

_Ranges_ semi-abertos são particularmente úteis quando você trabalha com listas baseadas em zero, como arrays, onde é útil contar até (mas não incluir) o comprimento da lista:

``` swift
let nomes = ["Ana", "Alex", "Bruno", "José"]
let contagem = nomes.count

for i in 0..<contagem {
    print("Pessoa \(i + 1) é chamada \(nomes[i])")
}
// A pessoa 1 se chama Ana
// A pessoa 2 se chama Alex
// A pessoa 3 se chama Bruno
// A pessoa 4 é chamada de José
```

Observe que a lista contém quatro itens, mas `0..<contagem` conta apenas até `3` (o índice do último item), porque é um _range_ semi-aberto. 

> Nota: Obteremos mais informações sobre array mais à frente.

### Ranges Unilaterais _(One-Sided)_

O operador de _range_ fechado tem uma forma alternativa para _ranges_ que continuam o mais longe possível em uma direção - por exemplo, um _range_ que inclui todos os elementos de uma lista do índice `2` até o final da lista. Nesses casos, você pode omitir o valor de um lado do operador de _range_. Esse tipo de intervalo é chamado de _range_ unilateral _(one-sided)_ porque o operador tem um valor em apenas um lado. Por exemplo:

``` swift
for nome in nomes[2...] {
    print(nome)
}
// Bruno
// José

for nome in nomes[...2] {
    print(nome)
}
// Ana
// Alex
// Bruno
```

O operador de _range_ semi-aberto também tem uma forma unilateral que é escrita apenas com seu valor final. Assim como quando você inclui um valor em ambos os lados, o valor final não faz parte do intervalo. Por exemplo:

``` swift
for nome in nomes[..<2] {
    print(nome)
}
// Ana
// Alex
```

_Ranges_ unilaterais podem ser usados ​​em outros contextos. Você não pode iterar em um _range_ unilateral que omite um valor inicial, porque não está claro onde a iteração deve começar. Você pode iterar em um _range_ unilateral que omite seu valor final; no entanto, como o intervalo continua indefinidamente, certifique-se de adicionar uma condição final explícita para a estrutura de repetição. Você também pode verificar se um _range_ unilateral contém um valor específico, conforme mostrado no código abaixo.

``` swift
let range = ...5
range.contains(7) // falso
range.contains(4) // verdadeiro
range.contains(-1) // verdadeiro
```

### Outras utilizações comuns de _ranges_

Passagem de valor para configuração de funções úteis como em:

1. Execução controlada do processamento de requisições HTTP, com validação de _status codes_ para respostas.

    ``` swift
        request(requestConfig)
            .validate(statusCode: 200..<300) // half-open range
            .response { ... }
    ```

1. Obtenção de valores randômicos dentro de um intervalo, como ao simulador o lançar de um dado.

    ``` swift
    let numeroSorteado = Int.random(in: 1...6) // closed range
    ```

## Outros tipos de dados

### Range

Um intervalo semi-aberto de um limite inferior até, mas não incluindo, um limite superior.

#### Declaração

``` swift
@frozen struct Range<Bound> where Bound : Comparable
```

#### Visão Geral

Você cria uma instância de `Range` usando o operador de _range_ semi-aberto (`..<`).

``` swift
let abaixoDeCinco = 0.0..<5.0
```

Você pode usar uma instância de `Range` para verificar rapidamente se um valor está contido em um determinado _range_ de valores. Por exemplo:

``` swift
abaixoDeCinco.contains(3.14)
// verdadeiro
abaixoDeCinco.contains(6.28)
// falso
abaixoDeCinco.contains(5.0)
// falso
```

As instâncias de _range_ podem representar um intervalo vazio, diferentemente de [`ClosedRange`](#closedrange).

``` swift
let vazio = 0.0..<0.0

vazio.contains(0.0)
// falso
vazio.isEmpty
// verdadeiro
```

#### Usando um _range_ como uma coleção de valores consecutivos

Quando um _range_ usa números inteiros como seus limites inferior e superior, ou qualquer outro tipo que esteja em conformidade com o protocolo `Strideable` - que define o comportamento de quem pode ser iterado através de passos contados por um inteiro - você pode usar esse _range_ em um _for-in_ ou com qualquer método de coleção ou sequência. Os elementos do _range_ são os valores consecutivos de seu limite inferior até, mas não incluindo, seu limite superior.

``` swift
for n in 3..<5 {
     print(n)
}
// Imprime "3"
// Imprime "4"
```

Nota: Como os tipos de ponto flutuante, como `Float` e `Double`, são seus próprios tipos `Stride`, eles não podem ser usados como limites de um intervalo contável. Se você precisar iterar sobre valores de ponto flutuante consecutivos, consulte a função `stride(from:to:by:)`.

#### Tópicos

##### Criando um _range_

Crie um novo _range_ usando o operador de _range_ semi-aberto (`..<`).

``` swift
static func ..< (Self, Self) -> Range<Self>
```
Retorna um _range_ semi-aberto que contém seu limite inferior, mas não seu limite superior.

##### Inspecionando um _range_

``` swift
var isEmpty: Bool
```
Um valor booleano que indica se o _range_ não contém elementos.

``` swift
let lowerBound: Bound
```
O limite inferior do _range_.

``` swift
let upperBound: Bound
```
O limite superior do _range_.

##### Verificação de contenção

``` swift
func contains(Bound) -> Bool
```
Retorna um valor booleano que indica se o elemento fornecido está contido no _range_.

``` swift
static func ~= (Range<Bound>, Bound) -> Bool
```
Retorna um valor booleano que indica se um valor está incluído em um intervalo.

> Nota: Para mais detalhes consulte a seção _Topics_ da documentação disponível no [portal de desenvolvedores da apple](https://developer.apple.com/documentation/swift/range).

### ClosedRange

Um intervalo de um limite inferior até um limite superior, incluído.

#### Declaração

``` swift
@frozen struct ClosedRange<Bound> where Bound : Comparable
```

#### Visão Geral

Você cria uma instância de `ClosedRange` usando o operador de intervalo fechado (`...`).

``` swift
let ateCinco = 0...5
```

Uma instância de `ClosedRange` contém seu limite inferior e seu limite superior.

``` swift
ateCinco.contains(3)
// verdadeiro
ateCinco.contains(10)
// falso
ateCinco.contains(5)
// verdadeiro
```

Como um intervalo fechado inclui seu limite superior, um intervalo fechado cujo limite inferior é igual ao limite superior contém esse valor. Portanto, uma instância de `ClosedRange` não pode representar um intervalo vazio.

``` swift
let zeroInclusive = 0...0
zeroInclusive.contains(0)
// verdadeiro
zeroInclusive.isEmpty
// falso
```

#### Usando um _range_ fechado como uma coleção de valores consecutivos

Quando um _range_ fechado usa números inteiros como seus limites inferior e superior, ou qualquer outro tipo que esteja em conformidade com o protocolo `Strideable`, você pode usar esse _range_ em um _for-in_ ou com qualquer método de coleção ou sequência. Os elementos do _range_ são os valores consecutivos de seu limite inferior até, inclusive, seu limite superior.

``` swift
for n in 3...5 {
    print(n)
}

// Imprime "3"
// Imprime "4"
// Imprime "5"
```

>Nota: Como os tipos de ponto flutuante, como `Float` e `Double`, são seus próprios tipos `Stride`, eles não podem ser usados como limites de um intervalo contável. Se você precisar iterar sobre valores de ponto flutuante consecutivos, consulte a função `stride(from:through:by:)`.

#### Tópicos

##### Criando um intervalo

Crie um novo intervalo usando o operador de _range_ fechado (`...`).

``` swift
static func ... (Self, Self) -> ClosedRange<Self>
```
Retorna um _range_ fechado que contém seus dois limites.

> Nota: Para mais detalhes consulte a seção _Topics_ da documentação disponível no [portal de desenvolvedores da apple](https://developer.apple.com/documentation/swift/closedrange).


### Decimal

Uma _struct_ que representa um número de base 10.

#### Declaração

``` swift
struct Decimal
```

#### Exemplos de utilizações comuns

##### Como representação de moeda

Embora possam existir outras possíveis soluções mais adequadas para domínios específicos, uma das possíveis aplicações comuns de `Decimal` é no contexto de representação de valores monetários.

Para que seja possível entender esta aplicação, voltamos um passo atrás. À primeira vista seria possível imaginar a utilização de tipos numéricos de ponto flutuante para tais representações. Mas casos simples (e clássicos) de uso nesse sentido podem demonstrar um problema das implementações de ponto flutuante bases das linguagens.

``` swift
let trintaCentavos = 0.1 + 0.2
print(trintaCentavos)

// Imprime 0.30000000000000004 ?
```

O exemplo acima demonstra um caso simples simulando a soma de duas parcelas monetárias, dez centavos e vinte centavos, que como se esperaria, resulta em trinta centavos. Porém os valores literais representados através de `Double` apresetam um resultando um pouco incomum de `0.30000000000000004`.

O problema se dá pela natureza da implementação de base para o número de ponto flutuante que tem dificuldades para representar números como `0.1`. A depender das capacidades do computador que você está utilizando o numero que terá no lugar será um arredondamento muito próximo do numero, o que traz a imprecisão em casos como acima. Estamos trabalhando essencialmente com uma representação de números binária.

<p align="center">
<img alt="Imagem com representação numérica binária" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-teoria-imagem-ponto-flutuante.jpeg?raw=true" width="75%" />
</p>

A utilização de `Decimal` nesse caso vem para evitar que detalhes de implementação como o citado acima interfiram no resultado esperado. Como citado acima um `Decimal` é uma _struct_ já projetada para representar um número denário (de representação na base 10).

``` swift
import Foundation

let trintaCentavos: Decimal = 0.1 + 0.2
print(trintaCentavos)

// Imprime 0.3
```

###### Implementações úteis

1. Conversor para obter um `Decimal` através de uma _string_:

    ``` swift
        static func paraDecimal(string: String) -> Decimal? {
            let formatter = NumberFormatter()
            formatter.numberStyle = .decimal
            formatter.locale = .init(identifier: "pt_BR") // ou .current
            
            return formatter.number(from: string)?.decimalValue
        }
    ```

1. Formatador de `Decimal` para _string_ com símbolo de moeda apropriado e casas decimais:

    ``` swift
        static func paraMoeda(decimal: Decimal) -> String {
            let formatador = NumberFormatter()
            formatador.numberStyle = .currency
            formatador.locale = Locale(identifier: "pt_BR") // ou .current
            
            return formatador.string(from: decimal as NSDecimalNumber)!
        }
    ```

> Nota: A classe `NumberFormatter` é definida pelo Foundation _framework_ em Objective-C, e, deste modo, seu método `string(from:)` opera sobre o tipo `NSNumber` definido na mesma linguagem (no geral o prefixo `NS` é uma boa pista sobre a origem no Objective-C). Para fins de interoperabilidade `Decimal` é a declaração da Swift para a classe Objective-C `NSDecimal`, e `NSDecimalNumber` usado no exemplo acima assegura a conversão para um tipo `NSNumber` equivalente. `NSDecimalNumber`, como a própria documentação apresenta, é um _wrapper_ para fazer aritmética sobre a base 10 e que faz uma ponte para a _struct_ Swift `Decimal`.

#### Tópicos

##### Criando um Decimal vazio

``` swift
init()
```
Cria um decimal inicializado em 0.

##### Criando um Decimal a partir de um Número de Ponto Flutuante

``` swift
init(Double)
```
Cria e inicializa um decimal com o valor de ponto flutuante fornecido.

``` swift
init(floatLiteral: Double)
```
Cria e inicializa um decimal com o valor de ponto flutuante fornecido.

##### Criando um Decimal a partir de um Inteiro

``` swift
init(Int)
```
Cria e inicializa um decimal com o valor inteiro fornecido.

##### Executando aritmética

``` swift
static func * (Decimal, Decimal) -> Decimal
```
Multiplica dois números decimais.

``` swift
static func *= (inout Decimal, Decimal)
```
Multiplica dois números decimais, armazenando o resultado no primeiro número.

``` swift
static func + (Decimal, Decimal) -> Decimal
```
Adiciona dois números decimais.

``` swift
static func += (inout Decimal, Decimal)
```
Adiciona dois números decimais, armazenando o resultado no primeiro número.

``` swift
static func - (Decimal, Decimal) -> Decimal
```
Subtrai um número decimal de outro.

``` swift
static func -= (inout Decimal, Decimal)
```
Subtrai um número decimal de outro, armazenando o resultado no primeiro número.

``` swift
static func / (Decimal, Decimal) -> Decimal
```
Divide um número decimal por outro.

``` swift
static func /= (inout Decimal, Decimal)
```
Divide um número decimal por outro, armazenando o resultado no primeiro número.

``` swift
func pow(Decimal, Int) -> Decimal
```
Retorna um número decimal elevado a uma determinada potência.

``` swift
func negate()
```
Nega este decimal.


##### Obtendo as características de um decimal

``` swift
var sign: FloatingPointSign
```
O sinal do decimal.

``` swift
var exponent: Int
```
O expoente do decimal.

``` swift
var significand: Decimal
```
O significando do decimal.

``` swift
var magnitude: Decimal
```
A magnitude deste decimal.

> Nota: Para mais detalhes consulte a seção _Topics_ da documentação disponível no [portal de desenvolvedores da apple](https://developer.apple.com/documentation/foundation/decimal).
