# Mais sobre Swift: Arrays

> Esse material teórico foi traduzido e adaptado da fonte original disponível em https://docs.swift.org/swift-book/LanguageGuide/CollectionTypes.html

Swift oferece três tipos principais para armazenar coleções de valores, são eles: _arrays_, conjuntos (_sets_) e dicionários. Neste documento apresentamos os Arrays. Enquanto as outras opções oferecem coleções não ordenadas, e com a possibilidade de organizar valores únicos, Arrays são coleções valores ordenadas através de índices e que, por padrão, permitem repetição de valores.

<p align="center">
<img alt="Ilustração de exemplo de uma estrutura de _array_, contendo cinco itens que formam ingredientes de uma receita de panqueca organizados ou indexados de zero até quatro" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/table-views-teoria-arrays-imagem-conceito-array.jpg?raw=true" width="70%" />
</p>

Arrays são sempre claros sobre os tipos de valores que podem armazenar. Isso significa que você não pode inserir um valor do tipo errado em um Array por engano. Isso também significa que você pode ter certeza sobre o tipo de valores que recuperará de um Array.

## Mutabilidade das coleções

Se você criar um _array_ - ou mesmo conjuntos ou dicionários - e atribuí-lo a uma **variável** (`var`), a coleção criada será **mutável**. Isso significa que você pode alterar (ou mutar) a coleção depois de criá-la adicionando, removendo ou alterando itens na coleção. Se você atribuir um _array_ a uma **constante** (`let`), essa coleção será imutável e seu tamanho e conteúdo não poderão ser alterados.

> Nota: É uma boa prática criar coleções imutáveis em todos os casos em que a coleção não precisa ser alterada. Isso facilita o raciocínio sobre seu código e permite que o compilador Swift otimize o desempenho das coleções que você cria.

## Arrays

Um _array_ armazena valores do mesmo tipo em uma lista ordenada. O mesmo valor pode aparecer em um _array_ várias vezes em posições diferentes.

> Nota: O tipo Array do Swift é conectado à classe `NSArray` da Foundation.

### Sintaxe abreviada do tipo de Array

O tipo de um _array_ Swift é escrito na íntegra como `Array<Element>`, onde `Element` é o tipo de valores que o _array_ pode armazenar. Você também pode escrever o tipo de um _array_ na forma abreviada como `[Element]`. Embora as duas formas sejam funcionalmente idênticas, a forma abreviada é a preferida e é usada em todo este guia ao se referir ao tipo de um _array_.

### Criando um _array_ vazio

Você pode criar um _array_ vazio de um determinado tipo usando a sintaxe do inicializador:

``` swift
var algunsInts: [Int] = []

print("algunsInts é do tipo [Int] com \(algunsInts.count) itens.");
// Imprime "algunsInts é do tipo [Int] com 0 itens."
```

Observe que o tipo da variável `algunsInts` é inferido como `[Int]` do tipo do inicializador.

Alternativamente, se o contexto já fornece informações de tipo, como um argumento de função ou uma variável ou constante já escrita, você pode criar um _array_ vazio com um literal de _array_ vazio, que é escrito como [] (um par vazio de colchetes):

``` swift
algunsInts.append(3)
// algunsInts agora contém um valor do tipo Int

algunsInts = []
// algunsInts agora é um _array_ vazio, mas ainda é do tipo [Int]
```

### Criando um _array_ com um valor padrão

O tipo Array do Swift também fornece um inicializador para criar um _array_ de um determinado tamanho com todos os seus valores definidos para o mesmo valor padrão. Você passa a esse inicializador um valor padrão do tipo apropriado (chamado `repeating`): e o número de vezes que esse valor é repetido no novo _array_ (chamado `count`):

``` swift
var tresDoubles = Array(repeating: 0.0, count: 3)
// tresDoubles é do tipo [Double] e é igual a [0.0, 0.0, 0.0]
```

### Criando um _array_ através da adição de dois outros

Você pode criar um novo _array_ adicionando dois _arrays_ existentes com tipos compatíveis com o operador de adição (`+`). O tipo do novo _array_ é inferido a partir do tipo dos dois _arrays_ que você adiciona:

``` swift
var outrosTresDoubles = Array(repeating: 2.5, count: 3)
// outrosTresDoubles é do tipo [Double] e é igual a [2.5, 2.5, 2.5]

var seisDoubles = tresDoubles + outrosTresDoubles
// seisDoubles é inferido como [Double], e é igual a [0.0, 0.0, 0.0, 2.5, 2.5, 2.5]
```

