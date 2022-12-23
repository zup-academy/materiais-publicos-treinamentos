# Generics

Código genérico permite que você escreva funções e tipos flexíveis e reutilizáveis que podem funcionar com qualquer tipo de dado, sujeitos aos requisitos que você definir. Você pode escrever um código que evite a duplicação e expresse sua intenção de maneira clara e abstrata.

_Generics_ são um dos recursos mais poderosos do Swift, e grande parte da biblioteca padrão do Swift é construída com código genérico. Na verdade, você tem usado _generics_ em todo o trabalho com a linguagem, mesmo que não tenha percebido. Por exemplo, os tipos _Array_ e _Dictionary_ do Swift são coleções genéricas. Você pode criar um array que contenha valores _Int_, ou um array que contenha valores _String_, ou mesmo um array para qualquer outro tipo que possa ser criado em Swift. Da mesma forma, você pode criar um dicionário para armazenar valores de qualquer tipo especificado e não há limitações sobre o que esse tipo pode ser.

## O problema que _generics_ resolve

Aqui está uma função não genérica padrão chamada `swap(_:_:)`, que troca dois valores `Int`:

``` swift
func swap(_ a: inout Int, _ b: inout Int) {
     let tempA = a
     a = b
     b = tempA
}
```

Esta função utiliza parâmetros de entrada-saída (`inout`) para trocar os valores de `a` e `b`.

A função `swap(_:_:)` troca o valor original de `b` por `a`, e o valor original de `a` por `b`. Você pode chamar esta função para trocar os valores em duas variáveis Int:

``` swift
var umInt = 3
var outroInt = 107

swap(&umInt, &outroInt)
print("umInt agora é \(umInt), e outroInt agora é \(outroInt)")
// Imprime "umInt agora é 107, e outroInt agora é 3"
```

A função `swap(_:_:)` é útil, mas só pode ser usada com valores `Int`. Se você quiser trocar dois valores `String`, ou dois valores `Double`, você deve escrever mais funções, como as funções `swapStrings(_:_:)` e `swapDoubles(_:_:)` abaixo:

``` swift
func swapStrings(_ a: inout String, _ b: inout String) {
     let tempA = a
     a = b
     b = tempA
}

func swapDoubles(_ a: inout Double, _ b: inout Double) {
     let tempA = a
     a = b
     b = tempA
}
```

Você deve ter notado que os corpos das funções `swap(_:_:)`, `swapStrings(_:_:)` e `swapDoubles(_:_:)` são idênticos. A única diferença é o tipo dos valores que aceitam (`Int`, `String` e `Double`).

É mais útil e consideravelmente mais flexível escrever uma única função que troque dois valores de qualquer tipo. Código genérico permite que você escreva tal função. (Uma versão genérica dessas funções é definida abaixo.)

> Nota: Em todas as três funções, os tipos de `a` e `b` devem ser os mesmos. Se `a` e `b` não forem do mesmo tipo, não é possível trocar seus valores. Swift é uma linguagem type-safe e não permite (por exemplo) que uma variável do tipo `String` e uma variável do tipo `Double` troquem valores entre si. Tentar fazer isso resulta em um erro de tempo de compilação.

## Generic Functions

As funções genéricas podem funcionar com qualquer tipo. Aqui está uma versão genérica da função `swap(_:_:)` acima, chamada `swapValues(_:_:)`:

``` swift
func swapValues<T>(_ a: inout T, _ b: inout T) {
     let tempA = a
     a = b
     b = tempA
}
```

O corpo da função `swapValues(_:_:)` é idêntico ao corpo da função `swap(_:_:)`. No entanto, a primeira linha de `swapValues(_:_:)` é ligeiramente diferente de `swap(_:_:)`. Veja como as primeiras linhas se comparam:

``` swift
func swap(_ a: inout Int, _ b: inout Int)
func swapValues<T>(_ a: inout T, _ b: inout T)
```

