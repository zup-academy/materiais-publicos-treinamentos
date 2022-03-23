# Enums, Structs e Classes

>Esse material teórico foi adaptado da fonte original disponível em https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html

Swift oferece algumas formas de construir tipos customizados. Além das já conhecidas classes, veremos neste documento como se utilizar de _enumerations_ e _structs_ para aumentar suas possibilidades.

## Enumerations

Uma _enumeration_ (_enum_ ou enumeração) define um tipo comum para um grupo de valores relacionados e permite que você trabalhe com esses valores de maneira segura em seu código.

Se você estiver familiarizado com C, saberá que as _enums_ C atribuem nomes relacionados a um conjunto de valores inteiros. As _enums_ em Swift são muito mais flexíveis e não precisam fornecer um valor para cada caso da _enum_. Se um valor (conhecido como _raw value_, valor bruto) for fornecido para cada caso de _enum_, o valor poderá ser uma string, um caractere ou um valor de qualquer tipo inteiro ou de ponto flutuante.

Como alternativa, os casos de _enum_ podem especificar valores associados de qualquer tipo a serem armazenados junto com cada valor de caso diferente, assim como as _unions_ ou variantes fazem em outras linguagens. Você pode definir um conjunto comum de casos relacionados como parte de uma _enum_, cada um com um conjunto diferente de valores de tipos apropriados associados a ele.

_Enums_ em Swift são tipos de primeira classe por si só. Eles adotam muitos recursos tradicionalmente suportados apenas por classes, como propriedades computadas para fornecer informações adicionais sobre o valor atual da _enum_ e métodos de instância para fornecer funcionalidade relacionada aos valores que a _enum_ representa. As _enums_ também podem definir inicializadores para fornecer um valor para um caso inicial; podem ser estendidos para expandir sua funcionalidade além de sua implementação original; e pode estar em conformidade com protocolos para fornecer a padronização necessária para alguma funcionalidade.

Para obter mais informações sobre esses recursos, veremos adiante mais sobre propriedades, métodos e outras funcionalidades.

### Sintaxe das _Enums_

Você introduz _enums_ com a palavra-chave `enum` e coloca sua definição inteira dentro de um par de chaves:

``` swift
enum MinhaEnum {
    // definição da enum vai aqui
}
```

Aqui está um exemplo para os quatro pontos principais de uma bússola:

``` swift
enum PontoDaBussola {
    case norte
    case sul
    case leste
    case oeste
}
```

Os valores definidos em uma _enum_ (como norte, sul, leste e oeste) são seus casos de enumeração. Você usa a palavra-chave case para introduzir novos casos de enumeração.

> Nota: Os casos de enumeração Swift não têm um valor inteiro definido por padrão, ao contrário de linguagens como C e Objective-C. No exemplo `PontoDaBussola` acima, norte, sul, leste e oeste não são implicitamente iguais a `0`, `1`, `2` e `3`. Em vez disso, os diferentes casos de enumeração são valores por si só, com um tipo de PontoDaBussola explicitamente definido.

Vários casos podem aparecer em uma única linha, separados por vírgulas:

``` swift
enum Planeta {
    case mercurio, venus, terra, marte, jupiter, saturno, urano, netuno
}
```

Cada definição de _enum_ define um novo tipo. Como outros tipos em Swift, seus nomes (como PontoDaBussola e Planeta) começam com uma letra maiúscula. Dê a tipos definidos por _enums_ nomes singulares em vez de plurais, para que sejam lidos como auto-evidentes:

``` swift
var direcaoASeguir = PontoDaBussola.oeste
```

O tipo de `direcaoASeguir` é inferido quando é inicializado com um dos valores possíveis de PontoDaBussola. Uma vez que `direcaoASeguir` é declarado como um `PontoDaBussola`, você pode defini-lo para um valor de `PontoDaBussola` diferente usando uma sintaxe de ponto simplificada:

``` swift
direcaoASeguir = .leste
```

O tipo de `direcaoASeguir` já é conhecido e, portanto, você pode descartar o tipo ao definir seu valor. Isso torna o código altamente legível ao trabalhar com valores de _enum_ explicitamente tipados.

### Correspondência de valores de _enum_ em estruturas _switch_

Você pode combinar valores de enumeração individuais com uma instrução switch:

``` swift
direcaoASeguir = .sul

switch direcaoASeguir {
case .norte:
    print("Muitos planetas tem norte")
case .sul:
    print("Cuidado com os pinguins")
case .leste:
    print("Onde o sol nasce")
case .west:
    print("Onde os céus são azuis")
}
// Imprime "Cuidado com os pinguins"
```

Você pode ler este código como:

“Considere o valor de `direcaoASeguir`. No caso de ser igual a `.norte`, imprima "Muitos planetas tem norte". Caso seja igual a `.sul`, imprima "Cuidado com os pinguins".

…e assim por diante.

Conforme descrito em materiais anteriores, uma instrução `switch` deve ser exaustiva ao considerar os casos de uma _enum_. Se o caso de `.oeste` for omitido, esse código não compila, porque não considera a lista completa de casos do `PontoDaBussola`. A exigência de exaustividade garante que os casos de _enum_ não sejam omitidos acidentalmente.

Quando não for apropriado fornecer um `case` para cada caso de uma _enum_, você pode fornecer um caso padrão para cobrir todos os casos que não são abordados explicitamente:

``` swift
let algumPlaneta = Planeta.terra

switch algumPlaneta {
case .terra:
    print("O mais gentil")
case:
    print("Não é um lugar seguro para humanos")
}
// Imprime "O mais gentil"
```

### Iterando sobre casos de uma _enum_ 

Para algumas _enums_, é útil ter uma coleção de todos os seus casos. Você habilita isso escrevendo `: CaseIterable` após o nome da _enum_. O Swift então expõe uma coleção de todos os casos como uma propriedade `allCases` no tipo da _enum_. Aqui está um exemplo:

``` swift
enum Bebida: CaseIterable {
    case cafe, cha, suco
}

let numeroDeBebidas = Bebida.allCases.count
print("\(numeroDeBebidas) bebidas disponíveis")
// Imprime "3 bebidas disponíveis"
```

No exemplo acima, você escreve `Bebida.allCases` para acessar uma coleção que contém todos os casos da _enum_ `Bebida`. Você pode usar `allCases` como qualquer outra coleção — os elementos da coleção são instâncias do tipo da _enum_, portanto, neste caso, são valores de `Bebida`. O exemplo acima conta quantos casos existem, e o exemplo abaixo usa um loop para iterar sobre todos os casos.

``` swift
for bebida in Bebida.allCases {
    print(bebida)
}
// cafe
// che
// suco
```

A sintaxe usada nos exemplos acima marca a _enum_ como em conformidade com o protocolo `CaseIterable`. Veremos na próxima seção tudo sobre coleções, formas de iteração e protocolos. Tudo a seu tempo!

### Valores associados (_Associated Values_)

Os exemplos na seção anterior mostram como os casos de uma _enum_ são um valor definido (e tipado) por si só. Você pode definir uma constante ou variável para `Planeta.terra` e verificar esse valor posteriormente. No entanto, às vezes é útil poder armazenar valores de outros tipos ao lado desses valores de caso. Essas informações adicionais são chamadas de valores associados (ou _associated values_) e variam cada vez que você usa esse caso como um valor em seu código.

Você pode definir _enums_ Swift para armazenar valores associados de qualquer tipo, e os tipos dos valores podem ser diferentes para cada caso da enumeração, se necessário. _Enums_ semelhantes a essas são conhecidas como uniões discriminadas, uniões tagueadas ou variantes em outras linguagens de programação.

Por exemplo, suponha que um sistema de rastreamento de estoque precise rastrear produtos por dois tipos diferentes de código de barras. Alguns produtos são rotulados com códigos de barras 1D no formato UPC, que usa os números de 0 a 9. Cada código de barras possui um dígito do sistema numérico, seguido por cinco dígitos do código do fabricante e cinco dígitos do código do produto. Estes são seguidos por um dígito de verificação para verificar se o código foi digitalizado corretamente:

<p align="center">
<img alt="Imagem de exemplo de um código de barras seguindo o padrão citado acima" src="https://docs.swift.org/swift-book/_images/barcode_UPC_2x.png" width="35%" />
</p>

Outros produtos são rotulados com códigos de barras 2D em formato de _QR code_, que podem usar qualquer caractere ISO 8859-1 e podem codificar uma string de até 2.953 caracteres:

<p align="center">
<img alt="Imagem de exemplo de um QR code seguindo o padrão citado acima" src="https://docs.swift.org/swift-book/_images/barcode_QR_2x.png" width="20%" />
</p>