### Criando um Array através de Array Literal

Você também pode inicializar um _array_ com um literal de _array_, que é uma forma abreviada de escrever um ou mais valores como um _array_. Um _array literal_ é escrito como uma lista de valores, separados por vírgulas, cercados por um par de colchetes:

``` swift
[valor1, valor2, valor3]
```

O exemplo abaixo cria um _array_ chamado `listaDeCompras` para armazenar valores de `String`:

``` swift
var listaDeCompras: [String] = ["Ovos", "Leite"]
// listaDeCompras foi inicializado com dois itens iniciais
```

A variável `listaDeCompras` é dita como “um _array_ de valores string”, escrito como `[String]`. Como esse _array_ especificou um tipo de valor `String`, é permitido armazenar apenas valores `String`. Aqui, o _array_ `listaDeCompras` é inicializado com dois valores `String` (`"Ovos"` e `"Leite"`), escritos dentro de um _array literal_.

> Nota: O _array_ `listaDeCompras` é declarado como uma variável (`var`) e não uma constante (`let`) porque mais itens são adicionados à lista de compras nos exemplos abaixo.

Nesse caso, o _array literal_ contém dois valores `String` e nada mais. Isso corresponde ao tipo de declaração da variável `listaDeCompras` (um _array_ que pode conter apenas valores `String`) e, portanto, a atribuição do literal do _array_ é permitida como forma de inicializar `listaDeCompras` com dois itens iniciais.

Graças à inferência de tipo do Swift, você não precisa escrever o tipo do _array_ se estiver inicializando-o com um literal de _array_ contendo valores do mesmo tipo. A inicialização de `listaDeCompras` poderia ter sido escrita em uma forma mais curta:

``` swift
var listaDeCompras = ["Ovos", "Leite"]
```

Como todos os valores no _array_ literal são do mesmo tipo, Swift pode inferir que `[String]` é o tipo correto a ser usado para a variável `listaDeCompras`.

### Acessando e modificando um _array_

Você acessa e modifica um _array_ por meio de seus métodos e propriedades ou usando a sintaxe de subscrito. Para descobrir o número de itens em um _array_, verifique sua propriedade de contagem somente leitura:

``` swift
print("A lista de compras contém \(listaDeCompras.count) itens.");
// Imprime "A lista de compras contém 2 itens."
```

Use a propriedade booleana `isEmpty` como atalho para verificar se a propriedade `count` é igual a 0:

``` swift
if listaDeCompras.isEmpty {
    print("A lista de compras está vazia.")
} else {
    print("A lista de compras não está vazia.");
}
// Imprime "A lista de compras não está vazia."
```

Você pode adicionar um novo item ao final de um _array_ chamando o método `append(_:)` do _array_:

``` swift
listaDeCompras.append("Farinha")
// listaDeCompras agora contém 3 itens e alguém está fazendo panquecas
```

Alternativamente, anexe um _array_ de um ou mais itens compatíveis com o operador de atribuição de adição (`+=`):

``` swift
listaDeCompras += ["Fermento"]
// listaDeCompras agora contém 4 itens

listaDeCompras += ["Chocolate", "Queijo", "Manteiga"]
// listaDeCompras agora contém 7 itens
```

Recupere um valor do _array_ usando a sintaxe do subscrito, passando o índice do valor que deseja recuperar entre colchetes imediatamente após o nome do _array_:

``` swift
var primeiroItem = listaDeCompras[0]
// primeiroItem é igual a "Ovos"
```

> Nota: O primeiro item no _array_ tem um índice de 0, não 1. Arrays em Swift são sempre indexados a zero.

Você pode usar a sintaxe de subscrito para alterar um valor existente em um determinado índice:

``` swift
listaDeCompras[0] = "Seis ovos"
// o primeiro item da lista agora é igual a "Seis ovos" em vez de "Ovos"
```

Ao usar a sintaxe de subscrito, o índice especificado precisa ser válido. Por exemplo, escrever `listaDeCompras[listaDeCompras.count] = "Sal"` para tentar anexar um item ao final do _array_ resulta em um erro em tempo de execução.