A versão genérica da função usa um _placeholder_ para nome de tipo (chamado `T`, neste caso) em vez de um nome de tipo real (como `Int`, `String` ou `Double`). O _placeholder_ do tipo não diz nada sobre o que `T` deve ser, mas diz que tanto `a` quanto `b` devem ser do mesmo tipo `T`, independentemente do que `T` represente. O tipo real a ser usado no lugar de `T` é determinado cada vez que a função `swapValues(_:_:)` é chamada.

A outra diferença entre uma função genérica e uma função não genérica é que o nome da função genérica (`swapValues(_:_:)`) é seguido pelo _placeholder_ para nome do tipo (`T`) dentro de colchetes angulares (`<T>`). Os colchetes informam ao Swift que `T` é um _placeholder_ para o tipo na definição da função `swapValues(_:_:)`. Como `T` é um _placeholder_ (um espaço reservado, ou uma espécie de lacuna a ser preenchida pela chamada da função), o Swift não procura por um tipo real chamado `T`.

A função `swapValues(_:_:)` agora pode ser chamada da mesma forma que `swap(_:_:)`, exceto que pode receber dois valores de qualquer tipo, desde que ambos sejam do mesmo tipo. Cada vez que `swapValues(_:_:)` é chamada, o tipo a ser usado para `T` é inferido dos tipos de valores passados para a função.

Nos dois exemplos abaixo, `T` é inferido como `Int` e `String`, respectivamente:

``` swift
var umInt = 3
var outroInt = 107

swapValues(&umInt, &outroInt)
// umInt agora é 107, e outroInt agora é 3

var umaString = "olá"
var outraString = "mundo"

swapValues(&umaString, &outraString)
// umaString agora é "world", e outraString agora é "hello"
```

> Nota: A função `swapValues(_:_:)` definida acima é inspirada em uma função genérica chamada `swap`, que faz parte da biblioteca padrão Swift, e é disponibilizada automaticamente para você usar em seus aplicativos. Se você precisar do comportamento da função `swapValues(_:_:)` em seu próprio código, poderá usar a função `swap(_:_:)` existente do Swift em vez de fornecer sua própria implementação.

## Parâmetros de tipo _(Type Parameters)_

No exemplo `swapValues(_:_:)` acima, o _placeholder_ T é um exemplo de parâmetro de tipo. Os parâmetros de tipo especificam e nomeiam um _placeholder_ para tipos e são escritos imediatamente após o nome da função, entre um par de colchetes angulares correspondentes (como `<T>`).

Depois de especificar um _Type Parameter_, você pode usá-lo para definir o tipo dos parâmetros de uma função (como os parâmetros `a` e `b` da função `swapValues(_:_:)`), ou como o tipo de retorno da função ou como um tipo anotação dentro do corpo da função. Em cada caso, o _Type Parameter_ é substituído por um tipo real sempre que a função é chamada. (No exemplo `swapValues(_:_:)` acima, `T` foi substituído por `Int` na primeira vez que a função foi chamada e foi substituído por `String` na segunda vez que foi chamada.)

Você pode fornecer mais de um _Type Parameter_ escrevendo vários nomes de _Type Parameter_ entre colchetes angulares, separados por vírgulas.

## Nomeando _Type Parameters_

Na maioria dos casos, os _Type Parameters_ têm nomes descritivos, como `Key` e `Value` em `Dictionary<Key, Value>` e `Element` em `Array<Element>`, que informa ao leitor sobre a relação entre o _Type Parameter_ e o tipo genérico ou função em que é usado. No entanto, quando não há um relacionamento significativo entre eles, é tradicional nomeá-los usando letras únicas, como `T`, `U` e `V`, como `T` na função `swapValues(_:_:)` acima.

> Nota: Sempre forneça nomes em maiúsculas aos _Type Parameters_ (como `T` e `TypeParameter`) para indicar que eles são um _placeholder_ para um tipo, não um valor.

## Tipos Genéricos

Além das funções genéricas, o Swift permite que você defina seus próprios tipos genéricos. São classes, _structs_ e _enums_ personalizadas que podem funcionar com qualquer tipo, de maneira semelhante a `Array` e `Dictionary`.