É conveniente para um sistema de rastreamento de inventário armazenar códigos de barras UPC como uma tupla de quatro números inteiros e códigos de barras de código QR como uma string de qualquer comprimento.

Em Swift, uma _enum_ para definir códigos de barras de produtos de qualquer tipo pode ser assim:

``` swift 
enum CodigoDeBarras {
    case upc(Int, Int, Int, Int)
    case qrCode(String)
}
```

Isso pode ser lido como:

“Defina um tipo de _enum_ chamado `CodigoDeBarras`, que pode ter um valor de `upc` com um valor associado do tipo `(Int, Int, Int, Int)` ou um valor de `qrCode` com um valor associado do tipo `String`.”

Essa definição não fornece nenhum valor real de `Int` ou `String` — apenas define o tipo de valores associados que as constantes e variáveis ​​de código de barras podem armazenar quando são iguais a `CodigoDeBarras.upc` ou `CodigoDeBarras.qrCode`.

Você pode então criar novos códigos de barras usando qualquer um dos tipos:

``` swift
var codidoDeBarrasDoProduto = CodigoDeBarras.upc(8, 85909, 51226, 3)
```

Este exemplo cria uma nova variável chamada `codidoDeBarrasDoProduto` e atribui a ela um valor de `CodigoDeBarras.upc` com um valor de tupla associado de `(8, 85909, 51226, 3)`.

Você pode atribuir ao mesmo produto um tipo diferente de código de barras, alterando seu estado:

``` swift
codidoDeBarrasDoProduto = .qrCode("ABCDEFGHIJKLMNOP")
```

Neste ponto, o `CodigoDeBarras.upc` original e seus valores inteiros são substituídos pelo novo `CodigoDeBarras.qrCode` e seu valor string. Constantes e variáveis ​​do tipo `CodigoDeBarras` podem armazenar um `.upc` ou um `.qrCode` (junto com seus valores associados), mas podem armazenar apenas um deles por vez.