Você também pode usar a sintaxe de subscrito para alterar um intervalo de valores de uma só vez, mesmo que o conjunto de valores de substituição tenha um comprimento diferente do intervalo que você está substituindo. O exemplo a seguir substitui `"Chocolate"`, `"Queijo"` e `"Manteiga"` por `"Bananas"` e `"Maças"`:

``` swift
listaDeCompras[4...6] = ["Bananas", "Maçãs"]
// listaDeCompras agora contém 6 itens
```

Para inserir um item no _array_ em um índice especificado, chame o método `insert(_:at:)` do _array_:

``` swift
listaDeCompras.insert("Xarope", at: 0)
// listaDeCompras agora contém 7 itens
// "Xarope" agora é o primeiro item da lista
```

Essa chamada ao método `insert(_:at:)` insere um novo item com um valor de `"Xarope"` no início da lista de compras, indicado por um índice de 0.

Da mesma forma, você remove um item do _array_ com o método `remove(at:)`. Este método remove o item no índice especificado e retorna o item removido (embora você possa ignorar o valor retornado se não precisar dele):

``` swift
let xarope = listaDeCompras.remove(at: 0)
// o item que estava no índice 0 acaba de ser removido
// listaDeCompras agora contém 6 itens e nenhum Xarope
// a constante xarope agora é igual à string "Xarope" removida
```

> Nota: Se você tentar acessar ou modificar um valor para um índice que está fora dos limites existentes de um _array_, você acionará um erro em tempo de execução. Você pode verificar se um índice é válido antes de usá-lo comparando-o com a propriedade count do _array_. O maior índice válido em um _array_ é `count - 1` porque os _arrays_ são indexadoss a partir de zero — no entanto, quando `count` é `0` (significando que o _array_ está vazio), não há índices válidos.

Quaisquer lacunas em um _array_ são fechadas quando um item é removido e, portanto, o valor no índice 0 é novamente igual a `"Seis ovos"`:

``` swift
primeiroItem = listaDeCompras[0]
// primeiroItem agora é igual a "Seis ovos"
```

Se você deseja remover o item final de um _array_, use o método `removeLast()` em vez do método `remove(at:)` para evitar a necessidade de consultar a propriedade `count` do _array_. Assim como o método `remove(at:)`, `removeLast()` retorna o item removido:

``` swift
let macas = listaDeCompras.removeLast()
// o último item do array acabou de ser removido
// listaDeCompras agora contém 5 itens e nenhuma maçã
// a constante maçãs agora é igual à string "Maçãs" removida
```

### Iterando sobre um _array_

Você pode iterar sobre todo o conjunto de valores em um _array_ com um _loop_ `for-in`:

``` swift
for item in listaDeCompras {
    print(item)
}

// Seis ovos
// Leite
// Farinha
// Fermento
// Bananas
```

Se você precisar do inteiro representando o índice de cada item, bem como seu valor, use o método `enumerated()` para iterar sobre o _array_. Para cada item no _array_, o método `enumerated()` retorna uma tupla composta por um inteiro e o item. Os números inteiros começam em zero e contam de um em um para cada item; se você enumerar um _array_ inteira, esses números inteiros correspondem aos índices dos itens. Você pode decompor a tupla em constantes ou variáveis ​​temporárias como parte da iteração:

``` swift
for (indice, valor) in listaDeCompras.enumerated() {
    print("Item \(indice + 1): \(valor)")
}

// Item 1: Seis ovos
// Item 2: Leite
// Item 3: Farinha
// Item 4: Fermento
// Item 5: Bananas
```

## Manipulando coleções de maneira fluida

Assim como outras coleções os Arrays contam com uma API bastante interessante para auxiliar o trabalho manipulando e transformando dados. Nos tópicos adiante veremos as principais operações que podem ser executadas através de métodos, permitindo inclusive sua composição para trabalhar com os valores em uma espécie de fluxo de execução dos dados.

### Uma outra forma de iteração: `forEach(_:)`

Nos exemplos anteriores notamos como era possível iterar sobre um _array_ utilizando um _loop_ `for-in`, como o código a seguir ilustra.

``` swift
var listaDeCompras = ["Ovos", "Leite", "Farinha"]

for item in listaDeCompras {
    print(item)
}

// Ovos
// Leite
// Farinha
```

A API de `Array` fornece uma maneira análoga de iteração por todo seu conjunto de dados através do método `forEach(_:)`:

``` swift
listaDeCompras.forEach { item in
    print(item)
}

// Ovos
// Leite
// Farinha
```

A diferença fica, principalmente, por conta do estilo de programação. Através do _loop_ `for-in` definimos nosso bloco de código a executar num contexto externo ao Array. Já através de `forEach(_:)` passamos adiante para a função qual a implementação deve ser executada (contexto interno do Array) em repetição. Os detalhes de implementação acerca da iteração estão encapsulados no tipo Array.

> Nota: A declaração de `forEach(_:)` é definida por `func forEach(_ body: (Element) throws -> Void) rethrows`. O parâmetro body espera receber um bloco de código executável definido por uma função ou expressão closure que será aplicada para cada elemento. Mais a frente neste treinamento entraremos em mais detalhes sobre essas expressões em particular.
>
> Talvez você esteja se perguntando como `forEach(_:)`, sendo um função, pode ser invocada como um bloco de código, como no código acima. A explicação para isso se dá por conta de uma facilitação de sintaxe da linguagem. Se uma função for passada como único argumento na invocação, e você inferir sua implementação através de uma expressão closure, é possível omitir os parenteses na instrução. A forma completa de invocação poderia ser atingida pelo código abaixo.
>
>``` swift
>listaDeCompras.forEach({ item in
>    print(item)
>})
>```
>
>Caso a definição de `forEach` prevesse a utilização de algum label de argumento, como por exemplo, `func forEach(apply body: (Element) throws -> Void) rethrows`, a invocação se daria ainda de forma mais conhecida.
>
>``` swift
>listaDeCompras.forEach(apply: { item in
>    print(item)
>})
>```
>
>Existem formas ainda mais simples de construção tirando proveito das inferências que permitem uma sintaxe ainda mais sucinta.
>
>``` swift
>listaDeCompras.forEach { print($0) }
>```
>
>O código acima omite a declaração do nome para o parâmetro utilizado no corpo da expressão, e tira proveito da forma abreviada >para nomes de parâmento utilizando `$0` em seu lugar. Isso permite que passemos apenas o corpo da expressão, com as instruções que serão executas. 
>
>Caso função esperada como parâmetro por `forEach(_:)`, contassem com mais >argumentos, poderíamos utilizar `$0`, `$1`, `$n`, de acordo com a ordem na lista de parâmetros.

### Aplicando transformações com `map(_:)`

Para algumas ocasiões poderia ser útil ter meios práticos de prover implementação que oriente transformações sobre um conjunto de valores. A função `map(_:)` da Array é uma das que seguem exatamente a esse propósito.

Para que seja possível entender sua utilização, considere o código abaixo que visa obter uma coleção de valores contendo apenas os primeiros nomes dos instrutores da Zup Edu.

``` swift
let instrutores = ["Rafael Rollo", "Paula Santana", "Yuri Matheus"]

var primeirosNomes: [String] = []

for item in instrutores {
    primeirosNomes.append(item.components(separatedBy: " ").first!)
}

print(primeirosNomes)
// ["Rafael", "Paula", "Yuri"]
```

A solução acima atende ao objetivo mas parece um pouco verbosa. Pelo uso da função `map(_:)` conseguimos o mesmo resultado com um código muito mais sucinto.

``` swift
let primeirosNomes = instrutores.map { item in
    return item.components(separatedBy: " ").first!
}
```

Na verdade, essa solução pode se tornar ainda mais fluida.

``` swift
let primeirosNomes = instrutores.map { $0.components(separatedBy: " ").first! }
```

A função `map(_:)` tem sua declaração por `func map<T>(_ transform: (Element) throws -> T) rethrows -> [T]`. Excluindo alguns detalhes menos importantes para o escopo deste documento, é possível perceber que é esperado como parâmetro (`transform`) uma função que recebe um elemento e visa devolver outro elemento (expresso genericamente através de `T`) que pode ser informado no cabeçalho da expressão, ou inferido pelo tipo de retorno da implementação. Como retorno, a função `map(_:)` tem outro Array do mesmo tipo inferido a partir da expressão. A documentação da função garante a execução da implementação passada como parâmetro para cada elemento da lista e a devolução do Array com os valores transformados. 

### Selecionando elementos com `filter(_:)`

Em situações onde se deseja reduzir o universo de valores a partir da identificação de algum predicado, a função `filter(_:)` pode ajudar. Para entender sua utilização considere o código abaixo que se propõe a selecionar elementos de uma coleção de livros, tendo como corte o preço dos mesmos.