Esta seção mostra como escrever um tipo de coleção genérico chamado `Stack`. Uma pilha é um conjunto ordenado de valores, semelhante a um _array_, mas com um conjunto de operações mais restrito do que o tipo `Array` do Swift. Um _array_ permite que novos itens sejam inseridos e removidos em qualquer local do _array_. Uma pilha, no entanto, permite que novos itens sejam anexados apenas no topo da coleção (conhecido como empurrar um novo valor para a pilha, _push_). Da mesma forma, uma pilha permite que os itens sejam removidos apenas do topo da coleção (conhecido como remover um valor da pilha, _pop_).

> Nota: O conceito de pilha é usado pela classe `UINavigationController` para modelar os _view controllers_ em sua hierarquia de navegação. Você chama o método `pushViewController(_:animated:)` da classe `UINavigationController` para adicionar (_push_) um _view controller_ à pilha de navegação e seu método `popViewControllerAnimated(_:)` para remover (_pop_) um _view controller_ da pilha de navegação. Uma pilha é um modelo de coleção útil sempre que você precisa de uma abordagem rígida de “último a entrar, primeiro a sair” no gerenciamento de uma coleção.

A ilustração abaixo mostra o comportamento de _push_ e _pop_ de uma pilha:

<p align="center">
<img alt="Imagem com exemplo de modelo de push e pop de uma pilha, com elementos sendo adicionados e removidos apenas do topo da coleção" src="https://docs.swift.org/swift-book/_images/stackPushPop_2x.png" width="75%" />
</p>

1. No início, existem três valores na pilha.

1. Um quarto valor é colocado no topo da pilha.

1. A pilha agora contém quatro valores, com o mais recente no topo.

1. O item do topo da pilha é removido.

1. Depois de remover um valor, a pilha mais uma vez contém três valores.

Veja como escrever uma versão não genérica de uma pilha, neste caso para uma pilha de valores `Int`:

``` swift
struct IntStack {
    var itens: [Int] = []
    
    mutating func push(_ item: Int) {
        itens.append(item)
    }
    
    mutating func pop() -> Int {
        return itens.removeLast()
    }
}
```

Essa estrutura usa uma propriedade de tipo `Array` chamada `itens` para armazenar os valores na pilha. `IntStack` fornece dois métodos, `push` e `pop`, para inserir e remover valores da pilha. Esses métodos são marcados como mutáveis, porque precisam modificar o estado do _array_ de itens da _struct_. 

No entanto, o tipo `IntStack` mostrado acima só pode ser usado com valores `Int`. Seria muito mais útil definir uma estrutura `Stack` genérica, que pudesse gerenciar uma pilha de qualquer tipo de valor. Aqui está uma versão genérica do mesmo código:

``` swift
struct Stack<Element> {
    var itens: [Element] = []
    
    mutating func push(_ item: Element) {
        itens.append(item)
    }
    
    mutating func pop() -> Element {
        return itens.removeLast()
    }
}
```

Observe como a versão genérica de `Stack` é essencialmente igual à versão não genérica, mas com um _Type Parameter_ chamado `Element` em vez de um tipo real de `Int`. Este _Type Parameter_ é escrito dentro de um par de colchetes angulares (`<Element>`) imediatamente após o nome da _struct_. 

O elemento define um nome de _placeholder_ para um tipo a ser fornecido posteriormente. Esse tipo futuro pode ser referido como `Element` em qualquer lugar dentro da definição da _struct_. Nesse caso, `Element` é usado como espaço reservado em três locais:

* Para criar uma propriedade chamada `itens`, que é inicializada com um _array_ vazio de valores do tipo `Element`;

* Para especificar que o método `push(_:)` possui um único parâmetro chamado `item`, que deve ser do tipo `Element`;

* Para especificar que o valor retornado pelo método `pop()` será um valor do tipo `Element`.

Por ser um tipo genérico, `Stack` pode ser usado para criar uma pilha de qualquer tipo válido no Swift, de maneira semelhante a `Array` e `Dictionary`.