Você pode verificar os diferentes tipos de código de barras usando uma instrução `switch`, semelhante ao exemplo em [Correspondência de valores de _enum_ em estruturas _switch_](#correspondência-de-valores-de-enum-em-estruturas-switch). Desta vez, no entanto, os valores associados são extraídos como parte da instrução switch. Você extrai cada valor associado como uma constante (com o prefixo let) ou uma variável (com o prefixo var) para uso no corpo do `switch case`:

``` swift
switch codigoDeBarrasDoProduto {
case .upc(let numeroDoSistema, let fabricante, let produto, let verificacao):
    print("UPC: \(numeroDoSistema), \(fabricante), \(produto), \(verificacao).")
case .qrCode(let codigoDoProduto):
    print("QR code: \(codigoDoProduto).")
}
// Imprime "QR code: ABCDEFGHIJKLMNOP."
```

Se todos os valores associados para um caso de enumeração forem extraídos como constantes, ou se todos forem extraídos como variáveis, você poderá colocar uma única anotação var ou let antes do nome do caso, por questões de brevidade:

``` swift
switch codigoDeBarrasDoProduto {
case let .upc(numeroDoSistema, fabricante, produto, verificacao):
    print("UPC : \(numeroDoSistema), \(fabricante), \(produto), \(verificacao).")
case let .qrCode(codigoDoProduto):
    print("QR code: \(codigoDoProduto).")
}
// Imprime "QR code: ABCDEFGHIJKLMNOP."
```

### Valores brutos (_Raw Values_)

O exemplo de código de barras em [Valores Associados](#valores-associados-associated-values) mostra como os casos de uma _enum_ podem declarar que armazenam valores associados de diferentes tipos. Como alternativa aos valores associados, os casos de enumeração podem ser pré-preenchidos com valores padrão (chamados _raw values_, ou valores brutos), que são todos do mesmo tipo.

Aqui está um exemplo que armazena valores ASCII brutos ao lado de casos de enumeração nomeados:

``` swift
enum CaracteresDeControleASCII: Character {
    case tab = "\t"
    case quebraDeLinha = "\n"
    case retornoDeCarro = "\r"
}
```

Aqui, os _raw values_ para uma _enum_ chamada `CaracteresDeControleASCII` são definidos como do tipo `Character` e são definidos para alguns dos caracteres de controle ASCII mais comuns.

Os _raw values_ podem ser strings, caracteres ou qualquer um dos tipos de número inteiro ou de ponto flutuante. Cada valor bruto deve ser exclusivo em sua declaração de _enum_.

>Nota: Os _raw values_ não são os mesmos que os valores associados. Os valores brutos são definidos para valores pré-preenchidos quando você define pela primeira vez a enumeração em seu código, como os três códigos ASCII acima. O valor bruto para um caso de enumeração específico é sempre o mesmo. Os valores associados são definidos quando você cria uma nova constante ou variável com base em um dos casos da enumeração e podem ser diferentes cada vez que você faz isso.

#### Valores brutos atribuídos implicitamente

Ao trabalhar com _enums_ que armazenam valores brutos inteiros ou de string, você não precisa atribuir explicitamente um valor bruto para cada caso. Quando você não faz isso, o Swift atribui automaticamente os valores para você.

Por exemplo, quando números inteiros são usados ​​para valores brutos, o valor implícito para cada caso é um a mais que o caso anterior. Se o primeiro caso não tiver um valor definido, seu valor será `0`.

A _enum_ abaixo é um refinamento da enumeração `Planeta` anterior, com valores brutos inteiros para representar a ordem de cada planeta no sistema solar:

``` swift
enum Planeta: Int {
    case mercurio = 1, venus, terra, marte, jupiter, saturno, urano, netuno
}
```

No exemplo acima, `Planeta.mercurio` tem um valor bruto explícito de `1`, `Planeta.venus` tem um valor bruto implícito de `2` e assim por diante.

Quando strings são usadas para valores brutos, o valor implícito para cada caso é o texto do nome desse caso.

A _enum_ abaixo é um refinamento da enumeração `PontoDaBussola` anterior, com valores brutos de string para representar o nome de cada direção:

``` swift
enum PontoDaBussola: String {
    case norte, sul, leste, oeste
}
```

No exemplo acima, `PontoDaBussola.sul` tem um valor bruto implícito de `"sul"` e assim por diante.

Você acessa o valor bruto de um caso de _enum_ com sua propriedade `rawValue`:

``` swift
let ordemDaTerra = Planeta.terra.rawValue
// ordemDaTerra é 3

let direcaoDoPorDoSol = PontoDaBussola.oeste.rawValue
// direcaoDoPorDoSol é "oeste"
```

#### Inicializando a partir de um valor bruto

Se você definir uma _enum_ com um tipo de valor bruto, a _enum_ recebe automaticamente um inicializador que recebe um valor do mesmo tipo do valor bruto (como um parâmetro chamado `rawValue`) e retorna um caso da _enum_ ou `nil`. Você pode usar esse inicializador para tentar criar uma nova instância da _enum_.

Este exemplo identifica Urano a partir de seu valor bruto `7`:

``` swift
let possivelPlaneta = Planeta(rawValue: 7)
// possivelPlaneta é do tipo Planeta? e é igual a Planeta.urano
```

No entanto, nem todos os valores `Int` possíveis encontrarão um planeta correspondente. Por isso, o inicializador de valor bruto sempre retorna um opcional para o caso da _enum_. No exemplo acima, `possivelPlaneta` é do tipo `Planeta?`, ou “Planeta opcional”.

> Nota: O inicializador por _raw value_ é um inicializador com falha, porque nem todo valor bruto retornará um caso da _enum_.

Se você tentar encontrar um planeta com uma posição `11`, o valor opcional do Planeta retornado pelo inicializador de _raw value_ será `nil`:

``` swift
let posicao = 11

if let algumPlaneta = Planeta(rawValue: posicao) {
    switch algumPlaneta {
    case .terra:
        print("O mais gentil")
    default:
        print("Não é um lugar seguro para humanos")
    }
} else {
    print("Não há um planeta na posição \(posicao)")
}
// Imprime "Não há um planeta na posição 11"
```

Este exemplo usa um _optional binding_ para tentar acessar um planeta com um valor bruto de 11. A instrução `if let algumPlaneta = Planeta(rawValue: 11)` cria um Planeta opcional e define `algumPlaneta` para o valor desse Planeta opcional se ele puder ser recuperado. Nesse caso, não é possível recuperar um planeta com uma posição de 11 e, portanto, a _branch_ de código do `else` é executada.
