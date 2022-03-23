# Enums, Structs e Classes

>Esse material teórico foi adaptado das fontes originais disponíveis em:
>   * https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html
>   * https://docs.swift.org/swift-book/LanguageGuide/ClassesAndStructures.html
>   * https://developer.apple.com/documentation/swift/choosing_between_structures_and_classes

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

## Structs e Classes 

_Structs_ e Classes são construções flexíveis de propósito geral que se tornam os blocos de construção do código do seu programa. Você define propriedades e métodos para adicionar funcionalidade às suas estruturas e classes usando a mesma sintaxe usada para definir constantes, variáveis e funções.

>Nota: Uma instância de uma classe é tradicionalmente conhecida como um objeto. No entanto, as estruturas e classes do Swift são muito mais próximas em funcionalidade do que em outras linguagens, e grande parte deste capítulo descreve as funcionalidades que se aplicam tanto à instâncias de classe quanto à de _structs_. Por causa disso, o termo mais geral instância é usado.

### Comparando Structs e Classes

_Structs_ e classes em Swift têm muitas coisas em comum. Ambos podem:

* Definir propriedades para armazenar valores;
* Definir métodos para fornecer funcionalidade;
* Definir inicializadores para configurar seu estado inicial;
* Ser estendido para expandir sua funcionalidade além de uma implementação padrão;
* Estar em conformidade com protocolos para fornecer a padronização necessária para alguma funcionalidade;

As classes têm recursos adicionais que as _structs_ não têm:

* A herança permite que uma classe herde as características de outra.
* A conversão de tipo permite verificar e interpretar o tipo de uma instância de classe em tempo de execução.
* Os desinicializadores permitem que uma instância de uma classe libere quaisquer recursos atribuídos.
* A contagem de referência permite mais de uma referência a uma instância de classe.