Você cria uma nova instância `Stack` escrevendo o tipo a ser armazenado na pilha entre colchetes angulares. Por exemplo, para criar uma nova pilha de _strings_, você escreve `Stack<String>()`:

``` swift
var stack = Stack<String>()

stackOfStrings.push("uno")
stackOfStrings.push("dos")
stackOfStrings.push("tres")
stackOfStrings.push("cuatro")

// a stack agora contém 4 strings
```

Veja como `stack` fica depois de colocar esses quatro valores na pilha:

<p align="center">
<img alt="Imagem com o modelo de execução do código da stack de strings" src="https://docs.swift.org/swift-book/_images/stackPushedFourStrings_2x.png" width="75%" />
</p>

Desempilhar um valor da pilha remove e retorna o valor superior, `"cuatro"`:

``` swift
let string = stack.pop()
// string é igual a "cuatro", e a stack agora contém 3 strings
```

Veja como a pilha fica depois de desempilhar seu valor superior:

<p align="center">
<img alt="Imagem com o modelo de execução do código da stack de strings depois de remover o valor do topo" src="https://docs.swift.org/swift-book/_images/stackPoppedOneString_2x.png" width="50%" />
</p>

## Estendendo um tipo genérico

Ao estender um tipo genérico, você não fornece uma lista de _Type Parameters_ como parte da definição da extensão. Em vez disso, a lista de _Type Parameters_ da definição de tipo original está disponível no corpo da extensão e os nomes dos _Type Parameters_ originais são usados ​​para fazer referência aos _Type Parameters_ da definição original. 

O exemplo a seguir estende o tipo `Stack` genérico para adicionar uma propriedade computada somente leitura chamada `topItem`, que retorna o item superior da pilha sem removê-lo da pilha:

``` swift
extension Stack {
    var topItem: Element? {
        return itens.isEmpty ? nil : itens[itens.count - 1]
    }
}
```

A propriedade `topItem` retorna um valor opcional do tipo `Element`. Se a pilha estiver vazia, `topItem` retornará `nil`; se a pilha não estiver vazia, `topItem` retorna o item ao final do _array_ de itens. 

Observe que esta extensão não define uma lista de _Type Parameters_. Em vez disso, o nome do _Type Parameter_ existente do tipo `Stack`, `Element`, é usado na extensão para indicar o tipo opcional da propriedade computada `topItem`. 

A propriedade computada `topItem` agora pode ser usada com qualquer instância `Stack` para acessar e consultar seu item do topo sem removê-lo.

``` swift
if let topItem = stack.topItem {
    print("O topItem na stack é \(topItem).")
}
// Imprime "O topItem na stack é tres."
```

As extensões de um tipo genérico também podem incluir requisitos que as instâncias do tipo estendido devem satisfazer para obter a nova funcionalidade, conforme discutido na seção abaixo.

## Restrições para um tipo _(Type Constraints)_

A função `swapValues(_:_:)` e o tipo `Stack` podem funcionar com qualquer tipo. No entanto, às vezes é útil impor certas restrições de tipo nos tipos que podem ser usados com funções genéricas e tipos genéricos. As restrições de tipo especificam que um parâmetro de tipo deve herdar de uma classe específica ou estar de acordo com um determinado protocolo ou composição de protocolos.

Por exemplo, o tipo `Dictionary` do Swift coloca uma limitação nos tipos que podem ser usados como chaves para um dicionário. O tipo de chave de um dicionário deve ser passível de `hash`. Ou seja, deve fornecer uma maneira de se tornar exclusivamente representável. O dicionário precisa que suas chaves sejam _hasheáveis_ para que possa verificar se já contém um valor para uma chave específica. Sem esse requisito, o `Dictionary` não poderia dizer se deveria inserir ou substituir um valor para uma determinada chave, nem seria capaz de encontrar um valor para uma determinada chave que já está no dicionário.