``` swift
let livros = [
    Livro("Uma breve história do tempo", por: 54.97),
    Livro("Respostas de um astrofísico", por: 42.90),
    Livro("Breves respostas para grandes questões", por: 47.90),
    Livro("Origens", por: 51.90),
    Livro("Buracos negros", por: 21.89),
]

var menoresPrecos: [Livro] = []

for livro in livros {
    if livro.preco < 50 {
        menoresPrecos.append(livro)
    }
}

// títulos selecionados são
// Respostas de um astrofísico
// Breves respostas para grandes questões
// Buracos negros
``` 

Com a utilização da função podemos ter o código a seguir.

``` swift
let livros = [
    Livro("Uma breve história do tempo", por: 54.97),
    Livro("Respostas de um astrofísico", por: 42.90),
    Livro("Breves respostas para grandes questões", por: 47.90),
    Livro("Origens", por: 51.90),
    Livro("Buracos negros", por: 21.89),
]

let menoresPrecos = livros.filter { $0.preco < 50 }
```

A função `filter(_:)` tem sua definição por `func filter(_ isIncluded: (Element) throws -> Bool) rethrows -> [Element]`. Embora a documentação não nos forneça muita informação de valor, pela assinatura da função podemos ter alguma ideia de seu funcionamento. Ela espera receber um único parâmetro que é uma função que será aplicada para cada elemento do conjunto de valores do _array_. Esta função deve receber um elemento como parâmetro e devolver um booleano, indicando se o elemento contém ou não o predicado desejado para que seja incluído entre os resultados. Ao fim da execução a função `filter(_:)` retorna outro _array_ de mesmo tipo contendo todos os elementos filtrados. 

### Gerando produtos através de transformações com `reduce(_:_:)` e `joined()`

Ao lidar com coleções de valores, em certas situações precisamos executar operação para totalizar valores. Transformações aplicadas para tal podem ser facilitadas pelo uso da função `reduce(_:_:)`.

Para entender sua utilização, antes, vejamos um trecho de código que visa calcular o valor total que um cliente investiria se comprasse todos os títulos de menores preços que foram selecionados.

``` swift
let titulosComMenoresPrecos = [
    Livro("Respostas de um astrofísico", por: 42.90),
    Livro("Breves respostas para grandes questões", por: 47.90),
    Livro("Buracos negros", por: 21.89),
]

var valorTotal: Decimal = 0.0

for livro in titulosComMenoresPrecos {
    valorTotal += livro.preco
}

print(valorTotal)
// 112.69
```

A solução novamente atende o objetivo mas se torna verbosa. Agora vejamos com a aplicação de `reduce(_:_:)`.

``` swift
let valorTotal = titulosComMenoresPrecos.reduce(0.0, { resultado, livro in
    return resultado + livro.preco
})

print(valorTotal)
// 112.69
```

A função `reduce(_:_:)` é definida por `func reduce<Result>(_ initialResult: Result, _ nextPartialResult: (Result, Element) throws -> Result) rethrows -> Result` e recebe dois parâmetros: o primeiro é um resultado (ou valor) inicial que será considerado para a totalização a cada iteração, e o segundo uma função `nextPartialResult`. Esta segunda função será aplicada para efetivamente totalizar o valor a cada iteração, recebendo dois parâmetros: o primeiro parâmetro, cujo tipo coincide com o valor inicial, será considerado como a entrada para a função, já o segundo é o próximo elemento do _array_, através do qual podemos acessar valores para utilizar na implementação da função. Ao fim das iterações, temos retornado o valor totalizado.

Podemos tornar o código ainda mais simples com:

``` swift
let valorTotal = titulosComMenoresPrecos.reduce(0.0, { $0 + $1.preco })

print(valorTotal)
// 112.69
```

Poderíamos ter uma aplicação ainda mais direta ao lidar com uma coleção de valores brutos relativos aos preços para os livros.

``` swift
let menoresPrecos = [42.90, 47.90, 21.89]

menoresPrecos.reduce(0.0, +)
// 112.69
```

O código acima se utiliza de um operador Swift como implementação base para a função que totaliza dois valores. Se pensarmos a respeito da inferência de tipos na aplicação exemplo, veremos que para este caso o parâmetro `nextPartialResult` espera qualquer função com assinatura compatível com `(Decimal, Decimal) -> Decimal`. Desta forma, é perfeitamente possível inferir a partir do operador `+`, ou qualquer outro operador como, por exemplo, `*`.