Os recursos adicionais que as classes suportam têm o custo de maior complexidade. Como orientação geral, **prefira _structs_ porque elas são mais simples e apresentam formas mais seguras de modelar seus tipos e use classes apenas quando for apropriado ou necessário**. Na prática, isso significa que a maioria dos tipos de dados personalizados que você define serão _structs_ e _enums_. Para uma comparação mais detalhada, veja [Escolhendo entre structs e classes](#escolhendo-entre-structs-e-classes).

#### Sintaxe de definição

_Structs_ e classes têm uma sintaxe de definição semelhante. Você introduz _structs_ com a palavra-chave `struct` e classes com a palavra-chave `class`. Ambos colocam sua definição inteira dentro de um par de chaves:

``` swift
struct UmaStruct {
    // a definição da struct vai aqui
}

class UmaClasse {
    // a definição da classe vai aqui
}
```

>Nota: Sempre que você define uma nova _struct_ ou classe, você define um novo tipo Swift. Dê aos tipos nomes UpperCamelCase (como `UmaStruct` e `UmaClasse` aqui) para corresponder à capitalização dos tipos padrão do Swift (como `String`, `Int` e `Bool`). Dê nomes de propriedades e métodos lowerCamelCase (como `taxaDeAtualizacao` e `incrementaContador`) para diferenciá-los dos nomes de tipo.

Aqui está um exemplo de uma definição de _struct_ e uma definição de classe:

``` swift
struct Resolucao {
    var largura = 0
    var altura = 0
}

class ModoDeVideo {
    var resolucao = Resolucao()
    var entrelacada = false
    var taxaDeAtualizacao = 0.0
    var nome: String?
}
```

O exemplo acima define uma nova _struct_ chamada `Resolucao`, para descrever uma resolução de exibição baseada em pixels. Essa _struct_ tem duas propriedades armazenadas chamadas largura e altura.

O exemplo acima também define uma nova classe chamada `ModoDeVideo`, para descrever um modo específico para exibição de vídeo.

#### Instâncias de _Structs_ e Classes

A definição da _struct_ `Resolucao` e a definição da classe `ModoDeVideo` descrevem apenas o modelo de uma `Resolucao` ou `ModoDeVideo`. Eles próprios não descrevem uma resolução específica ou modo de vídeo. Para fazer isso, você precisa criar uma instância da _struct_ ou classe.

A sintaxe para criar instâncias é muito semelhante para _structs_ e classes:

``` swift 
let umaResolucao = Resolucao()
let umModoDeVideo = ModoDeVideo()
```

_Structs_ e classes usam a sintaxe do inicializador para novas instâncias. A forma mais simples de sintaxe do inicializador usa o nome do tipo da classe ou _struct_ seguido por parênteses vazios, como `Resolucao()` ou `ModoDeVideo()`. Isso cria uma nova instância da classe ou _struct_, com todas as propriedades inicializadas com seus valores padrão.

#### Inicializadores inferidos para _Structs_

Todas as _structs_ têm um inicializador gerado automaticamente, que você pode usar para inicializar as propriedades de novas instâncias da _struct_. Os valores iniciais para as propriedades da nova instância podem ser passados para o inicializador de membro por nome:

``` swift
let vga = Resolucao(largura: 640, altura: 480)
```

Ao contrário das _structs_, as instâncias de classe não recebem um inicializador padrão.

### _Structs_ e _Enums_ são _value types_ (tipos valor)

Um _value type_ (tipo valor) é um tipo cujo valor é copiado quando é atribuído a uma variável ou constante, ou quando é passado para uma função.

Na verdade, você tem usado _value types_ extensivamente nas seções anteriores. Todos os tipos básicos em Swift — inteiros, números de ponto flutuante, booleanos, strings e outras coleções — são _value types_ e são implementados como _structs_ por baixo dos panos.

Todas as _structs_ e _enums_ são _value types_ em Swift. Isso significa que todas as instâncias de _struct_ e _enum_ que você cria — e quaisquer _value types_ que elas tenham como propriedades — são sempre copiadas quando são passadas em seu código.

> Nota: As strings e outras coleções definidas pela biblioteca padrão, usam uma otimização para reduzir o custo de desempenho da cópia. Em vez de fazer uma cópia imediatamente, essas coleções compartilham a memória onde os elementos são armazenados entre a instância original e quaisquer cópias. Se uma das cópias da coleção for modificada, os elementos serão copiados imediatamente antes da modificação. O comportamento que você vê em seu código é sempre como se uma cópia ocorresse imediatamente.

Considere este exemplo, que usa a _struct_ `Resolucao` do exemplo anterior:

``` swift
let hd = Resolucao(largura: 1920, altura: 1080)
var cinema = hd
```

Este exemplo declara uma constante chamada `hd` e a define como uma instância `Resolucao` inicializada com a largura e a altura do vídeo full HD (1920 pixels de largura por 1080 pixels de altura).

Em seguida, ele declara uma variável chamada `cinema` e a define com o valor atual de `hd`. Como a `Resolucao` é uma _struct_, é feita uma cópia da instância existente e essa nova cópia é atribuída à `cinema`. Embora `hd` e `cinema` agora tenham a mesma largura e altura, são duas instâncias completamente diferentes nos bastidores.

Abaixo, a propriedade de `largura` da instância referenciada por `cinema` é alterada para ser a largura um pouco mais ampla do padrão 2K, usado para projeção de cinema digital (2048 pixels de largura e 1080 pixels de altura):

``` swift
cinema.largura = 2048
```

A verificação da propriedade `largura` de `cinema` mostra que realmente mudou para 2048:

``` swift
print("Cinema agora tem \(cinema.largura) pixels de largura")
// Imprime "Cinema agora tem 2048 pixels de largura"
```

No entanto, a propriedade `largura` da instância em `hd` original ainda tem o valor antigo de `1920`:

``` swift
print("hd ainda tem \(hd.largura) pixels de largura")
// Imprime "hd ainda tem 1920 pixels de largura"
```

Quando `cinema` recebeu o valor atual de `hd`, os valores armazenados em `hd` foram copiados para a nova instância de `Resolucao` associada à `cinema`. O resultado final foram duas instâncias completamente separadas que continham os mesmos valores numéricos. No entanto, por serem instâncias separadas, definir a largura para `cinema` como `2048` não afeta a largura armazenada em `hd`, conforme mostrado na figura abaixo:

<p align="center">
<img alt="Ilustração sobre o estado da memória com a cópia de valor proveniente da passagem de value types" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-teoria-imagem-structs-value-types.jpg?raw=true" width="75%" />
</p>

O mesmo comportamento se aplica a _enums_:

``` swift
enum PontoDaBussola {
     case norte, sul, leste, oeste
     
     mutating func vireParaONorte() {
         self = .norte
     }
}

var direcaoAtual = PontoDaBussola.oeste
let direcaoRecordada = direcaoAtual

direcaoAtual.vireParaONorte()

print("A direção atual é \(direcaoAtual)")
print("A direção recordada é \(direcaoRecordada)")

// Imprime "A direção atual é norte"
// Imprime "A direção recordada é oeste"
```

Quando `direcaoRecordada` recebe o valor de `direcaoAtual`, na verdade ele é definido como uma cópia desse valor. Alterar o valor de `direcaoAtual` posteriormente não afeta a cópia do valor original que foi armazenado em `direcaoRecordada`.

#### Modificando _value types_ em métodos de instância

_Structs_ e _Enums são _value types_. Por padrão, as propriedades de um _value type_ não podem ser modificadas de dentro de seus métodos de instância.

No entanto, se você precisar modificar as propriedades de sua _struct_ ou _enum_ em um método específico, poderá adotar a característica de mutabilidade para este método. O método pode então _mutar_ (ou seja, alterar durante o ciclo de vida da instância) o estado de suas propriedades de dentro do método, e quaisquer alterações feitas são gravadas de volta na estrutura original quando o método termina. Um método também pode atribuir uma instância completamente nova à sua propriedade `self` implícita, e essa nova instância substituirá a existente quando o método terminar.

Você pode optar por esse comportamento colocando a palavra-chave `mutating` antes da palavra-chave `func` para esse método:

``` swift
struct Ponto {
    var x = 0.0, y = 0.0

    mutating func movePor(x deltaX: Double, y deltaY: Double) {
        x += deltaX
        y += deltaY
    }
}

var algumPonto = Ponto(x: 1.0, y: 1.0)
algumPonto.movePor(x: 2.0, y: 3.0)

print("O ponto agora está em (\(algumPonto.x), \(algumPonto.y))")
// Imprime "O ponto agora está em (3.0, 4.0)"
```

A _struct_ `Ponto` acima define um método mutante `movePor(x:y:)`, que move uma instância de `Ponto` por uma certa quantidade. Em vez de retornar um novo ponto, esse método na verdade modifica o ponto no qual é chamado. A palavra-chave `mutating` é adicionada à sua definição para permitir que ela modifique suas propriedades.

Observe que você não pode chamar um método mutante em uma constante de um tipo definido por um_struct_, porque suas propriedades não podem ser alteradas, mesmo que sejam propriedades variáveis:

``` swift
let pontoFixo = Ponto(x: 3.0, y: 3.0)
pontoFixo.movePor(x: 2.0, y: 3.0)
// isso irá reportar um erro
```

#### Atribuindo à self dentro de um método _mutating_

Os métodos mutantes podem atribuir uma instância inteiramente nova à propriedade `self` implícita. O exemplo `Ponto` mostrado acima poderia ter sido escrito da seguinte maneira:

``` swift
struct Ponto {
    var x = 0.0, y = 0.0
    
    mutating func movePor(x deltaX: Double, y deltaY: Double) {
        self = Ponto(x: x + deltaX, y: y + deltaY)
    }
}
```

Esta versão do método mutante `movePor(x:y:)` cria uma nova estrutura cujos valores `x` e `y` são definidos para o local de destino. O resultado final de chamar essa versão alternativa do método será exatamente o mesmo que chamar a versão anterior.

Métodos mutantes para _enums_ podem definir o parâmetro `self` implícito para ser um caso diferente da mesma _enum_:

``` swift
enum InterruptorDeTresEstados {
    case desligado, baixo, alto
    
    mutating func proximo() {
        switch self {
        case .desligado:
            self = .baixo
        case .baixo:
            self = .alto
        case .alto:
            self = .desligado
        }
    }
}

var luzDoForno = InterruptorDeTresEstados.baixo

luzDoForno.proximo()
// luzDoForno agora é igual a .alto

luzDoForno.proximo()
// luzDoForno agora é igual a .desligado
```

Este exemplo define uma _enum_ para um interruptor de três estados que alterna entre estados de energia diferentes (`.desligado`, `.baixo` e `.alto`) toda vez que seu método `proximo()` é chamado.

### Classes são tipos referência _(reference types)_

Ao contrário dos _value types_, os _reference types_ não são copiados quando são atribuídos a uma variável ou constante ou quando são passados ​​a uma função. Em vez de uma cópia, é usada uma referência à mesma instância existente.

Aqui está um exemplo, usando a classe `ModoDeVideo` definida acima:

``` swift
let milEOitenta = ModoDeVideo()
milEOitenta.resolucao = hd
milEOitenta.entrelacada = true
milEOitenta.nome = "1080p"
milEOitenta.taxaDeAtualizacao = 25.0
```

Este exemplo declara uma nova constante chamada `milEOitenta` e a define para se referir a uma nova instância da classe `ModoDeVideo`. O modo de vídeo recebe uma cópia da resolução HD de `1920` por `1080` de antes. Ele está definido para ser entrelaçado, seu nome está definido como "1080p" e sua taxa de atualização de quadros está definida como `25.0` quadros por segundo.

Abaixo, `milEOitenta` é atribuído a uma nova constante, chamada `tambemMilEOitenta`, e a taxa de quadros de `tambemMilEOitenta` é modificada:

``` swift
let tambemMilEOitenta = milEOitenta
tambemMilEOitenta.frameRate = 30.0
```

Como as classes são _reference types_, `milEOitenta` e `tambemMilEOitenta` na verdade se referem à mesma instância de `ModoDeVideo`. Efetivamente, eles são apenas dois nomes diferentes para a mesma instância única, conforme mostrado na figura abaixo:

<p align="center">
<img alt="Ilustração sobre o estado da memória com a cópia da referencia apenas proveniente da passagem de um reference types" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-teoria-imagem-classes-reference-types.jpg?raw=true" width="75%" />
</p>

A verificação da propriedade `taxaDeAtualizacao` de `milEOitenta` mostra que ela relata corretamente a nova taxa de quadros de `30.0` da instância de `ModoDeVideo` subjacente:

``` swift
print("A propriedade taxaDeAtualizacao de milEOitenta agora é \(milEOitenta.taxaDeAtualizacao)")
// Imprime "A propriedade taxaDeAtualizacao de milEOitenta agora é 30.0"
```

Este exemplo também mostra como os tipos referência podem ser mais difíceis gerenciar em muitos casos. Se `milEOitenta` e `tambemMilEOitenta` estiverem distantes no código do seu programa, pode ser difícil encontrar todas as maneiras pelas quais o modo de vídeo pode ser alterado. Onde quer que você use `milEOitenta`, você também deve pensar no código que usa `tambemMilEOitenta` e vice-versa. Por outro lado, os _value types_ são mais fáceis de gerenciar porque todo o código que interage com o mesmo valor está próximo em seus arquivos de origem.

Observe que `milEOitenta` e `tambemMilEOitenta` são declarados como constantes, em vez de variáveis. No entanto, você ainda pode alterar `milEOitenta.taxaDeAtualizacao` e `tambemMilEOitenta.taxaDeAtualizacao` porque os valores das constantes `milEOitenta` e `tambemMilEOitenta` não mudam. As constantes `milEOitenta` e `tambemMilEOitenta` não "armazenam" a instância de `ModoDeVideo` - em vez disso, ambos se referem a uma instância de `ModoDeVideo` nos bastidores. É a propriedade `taxaDeAtualizacao` do `ModoDeVideo` subjacente que foi alterada, não os valores das referências constantes a esse `ModoDeVideo`.

> Nota: Usar _reference types_ para modelar os tipos personalizados da sua lógica/domínio pode tornar mais difícil o tracking do estado das instâncias e, por sua vez, das suas aplicações como um todo.

#### Operadores de identidade

Como as classes são tipos referência, é possível que várias constantes e variáveis ​​se refiram à mesma instância única de uma classe nos bastidores. (O mesmo não é verdade para _structs_ e _enums_, porque elas sempre são copiadas quando são atribuídas a uma constante ou variável, ou passadas a uma função.)

Às vezes, pode ser útil descobrir se duas constantes ou variáveis ​​se referem exatamente à mesma instância de uma classe. Para habilitar isso, o Swift fornece dois operadores de identidade:

* Idêntico a (===)
* Não idêntico a (!==)

Use estes operadores para verificar se duas constantes ou variáveis ​​se referem à mesma instância única:

``` swift
if milEOitenta === tambemMilEOitenta {
    print("milEOitentaighty e tambemMilEOitenta referem-se à mesma instância de ModoDeVideo.")
}
// Imprime "milEOitentaighty e tambemMilEOitenta referem-se à mesma instância de ModoDeVideo.""
```

Observe que _idêntico a_ (representado por três sinais de igual, ou `===`) não significa a mesma coisa que _igual a_ (representado por dois sinais de igual, ou `==`). _Idêntico a_ significa que duas constantes ou variáveis ​​do tipo de classe referem-se exatamente à mesma instância de classe. _Igual a_ significa que duas instâncias são consideradas iguais ou equivalentes em valor, para algum significado apropriado de igual, conforme definido por quem projetou do modelo.

Quando você define suas próprias _structs_ e classes personalizadas, é sua responsabilidade decidir o que se qualifica como duas instâncias iguais.


## Escolhendo entre _Structs_ e Classes

Decida como armazenar dados e modelar o comportamento.

### Visão geral

_Structs_ e classes são boas opções para armazenar dados e modelar o comportamento em seus aplicativos, mas suas semelhanças podem dificultar a escolha de uma sobre a outra. Assim, considere as recomendações a seguir para ajudar a escolher qual opção faz sentido ao adicionar um novo tipo ao seu aplicativo.

* [Use _structs_ por padrão.](#escolha-structs-por-padrão)
* [Use classes quando precisar de interoperabilidade com Objective-C.](#use-classes-quando-precisar-de-interoperabilidade-objective-c)
* [Use classes quando precisar controlar a identidade dos dados que está modelando.](#use-classes-quando-precisar-controlar-a-identidade)
* [Use _structs_ junto com protocolos para adotar um comportamento compartilhando implementação.](#use-structs-e-protocolos-para-modelar-a-herança-e-compartilhar-o-comportamento)

#### Escolha _Structs_ por Padrão

Use _structs_ para representar tipos comuns de dados. _Structs_ em Swift incluem muitos recursos que são limitados a classes em outras linguagens: elas podem incluir propriedades armazenadas, propriedades computadas e métodos. Além disso, _structs_ Swift podem adotar protocolos para ganhar comportamento por meio de implementações padrão. A biblioteca padrão do Swift e o Foundation usam _structs_ para tipos que você usa com frequência, como números, strings, arrays e dicionários.

O uso de _structs_ facilita o raciocínio sobre uma parte do seu código sem a necessidade de considerar todo o estado do seu aplicativo. Como as _structs_ são _value types_, ao contrário das classes, as alterações locais em uma _struct_ não são visíveis para o restante do aplicativo, a menos que você comunique intencionalmente essas alterações como parte do fluxo do aplicativo. Como resultado, você pode examinar uma seção de código e ter mais confiança de que as alterações nas instâncias dessa seção serão feitas explicitamente, em vez de serem feitas de forma invisível a partir de uma chamada de função tangencialmente relacionada.

#### Use classes quando precisar de interoperabilidade Objective-C

Se você usa uma API Objective-C que precisa processar seus dados ou precisa ajustar seu modelo de dados em uma hierarquia de classes existente definida em uma estrutura Objective-C, pode ser necessário usar classes e herança de classe para modelar seus dados. Por exemplo, muitos frameworks Objective-C expõem classes das quais você deve criar uma subclasses.

#### Use classes quando precisar controlar a identidade

As classes no Swift vêm com uma noção interna de identidade porque são _reference types_. Isso significa que quando duas instâncias diferentes de uma classe têm o mesmo valor para cada uma de suas propriedades armazenadas, elas ainda são consideradas diferentes pelo operador de identidade (===). Isso também significa que, quando você compartilha uma instância de classe em seu aplicativo, as alterações feitas nessa instância ficam visíveis para cada parte do seu código que contém uma referência a essa instância. Use classes quando precisar que suas instâncias tenham esse tipo de identidade. Casos de uso comuns são identificadores de arquivos, conexões de rede e intermediários de hardware compartilhados, etc.

Por exemplo, se você tiver um tipo que representa uma conexão de banco de dados local, o código que gerencia o acesso a esse banco de dados precisa de controle total sobre o estado do banco de dados conforme visualizado em seu aplicativo. É apropriado usar uma classe neste caso, mas certifique-se de limitar quais partes do seu aplicativo têm acesso ao objeto de banco de dados compartilhado.

> Importante: Trate a identidade com cuidado. O compartilhamento de instâncias de classe de forma generalizada em um aplicativo aumenta a probabilidade de erros de lógica. Você pode não antecipar as consequências de alterar uma instância altamente compartilhada, portanto, é mais trabalhoso escrever esse código corretamente.

#### Use _structs_ quando você não controla a identidade

Use _structs_ ao modelar dados que contenham informações sobre uma entidade com uma identidade que você não controla.

Em um aplicativo que consulta um banco de dados remoto, por exemplo, a identidade de uma instância pode ser de propriedade total de uma entidade externa e comunicada por um identificador. Se a consistência dos modelos de um aplicativo estiver armazenada em um servidor, você poderá modelar registros como _structs_ com identificadores. No exemplo abaixo, `jsonResponse` contém uma instância `RegistroDeCorrespondencia` codificada de um servidor:

``` swift
struct RegistroDeCorrespondencia {
    let id: Int
    var apelido: String
    var idDeCorrespondenciaRecomendado: Int
}

var meuRegistro = try JSONDecoder().decode(RegistroDeCorrespondencia.self, from: jsonResponse)
```

Alterações locais em tipos de modelo como `RegistroDeCorrespondencia` são úteis. Por exemplo, um aplicativo pode recomendar várias correspondencias diferentes em resposta ao feedback do usuário. Como a estrutura `RegistroDeCorrespondencia` não controla a identidade dos registros de banco de dados subjacentes, não há risco de que as alterações feitas nas instâncias `RegistroDeCorrespondencia` locais alterem acidentalmente os valores no banco de dados.

Se outra parte do aplicativo alterar `apelido` e enviar uma solicitação de alteração de volta ao servidor, a recomendação de correspondência rejeitada mais recentemente não será selecionada por engano para alteração. Como a propriedade `id` é declarada como uma constante, ela não pode ser alterada localmente. Como resultado, as solicitações ao banco de dados não alterarão acidentalmente o registro errado.

#### Use _structs_ e protocolos para modelar a herança e compartilhar o comportamento

_Structs_ e classes suportam uma forma de herança. _Structs_ e protocolos só podem adotar protocolos; eles não podem herdar de classes. No entanto, os tipos de hierarquias de herança que você pode construir com herança de classe também podem ser modelados usando herança de protocolo e _structs_.

Se você estiver construindo um relacionamento de herança do zero, prefira a herança de protocolo. Os protocolos permitem que classes, _structs_ e _enums_ participem da herança, enquanto a herança de classe é compatível apenas com outras classes. Ao escolher como modelar seus dados, tente construir a hierarquia de tipos de dados usando primeiro a herança de protocolo e, em seguida, adote esses protocolos em suas _structs_.