Este requisito é reforçado por uma restrição de tipo no tipo de chave para `Dictionary`, que especifica que o tipo de chave deve estar em conformidade com o protocolo `Hashable`, um protocolo especial definido na biblioteca padrão Swift. Todos os tipos básicos do Swift (como `String`, `Int`, `Double` e `Bool`) são passíveis de hash por padrão. Para obter informações sobre como tornar seus próprios tipos personalizados em conformidade com o protocolo `Hashable`, consulte a [documentação](https://developer.apple.com/documentation/swift/hashable#2849490).

Você pode definir suas próprias restrições de tipo ao criar tipos genéricos personalizados, e essas restrições fornecem grande parte do poder da programação genérica. Conceitos abstratos como `Hashable` caracterizam tipos em termos de suas características conceituais, em vez de seu tipo concreto.

### Sintaxe de Restrições de Tipo

Você escreve restrições de tipo colocando uma única restrição de classe ou protocolo após o nome de um _type parameter_, separado por dois-pontos, como parte da lista de parâmetros de tipo. A sintaxe básica para restrições de tipo em uma função genérica é mostrada abaixo (embora a sintaxe seja a mesma para tipos genéricos):

``` swift
func umaFuncao<T: UmaClasse, U: UmProtocolo>(umT: T, umU: U) {
     // o corpo da função vai aqui
}
```

A função hipotética acima tem dois _type parameters_. O primeiro _type parameter_, `T`, tem uma restrição de tipo que exige que `T` seja uma subclasse de `UmaClasse`. O segundo _type parameter_, `U`, tem uma restrição de tipo que exige que `U` esteja em conformidade com o protocolo hipotético `UmProtocolo`.

### Restrições de tipo na prática

Aqui está uma função não genérica chamada `findIndex(ofString:in:)`, que recebe um valor `String` para localizar e um _array_ de valores `String` dentro dos quais quer encontrá-lo. A função `findIndex(ofString:in:)` retorna um valor `Int` opcional, que será o índice da primeira _string_ correspondente no _array_ se for encontrada, ou `nil` se a string não puder ser encontrada:

``` swift
func findIndex(ofString valueToFind: String, in array: [String]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

A função `findIndex(ofString:in:)` pode ser usada para encontrar um valor de string em um _array_ de strings:

``` swift
let strings = ["gato", "cachorro", "lhama", "periquito", "papagaio"]

if let indice = findIndex(ofString: "lhama", in: strings) {
     print("O índice da lhama é \(index)")
}
// Imprime "O índice da lhama é 2"
```

No entanto, o princípio de encontrar o índice de um valor em um _array_ não é útil apenas para _strings_. Você pode escrever a mesma funcionalidade de uma função genérica substituindo qualquer menção de strings por valores de algum tipo `T`.

Veja como você pode esperar que uma versão genérica de `findIndex(ofString:in:)`, chamada `findIndex(of:in:)`, seja escrita. Observe que o tipo de retorno dessa função ainda é `Int?`, porque a função retorna um número de índice opcional, não um valor opcional do _array_. Porém, note, esta função não compila, por razões explicadas após o exemplo:

``` swift
func findIndex<T>(of valueToFind: T, in array: [T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

Esta função não compila como descrito acima. O problema está na verificação de igualdade, `if value == valueToFind`. Nem todo tipo em Swift pode ser comparado com o operador igual a (`==`). Se você criar sua própria classe ou _struct_ para representar um modelo de dados complexo, por exemplo, o significado de `“igual a”` para essa classe ou _struct_ não é algo que o Swift possa adivinhar para você. Por causa disso, não é possível garantir que esse código funcione para todos os tipos `T` possíveis, e um erro apropriado é relatado quando você tenta compilar o código.

Nem tudo está perdido, no entanto. A biblioteca padrão Swift define um protocolo chamado `Equatable`, que requer qualquer tipo compatível para implementar o operador igual a (`==`) e o operador diferente de (`!=`) para comparar quaisquer dois valores desse tipo. Todos os tipos padrão do Swift suportam automaticamente o protocolo `Equatable`.

Qualquer tipo que seja `Equatable` pode ser usado com segurança com a função `findIndex(of:in:)`, porque é garantido o suporte ao operador `"igual a"`. Para expressar esse fato, você escreve uma restrição de tipo de `Equatable` como parte da definição do _type parameter_ ao definir a função:

``` swift
func findIndex<T: Equatable>(of valueToFind: T, in array: [T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

O parâmetro de tipo único para `findIndex(of:in:)` é escrito como `T: Equatable`, que significa “qualquer tipo `T` que esteja em conformidade com o protocolo `Equatable`”.

A função `findIndex(of:in:)` agora compila com sucesso e pode ser usada com qualquer tipo `Equatable`, como `Double`, `String` ou qualquer outro tipo customizado que você criar que implemente o protocolo:

``` swift
let indiceDoDouble = findIndex(of: 9.3, in: [3.14159, 0.1, 0.25])
// indiceDoDouble é um Int opcional sem valor, porque 9.3 não está no array

let indiceDaString = findIndex(of: ""Iasmyn, in: ["Matheus", "Yuri", "Iasmyn"])
// indiceDaString é um Int opcional contendo um valor de 2
```

## Cláusulas _Where_ em _Generics_

As restrições de tipo, conforme descritas acima, permitem definir requisitos nos _type parameters_ associados a uma função ou tipo genérico.

Também pode ser útil definir requisitos adicionais para os tipos que se associam às implementações genéricas. Você faz isso definindo uma cláusula `where` genérica. Uma cláusula `where` genérica permite exigir que um tipo associado esteja em conformidade com um determinado protocolo ou que determinados _type parameters_ sejam os mesmos. Uma cláusula `where` genérica começa com a palavra-chave `where`, seguida por restrições para tipos associados ou relações de igualdade entre tipos. Você escreve uma cláusula `where` genérica logo antes da chave de abertura de um tipo ou corpo de função.

O exemplo abaixo define uma função genérica chamada `allItemsMatch`, que verifica se duas instâncias de `Container` contêm os mesmos itens no mesmo pedido. A função retorna um valor booleano `true` se todos os itens corresponderem e um valor `false` se não corresponderem.

Os dois contêineres a serem verificados não precisam ser do mesmo tipo (embora possam ser), mas devem conter o mesmo tipo de itens. Esse requisito é expresso por meio de uma combinação de restrições de tipo e uma cláusula `where` genérica:

``` swift
func allItemsMatch<C1: Container, C2: Container>(_ umContainer: C1, _ outroContainer: C2) -> Bool
    where C1.Item == C2.Item, C1.Item: Equatable {

    // Verifique se ambos os contêineres contêm o mesmo número de itens.
    if umContainer.count != outroContainer.count {
        retorna false
    }

    // Verifica cada par de itens para ver se são equivalentes.
    for i in 0..<umContainer.count {
        if umContainer[i] != outroContainer[i] {
            return false
        }
    }

    // Todos os itens combinam, então retorne true.
    return true
}
```

Essa função recebe dois argumentos chamados `umContainer` e `outroContainer`. O argumento `umContainer` é do tipo `C1` e o argumento `outroContainer` é do tipo `C2`. Ambos `C1` e `C2` são _type parameters_ para dois tipos de contêiner a serem determinados quando a função é chamada.

Os seguintes requisitos são colocados nos dois _type parameters_ da função:

* `C1` deve estar em conformidade com o protocolo `Container` (escrito como `C1: Container`);

* `C2` também deve estar em conformidade com o protocolo `Container` (escrito como `C2: Container`);

* O `Item` para `C1` deve ser o mesmo que o `Item` para `C2` (escrito como `C1.Item == C2.Item`);

* O item para `C1` deve estar em conformidade com o protocolo `Equatable` (escrito como `C1.Item: Equatable`).

O primeiro e o segundo requisitos são definidos na lista de _type parameters_ da função, e o terceiro e o quarto requisitos são definidos na cláusula `where` genérica da função.

Esses requisitos significam:

* `umContainer` é um contêiner do tipo `C1`;

* `outroContainer` é um contêiner do tipo `C2`;

* `umContainer` e `outroContainer` contêm o mesmo tipo de itens;

* Os itens em `umContainer` podem ser verificados com o operador diferente (`!=`) para ver se são diferentes entre si.

O terceiro e o quarto requisitos se combinam para significar que os itens em outro `Container` também podem ser verificados com o operador `!=`, porque são exatamente do mesmo tipo que os itens em `umContainer`.

Esses requisitos permitem que a função `allItemsMatch(_:_:)` compare os dois contêineres, mesmo que sejam de um tipo de contêiner diferente.

A função `allItemsMatch(_:_:)` começa verificando se ambos os contêineres contêm o mesmo número de itens. Se eles contiverem um número diferente de itens, não há como eles corresponderem e a função retornará falso.

Depois de fazer essa verificação, a função itera sobre todos os itens em `umContainer` com um loop `for-in` e o operador de intervalo semi-aberto (`..<`). Para cada item, a função verifica se o item de `umContainer` não é igual ao item correspondente em `outroContainer`. Se os dois itens não forem iguais, os dois contêineres não correspondem e a função retornará falso.

Se o loop terminar sem encontrar uma incompatibilidade, os dois contêineres serão correspondentes e a função retornará verdadeiro.

Veja como a função `allItemsMatch(_:_:)` se parece em ação:

``` swift
var stack = Stack<String>()
stack.push("uno")
stack.push("dos")
stack.push("tres")

var array = ["uno", "dos", "três"]

if allItemsMatch(stack, array) {
    print("Todos os itens coincidem.")
} else {
    print("Nem todos os itens coincidem.")
}

// Imprime "Todos os itens correspondem."
```

> Nota: O exemplo acima leva em conta que tanto o tipo `Stack`, quanto o tipo `Array` da Swift estão em conformidade com um protocolo Container.

O exemplo acima cria uma instância Stack para armazenar valores String e coloca três strings na pilha. O exemplo também cria uma instância de Array inicializada com uma matriz literal contendo as mesmas três strings da pilha. Mesmo que a pilha e o array sejam de um tipo diferente, ambos estão em conformidade com o protocolo Container e ambos contêm o mesmo tipo de valores. Portanto, você pode chamar a função allItemsMatch(_:_:) com esses dois contêineres como seus argumentos. No exemplo acima, a função allItemsMatch(_:_:) informa corretamente que todos os itens nos dois contêineres correspondem.

## Extensões com uma cláusula `Where` genérica

Você também pode usar uma cláusula `where` genérica como parte de uma extensão. O exemplo abaixo estende a _struct_ `Stack` genérica dos exemplos anteriores para adicionar um método `isTop(_:)`.

``` swift
extension Stack where Element: Equatable {
    func isTop(_ item: Element) -> Bool {
        guard let topItem = itens.last else {
            return false
        }
        return topItem == item
    }
}
```

Esse novo método `isTop(_:)` primeiro verifica se a pilha não está vazia e, em seguida, compara o item fornecido com o item no topo da pilha. Se você tentasse fazer isso sem uma cláusula `where` genérica, teria um problema: a implementação de `isTop(_:)` usa o operador `==`, mas a definição de `Stack` não exige que seus itens sejam _equatable_, portanto, usar o `==` resulta em um erro em tempo de compilação. O uso de uma cláusula `where` genérica permite adicionar um novo requisito à extensão, para que a extensão adicione o método `isTop(_:)` somente quando os itens na pilha forem iguais.

Veja como o método `isTop(_:)` é utilizado na prática:

``` swift
if stack.isTop("tres") {
    print("Elemento superior é tres.")
} else {
    print("O elemento superior é outro.")
}

// Imprime "O elemento superior é tres."
```

Se você tentar chamar o método `isTop(_:)` em uma pilha cujos elementos não são _equatable_, receberá um erro de tempo de compilação.

``` swift
struct NaoEquatable {

}

var stack = Stack<NaoEquatable>()
let naoEquatable = NaoEquatable()

stack.push(naoEquatable)
stack.isTop(naoEquatable) // Erro
```

Você pode incluir vários requisitos em uma cláusula `where` genérica que faz parte de uma extensão, assim como você pode para uma cláusula `where` genérica que você escreve em outro lugar. Separe cada requisito na lista com uma vírgula.