O uso do operador (`+`) pode ser percebido também em reduções de coleções de _strings_. Quando esse é o caso, a cada iteração temos uma concatenação dos valores _string_ correntes.

``` swift
let caracteres = ["S", "W", "I", "F", "T"]

print(caracteres.reduce("", +))
// SWIFT
```

Para coleções de `String`, em alguns casos, pode ser indicado o uso de transformações que façam mais sentido para o contexto do valor a ser produzido. Caso tivéssemos um _array_ de nomes, a concatenação simples fornecida pela implementação do operador (`+`) poderia ser insufiente sem considerar algum padrão de separação entre os fragmentos. 

``` swift
let caracteres = ["Rafael", "Rollo"]

print(caracteres.reduce("", +))
// RafaelRollo
```

Nesse caso, seria necessário informar a implementação exata, adicionando um espaço entre os elementos:

``` swift
let nomes = ["Rafael", "Rollo"]

print(nomes.reduce("", { result, nome in
    return result + " " + nome
}))
//  Rafael Rollo
```

Se você se atentar ao retorno, vai perceber que existe um caractere de espaço adicional no início do valor produzido. Isso ocorre porque a implementação fornecida não leva em conta que na primeira iteração temos `"" + " " + "Rafael"`.

É necessário adicionar uma verificação para considerar esse detalhe. Podemos acessar a referência para o índice da iteração através do uso da função `enumerated()` do `Array` que retorna um _wrapper_ contendo o valor inteiro para o índice (_offset_) além do valor de fator (`element`). Assim temos: 

``` swift
let nomes = ["Rafael", "Rollo"]

print(nomes.enumerated().reduce("", { result, nome in
    if nome.offset < 1 {
        return nome.element
    }
    return result + " " + nome.element
}))
// Rafael Rollo
```

Com o código acima conseguimos chegar ao objetivo mas voltando a ter uma implementação verbosa. Isso porque existe meio mais fácil de chegar ao mesmo resultado.

``` swift
let nomes = ["Rafael", "Rollo"]

print(nomes.joined(separator: " "))
// Rafael Rollo
```

A função `joined()` já garante o comportamento esperado, sem a necessidade de qualquer lógica adicional como no caso de `reduce(_:_:)`.

### Compondo transformações para atingir grandes resultados com pouco esforço

Dada a natureza das funções, que retornam novas coleções como resultado das transformações aplicadas, é possível aplicá-las sucessivamente, compondo uma espécie de _pipeline_ de operações em um fluxo contínuo de dados. Esse estilo de escrita de código é uma herança de modelos de programação funcional e a maioria das APIs de linguagens modernas oferecem algum tipo de suporte.

Para que tenhamos uma prévia dos resultados que podemos atingir através da utilização das funções aqui apresentadas, usamos como exemplo o caso da totalização do valor dos livros de menor preço.

``` swift
let livros = [
    Livro("Uma breve história do tempo", por: 54.97),
    Livro("Respostas de um astrofísico", por: 42.90),
    Livro("Breves respostas para grandes questões", por: 47.90),
    Livro("Origens", por: 51.90),
    Livro("Buracos negros", por: 21.89),
]

let valorTotal = livros.filter({ livro in livro.preco <= 50 })
    .map({ livro in livro.preco })
    .reduce(Decimal.zero, { resultado, preco in resultado + preco })

print(valorTotal)
// 112.69
```

No código acima aplicamos em sequência algumas das funções estudadas nesta seção para que, através de cada transformação, obtivéssemos uma nova coleção mais próxima do resultado desejado. Em ordem executamos: (a) filtragem na lista original para garantir a obtenção dos livros com menores preços (menor ou igual à R$ 50.00); (b) mapeamento da lista filtrada de livros para uma lista filtrada de decimais (com os preços) para facilitar a totalização de valor e (c) redução dos preços para um valor decimal total.

É possível tornar o código ainda mais sucinto utilizando as já conhecidas abreviações para nomes de parâmetros das funções.

``` swift
let valorTotal = livros.filter({ $0.preco <= 50 })
    .map({ $0.preco })
    .reduce(Decimal.zero, +)

print(valorTotal)
// 112.69
```
