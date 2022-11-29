# Swift Closures

Closures são blocos independentes de funcionalidade que podem ser passados e usados da mesma forma como dados em seu código. Closures em Swift são semelhantes a blocos em C e Objective-C e a lambdas em outras linguagens de programação.

Closures podem capturar e armazenar referências a quaisquer constantes e variáveis do contexto em que são definidas. Isso é conhecido como enclausurar (daí, _closure_, clausura) essas constantes e variáveis. O Swift cuida de todo o gerenciamento de memória da captura para você.

> Esse material teórico foi traduzido e adaptado da fonte original disponível em https://docs.swift.org/swift-book/LanguageGuide/Closures.html

As funções globais e aninhadas, conforme apresentadas na teoria sobre funções, são, na verdade, casos especiais de _closures_. As _closures_ assumem uma das três formas:

* As funções globais são _closures_ que têm um nome e não capturam nenhum valor;

* As funções aninhadas são _closures_ que têm um nome e podem capturar valores do escopo de sua função delimitadora;

* Expressões _closure_ são _closures_ sem nome, escritos em uma sintaxe leve que pode capturar valores de seu contexto de definição.

As expressões _closure_ do Swift têm um estilo limpo e claro, com otimizações que incentivam uma sintaxe breve e descomplicada em cenários comuns. Essas otimizações incluem:

* Inferir parâmetros e tipos de retorno a partir do contexto;

* Retornos implícitos em  expressões _closures_ de única instrução;

* Nomes de argumentos abreviados

* Sintaxe de trailing _closures_

## Expressões Closure

Funções aninhadas, conforme introduzidas na teoria sobre funções, são um meio conveniente de nomear e definir blocos de código independentes como parte de uma função maior. No entanto, às vezes é útil escrever versões mais curtas de construções semelhantes a funções sem uma declaração e nome completos. Isso é particularmente verdadeiro quando você trabalha com funções ou métodos que aceitam funções como um ou mais de seus argumentos.

Expressões _closure_ são uma maneira de escrever _closures_ inline em uma sintaxe breve e bastante focada. As expressões _closure_ fornecem várias otimizações de sintaxe para escrever _closures_ de forma abreviada sem perda de clareza ou intenção. Os exemplos de expressão _closure_ abaixo ilustram essas otimizações refinando um único exemplo do método `sorted(by:)` em várias iterações, cada uma das quais expressa a mesma funcionalidade de maneira mais sucinta.

### O método `sorted(by:)`

A biblioteca padrão da Swift fornece um método chamado `sorted(by:)`, que classifica um _array_ de valores de um tipo conhecido, com base na saída de uma _closure_ de classificação que você fornece. Depois de concluir o processo de classificação, o método `sorted(by:)` retorna um novo _array_ do mesmo tipo e tamanho do antigo, com seus elementos na ordem correta de classificação. O _array_ original não é modificado pelo método `sorted(by:)`.

Os exemplos de expressão _closure_ abaixo usam o método `sorted(by:)` para classificar um _array_ de valores String em ordem alfabética reversa. Aqui está o _array_ inicial a ser classificado:

``` swift
let nomes = ["Caio", "Alberto", "Eduardo", "Iasmyn", "Danilo", "Matheus"]
```

O método `sorted(by:)` aceita uma _closure_ que recebe dois argumentos do mesmo tipo do conteúdo do _array_ e retorna um valor `Bool` para dizer se o primeiro valor deve aparecer antes ou depois do segundo valor depois de classificados. A _closure_ de classificação precisa retornar `true` se o primeiro valor deve aparecer antes do segundo valor e `false` caso contrário.

Este exemplo está classificando um _array_ de valores String e, portanto, a _closure_ de classificação precisa ser uma função do tipo `(String, String) -> Bool`.

Uma maneira de fornecer a _closure_ de classificação é escrever uma função normal do tipo correto e passá-la como um argumento para o método `sorted(by:)`:

``` swift
func reverse(_ s1: String, _ s2: String) -> Bool {
    return s1 > s2
}

var reversed = nomes.sorted(by: reverse)
// reversed é igual a ["Matheus", "Iasmyn", "Eduardo", "Danilo", "Caio", "Alberto"]
```

Se a primeira string (`s1`) for maior que a segunda string (`s2`), a função `reverse(_:_:)` retornará `true`, indicando que `s1` deve aparecer antes de `s2` no _array_ classificado. Para caracteres em string, “maior que” significa “aparece depois no alfabeto que”. Isso significa que a letra "B" é "maior que" a letra "A" e a string "Danilo" é maior que a string "Daniel". Isso dá uma ordem alfabética reversa, com "Caio" sendo colocado antes de "Alberto", e assim por diante.

No entanto, esta é uma maneira bastante prolixa de escrever o que é essencialmente uma função de expressão única (a > b). Neste exemplo, seria preferível escrever a _closure_ de classificação inline, usando a sintaxe de expressão _closure_.

### Sintaxe da Expressão Closure

A sintaxe da expressão _closure_ tem a seguinte forma geral:

<p align="center">
<img alt="Imagem com a sintaxe da expressão clojure" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-_closures_-sintaxe-expressao-_closure_.png?raw=true" width="70%" />
</p>

Os parâmetros na sintaxe da expressão _closure_ podem ser parâmetros de entrada e saída, mas não podem ter um valor padrão. Parâmetros variadic podem ser usados ​​se você nomear o parâmetro variadic. As tuplas também podem ser usadas como tipos de parâmetro e tipos de retorno.

O exemplo abaixo mostra uma versão da expressão _closure_ da função `reverse(_:_:)` acima:

``` swift
reversed = nomes.sorted(by: { (s1: String, s2: String) -> Bool in
    return s1 > s2
})
```

Observe que a declaração de parâmetros e o tipo de retorno para essa _closure_ inline é idêntica à declaração da função `reverse(_:_:)`. Em ambos os casos, é escrito como `(s1: String, s2: String) -> Bool`. No entanto, para a expressão _closure_ inline, os parâmetros e o tipo de retorno são escritos dentro das chaves, não fora delas.

O início do corpo da _closure_ é introduzido pela palavra-chave `in`. Esta palavra-chave indica que a definição dos parâmetros e tipo de retorno da _closure_ foi concluída e o corpo da _closure_ está prestes a começar.

Como o corpo da _closure_ é tão curto, pode até ser escrito em uma única linha:

``` swift
reversed = nomes.sorted(by: { (s1: String, s2: String) -> Bool in return s1 > s2 } )
```

Isso ilustra que a chamada geral para o método `sorted(by:)` permaneceu a mesma. Um par de parênteses ainda envolve todo o argumento do método. No entanto, esse argumento agora é uma _closure_ inline.

### Inferindo tipo a partir do contexto

Como a _closure_ de classificação é passada como um argumento para um método, o Swift pode inferir os tipos de seus parâmetros e o tipo do valor que ele retorna. O método `sorted(by:)` está sendo chamado em um _array_ de strings, então seu argumento deve ser uma função do tipo `(String, String) -> Bool`. Isso significa que os tipos `(String, String)` e `Bool` não precisam ser escritos como parte da definição da expressão _closure_. Como todos os tipos podem ser inferidos, a seta de retorno `(->)` e os parênteses ao redor dos nomes dos parâmetros também podem ser omitidos:

``` swift
reversed = nomes.sorted(by: { s1, s2 in return s1 > s2 } )
```

É sempre possível inferir os tipos de parâmetro e o tipo de retorno ao passar uma _closure_ para uma função ou método como uma expressão _closure_ inline. Como resultado, você nunca precisa escrever uma _closure_ inline em sua forma completa quando a _closure_ é usada como uma função ou argumento de método.

No entanto, você ainda pode tornar os tipos explícitos, se desejar, e isso é recomendável se evitar ambiguidade para os leitores de seu código. No caso do método `sorted(by:)`, o propósito da _closure_ é claro pelo fato de que a classificação está ocorrendo, e é seguro para um leitor assumir que a _closure_ provavelmente está funcionando com valores String, porque está auxiliando a classificação de um _array_ de strings.

### Retornos implícitos em expressões _closure_ de única instrução

Expressão _closure_ de única instrução podem retornar implicitamente o resultado de sua expressão omitindo a palavra-chave `return` de sua declaração, como nesta versão do exemplo anterior:


``` swift
reversed = nomes.sorted(by: { s1, s2 in s1 > s2 } )
```

Aqui, o tipo da função que define o argumento do método `sorted(by:)` deixa claro que um valor `Bool` deve ser retornado pela _closure_. Como o corpo da _closure_ contém uma única expressão `(s1 > s2)` que retorna um valor `Bool`, não há ambiguidade e a palavra-chave `return` pode ser omitida.

### Nomes de argumento abreviados

O Swift fornece automaticamente nomes de argumentos abreviados para _closures_ inline, que podem ser usados ​​para se referir aos valores dos argumentos da _closure_ pelos nomes `$0`, `$1`, `$2` e assim por diante.

Se você usar esses nomes de argumento abreviados em sua expressão de _closure_, poderá omitir a lista de argumentos da _closure_ de sua definição. O tipo dos nomes dos argumentos abreviados é inferido do tipo de função esperado, e o argumento abreviado de número mais alto que você usa determina o número de argumentos que a _closure_ usa. A palavra-chave `in` também pode ser omitida, porque a expressão _closure_ é composta inteiramente apenas de seu corpo:

``` swift
reversed = nomes.sorted(by: { $0 > $1 } )
```

Aqui, `$0` e `$1` referem-se ao primeiro e segundo argumentos `String` da _closure_. Como `$1` é o argumento abreviado com o número mais alto, entende-se que a _closure_ leva dois argumentos. Como a função `sorted(by:)` aqui espera um encerramento cujos argumentos são strings, os argumentos abreviados `$0` e `$1` são ambos do tipo String.

### Métodos de Operador _(Opetator Methods)_

Na verdade, existe uma maneira ainda mais curta de escrever a expressão _closure_ acima. O tipo `String` do Swift define sua implementação específica de string do operador **maior que** (`>`) como um método que possui dois parâmetros do tipo `String` e retorna um valor do tipo `Bool`. Isso corresponde exatamente ao tipo de método necessário para o método `sorted(by:)`. Portanto, você pode simplesmente passar o operador **maior que**, e o Swift inferirá que você deseja usar sua implementação específica de string:

``` swift
reversed = nomes.sorted(by: >)
```

## Trailing Closures

Se você precisar passar uma expressão _closure_ para uma função como o argumento final da função e a expressão _closure_ for longa, pode ser útil escrevê-la com a sintaxe de _trailing_. Você escreve uma trailing _closure_ após os parênteses da chamada de função, mesmo que a trailing _closure_ ainda seja um argumento para a função. Ao usar a sintaxe de trailing _closure_, você não escreve o rótulo do argumento para a primeira _closure_ como parte da chamada de função. Uma chamada de função pode incluir várias _trailing closure_; no entanto, os primeiros exemplos abaixo usam uma única _trailing closure_.

``` swift
func umaFuncaoQueRecebeUmaClosure(_closure_: () -> Void) {
    // o corpo da função vai aqui
}

// Veja como você chama essa função sem usar sintaxe de trailing _closure_:

umaFuncaoQueRecebeUmaClosure(_closure_: {
    // o corpo da _closure_ vai aqui
})

// Veja como você chama essa função com uma trailing _closure_:

umaFuncaoQueRecebeUmaClosure() {
    // o corpo da _closure_ final vai aqui
}
```

A _closure_ de classificação de string da seção acima pode ser escrito fora dos parênteses do método `sorted(by:)` como uma _trailing closure_:

``` swift
reversed = nomes.sorted() { $0 > $1 }
```

Se uma expressão _closure_ for fornecida como o único argumento da função ou método e você fornecer essa expressão como uma _trailing closure_, não será necessário escrever um par de parênteses `()` após o nome da função ou do método ao chamar a função:

``` swift
reversed = nomes.sorted { $0 > $1 }
```

_Trailing_ _closures_ são mais úteis quando a _closure_ é suficientemente longa para que não seja possível escrevê-la inline em uma única linha. Por exemplo, o tipo `Array` do Swift tem um método `map(_:)`, que usa uma expressão _closure_ como seu único argumento. A _closure_ é chamada uma vez para cada item no _array_ e retorna um valor mapeado alternativo (possivelmente de algum outro tipo) para esse item. Você especifica a natureza do mapeamento e o tipo do valor retornado escrevendo o código na _closure_ que você passa para `map(_:)`.

Depois de aplicar a _closure_ fornecida a cada elemento do _array_, o método `map(_:)` retorna um novo _array_ contendo todos os novos valores mapeados, na mesma ordem de seus valores correspondentes no _array_ original.

Veja como você pode usar o método `map(_:)` com uma _trailing closure_ para converter um _array_ de valores `Int` em um _array_ de valores `String`. O _array_ `[16, 58, 510]` é usado para criar o novo _array_ `["UmSeis", "CincoOito", "CincoUmZero"]`:

``` swift
let digitos = [
    0: "Zero", 1: "Um", 2: "Dois", 3: "Três", 4: "Quatro",
    5: "Cinco", 6: "Seis", 7: "Sete", 8: "Oito", 9: "Nove"
]

let numeros = [16, 58, 510]
```

O código acima cria um dicionário de mapeamentos entre os dígitos inteiros e as versões de seus nomes em português. Também define um _array_ de inteiros, prontos para serem convertidos em strings.

Agora você pode usar o _array_ de números para criar um _array_ de valores String, passando uma expressão _closure_ para o método _map(_:)_ do array como uma _trailing closure_:

``` swift
let strings = numeros.map { (numero) -> String in
    var numero = numero
    var saida = ""
    
    repeat {
        saida = digitos[numero % 10]! + saida
        numero /= 10
    } while numero > 0
    
    return saida
}
// as strings são inferidas como sendo do tipo [String]
// seu valor é ["UmSeis", "CincoOito", "CincoUmZero"]
```

O método `map(_:)` chama a expressão _closure_ uma vez para cada item no _array_. Você não precisa especificar o tipo do parâmetro de entrada da _closure_, número, porque o tipo pode ser inferido a partir dos valores no _array_ a ser mapeado.

Neste exemplo, a variável número é inicializada com o valor do parâmetro número da _closure_, para que o valor possa ser modificado dentro do corpo da _closure_. (Os parâmetros para funções e _closures_ são sempre constantes.) A expressão _closure_ também especifica um tipo de retorno de `String`, para indicar o tipo que será armazenado no _array_ de saída.

A expressão _closure_ cria uma string chamada saida toda vez que é chamada. Ele calcula o último dígito do número usando o operador restante `(numero % 10)` e usa esse dígito para procurar uma _string_ apropriada no dicionário `digitos`. A _closure_ pode ser usada para criar uma representação de _string_ de qualquer número inteiro maior que zero.

> NOTA: A chamada para o subscrito do dicionário `digitos` (`digitos[]`) é seguida por um ponto de exclamação (`!`), porque os subscritos do dicionário retornam um valor opcional para indicar que a pesquisa no dicionário pode falhar se a chave não existir. No exemplo acima, é garantido que o `numero % 10` sempre será uma chave de subscrito válida para o dicionário `digitos` e, portanto, um ponto de exclamação é usado para forçar o _unwrapping_ do valor `String` armazenado no valor de retorno opcional do subscrito.

A _string_ recuperada do dicionário `digitos` é adicionada à frente da saída, construindo efetivamente uma versão _string_ do número ao contrário. (A expressão `numero % 10` dá um valor de `6` para `16`, `8` para `58` e `0` para `510`.)

A variável numérica é então dividida por `10`. Como é um número inteiro, é arredondado para baixo durante a divisão, então `16` se torna `1`, `58` se torna `5` e `510` se torna `51`.

O processo é repetido até que o número seja igual a `0`, ponto em que a _string_ de saída é retornada pela _closure_ e adicionada ao _array_ de saída pelo método `map(_:)`.

O uso da sintaxe de _trailing closures_ no exemplo acima encapsula perfeitamente a funcionalidade da _closure_ imediatamente após a função que a _closure_ suporta, sem a necessidade de envolver toda a _closure_ dentro dos parênteses externos do método `map(_:)`.

Se uma função tiver várias _closures_, você omite o rótulo do argumento para a primeira _trailing closure_ e rotula as _trailing closures_ restantes. Por exemplo, a função abaixo carrega uma imagem para uma galeria de fotos:

``` swift
func carregaFoto(do servidor: Servidor, 
                completionHandler: (Imagem) -> Void,
                failureHandler: () -> Void) {
    if let foto = download("photo.jpg", do: servidor) {
        completionHandler(foto)
    } else {
        failureHandler()
    }
}
```

Ao chamar essa função para carregar uma imagem, você fornece duas _closures_. A primeira é um manipulador de completude, para o caso de sucesso, que exibe uma imagem após um download. A segunda _closure_ é um manipulador de erros que exibe um erro para o usuário.

``` swift
carregaFoto(do: servidor) { foto in
    algumaView.imagem = foto

} failureHandler: {
    print("Não foi possível baixar a próxima foto.")
}
```

Neste exemplo, a função `carregaFoto(do:completionHandler:failureHandler:)` despacha sua tarefa de download em uma _thread_ secundária e chama um dos dois manipuladores de conclusão quando a tarefa de download é concluída. Escrever a função dessa maneira permite separar claramente o código responsável por lidar com uma falha de download do código que atualiza a interface do usuário após um download bem-sucedido, em vez de usar apenas uma _closure_ que lida com ambas as circunstâncias.

## Capturando valores

Uma _closure_ pode capturar constantes e variáveis ​​do contexto ao rodar da qual é definida. A _closure_ pode então se referir e modificar os valores dessas constantes e variáveis ​​de dentro de seu corpo, mesmo que o escopo original que definiu as constantes e variáveis ​​não exista mais.

Em Swift, a forma mais simples de _closure_ que pode capturar valores é uma função aninhada, escrita dentro do corpo de outra função. Uma função aninhada pode capturar qualquer um dos argumentos de sua função externa e também pode capturar quaisquer constantes e variáveis ​​definidas na função externa.

Aqui está um exemplo de uma função chamada `getIncrementador`, que contém uma função aninhada chamada `incrementador`. A função `incrementador()` aninhada captura dois valores, `total` e `padraoDeIncremento`, de seu contexto. Depois de capturar esses valores, `incrementador` é retornado por `getIncrementador` como uma _closure_ que incrementa `total` por `padraoDeIncremento` cada vez que é chamada.

``` swift
func getIncrementador(para padraoDeIncremento: Int) -> () -> Int {
    var total = 0
    
    func incrementador() -> Int {
        total += padraoDeIncremento
        return total
    }

    return incrementador
}
```

O tipo de retorno de `getIncrementador` é `() -> Int`. Isso significa que ele retorna uma função, em vez de um valor simples. A função que ele retorna não tem parâmetros e retorna um valor `Int` toda vez que é chamada.

A função `getIncrementador(para:)` define uma variável inteira chamada `total`, para armazenar o total corrente atual do incrementador que será retornado. Esta variável é inicializada com um valor de `0`.

A função `getIncrementador(para:)` tem um único parâmetro `Int` com um rótulo de argumento de `para` e um nome de parâmetro de `padraoDeIncremento`. O valor do argumento passado para este parâmetro especifica quanto `total` deve ser incrementado cada vez que a função incrementadora retornada é chamada. A função `getIncrementador` define uma função aninhada chamada `incrementador`, que executa o incremento real. Essa função simplesmente adiciona um valor a `total` e retorna o resultado.

Quando considerada isoladamente, a função `incrementador()` aninhada pode parecer incomum:

``` swift
func incrementador() -> Int {
    total += padraoDeIncremento
    return total
}
```

A função `incrementador()` não possui nenhum parâmetro e, ainda assim, refere-se a `total` e `padraoDeIncremento` de dentro do corpo da função. Ele faz isso capturando uma referência a `total` e a `padraoDeIncremento` da função e usando-os dentro de seu próprio corpo de função. Capturar por referência garante que `total` e `padraDeIncremento` não desapareçam quando a chamada para `getIncrementador` terminar e também garante que `total` esteja disponível na próxima vez que a função `incrementador` for chamada.

>Nota: Como uma otimização, o Swift pode, em vez disso, capturar e armazenar uma cópia de um valor se esse valor não for alterado por uma _closure_ e se o valor não for alterado após a criação da _closure_.
>
>O Swift também lida com todo o gerenciamento de memória envolvido na eliminação de variáveis ​​quando elas não são mais necessárias.

Aqui está um exemplo de `getIncrementador` em ação:

``` swift
let incrementaPorDez = getIncrementador(para: 10)
```

Este exemplo define uma constante chamada `incrementaPorDez` para se referir a uma função incrementadora que adiciona `10` à sua variável `total` cada vez que é chamada. Chamar a função várias vezes mostra esse comportamento em ação:

``` swift
incrementaPorDez()
// retorna um valor de 10
incrementaPorDez()
// retorna um valor de 20
incrementaPorDez()
// retorna um valor de 30
```

Se você criar um segundo incrementador, ele terá sua própria referência armazenada para uma nova variável `total` separada:

``` swift
let incrementaPorSete = getIncrementador(para: 7)
incrementaPorSete()
// retorna um valor de 7
```

Chamar o incrementador original (`incrementaPorDez`) novamente continua a incrementar sua própria variável `total` e não afeta a variável capturada por `incrementaPorSete`:

``` swift
incrementaPorDez()
// retorna um valor de 40
```

>Nota: Se você atribuir uma _closure_ a uma propriedade de uma instância de classe e a _closure_ capturar essa instância referindo-se à instância ou a seus membros, você criará um forte ciclo de referência entre a _closure_ e a instância. O Swift usa listas de captura _(capture lists)_ para quebrar esses fortes ciclos de referência. Para obter mais informações, consulte [Ciclos de referência fortes para Closures](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID56).

## Closures são tipos referência

No exemplo acima, `incrementaPorSete` e `incrementaPorDez` são constantes, mas as _closures_ a que essas constantes se referem ainda são capazes de incrementar as variáveis `total` que capturaram. Isso ocorre porque funções e _closures_ são tipos referência.

Sempre que você atribui uma função ou _closure_ a uma constante ou variável, na verdade está configurando essa constante ou variável para ser uma referência à função ou _closure_. No exemplo acima, é a escolha da _closure_ a que `incrementaPorDez` se refere que é constante, e não o conteúdo da própria _closure_.

Isso também significa que, se você atribuir uma _closure_ a duas constantes ou variáveis ​​diferentes, ambas as constantes ou variáveis ​​se referirão à mesma _closure_.

``` swift
let tambemIncrementaPorDez = incrementaPorDez
tambemIncrementaPorDez()
// retorna um valor de 50

incrementaPorDez()
// retorna um valor de 60
```

O exemplo acima mostra que chamar `tambemIncrementaPorDez` é o mesmo que chamar `incrementaPorDez`. Como ambos se referem a mesma _closure_, ambos incrementam e retornam o mesmo total corrente.

## Escape de _closures_

Diz-se que uma _closure_ escapa _(escaping)_ de uma função quando a _closure_ é passada como um argumento para a função, mas é chamada após o retorno da função. Ao declarar uma função que aceita uma _closure_ como um de seus parâmetros, você pode escrever `@escaping` antes do tipo do parâmetro para indicar que a _closure_ pode escapar.

Uma maneira de uma _closure_ escapar é sendo armazenado em uma variável definida fora da função. Por exemplo, muitas funções que iniciam uma operação assíncrona usam um argumento de _closure_ como um _handler_. A função retorna após iniciar a operação, mas a _closure_ não é chamada até que a operação seja concluída - a _closure_ precisa escapar, para ser chamada posteriormente. Por exemplo:

``` swift
var completionHandlers: [() -> Void] = []

func umaFuncaoComEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
```

A função `umaFuncaoComEscapingClosure(_:)` usa uma _closure_ como seu argumento e o adiciona a um _array_ declarado fora da função. Se você não marcasse o parâmetro desta função com `@escaping`, obteria um erro de tempo de compilação.

Uma _escaping closure_ que se refere a `self` precisa de consideração especial se `self` se referir a uma instância de uma classe. Capturar `self` em uma _escaping closure_ facilita a criação acidental de um forte ciclo de referência _(strong reference cycle)_. Para obter informações sobre ciclos de referência, consulte [Contagem automática de referências](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html).

Normalmente, uma _closure_ captura variáveis ​​implicitamente usando-as no corpo da _closure_, mas neste caso você precisa ser explícito. Se você quiser capturar `self`, escreva `self` explicitamente ao usá-lo ou inclua `self` na lista de captura da _closure_. Escrever `self` explicitamente permite que você expresse sua intenção e o lembra de confirmar que não há um ciclo de referências. Por exemplo, no código abaixo, a _closure_ passada para `umaFuncaoComEscapingClosure(_:)` refere-se explicitamente a `self`. Em contraste, a _closure_ passada para `umaFuncaoComNonEscapingClosure(_:)` é uma _closure_ sem escape, o que significa que ela pode se referir `self` implicitamente.

``` swift
func umaFuncaoComNonEscapingClosure(_closure_: () -> Void) {
    _closure_()
}

class AlgumaClasse {
    var x = 10
    func fazAlgo() {
        umaFuncaoComEscapingClosure { self.x = 100 }
        umaFuncaoComNonEscapingClosure { x = 200 }
    }
}

let instancia = AlgumaClasse()
instancia.fazAlgo()
print(instancia.x)
// Imprime "200"

completionHandlers.first?()
print(instancia.x)
// Imprime "100"
```

Aqui está uma versão de `fazAlgo()` que captura `self` incluindo-o na lista de captura da _closure_ e, em seguida, refere-se a `self` implicitamente:

``` swift
class AlgumaOutraClasse {
    var x = 10
    func fazerAlgo() {
        umaFuncaoComEscapingClosure { [self] in x = 100 }
        umaFuncaoComNonEscapingClosure { x = 200 }
    }
}
```

Se `self` for uma instância de uma `struct` ou `enum`, você sempre poderá se referir a `self` implicitamente. No entanto, um _escaping closure_ não pode capturar uma referência mutável a `self` quando `self` é uma instância de uma _struct_ ou uma _enum_. _Structs_ e _enums_ não permitem mutabilidade compartilhada.

``` swift
struct AlgumaStruct {
    var x = 10
    mutating func fazAlgo() {
        umaFuncaoComNonEscapingClosure { x = 200 } // Ok
        umaFuncaoComEscapingClosure { x = 100 } // Erro
    }
}
```

A chamada para a função `umaFuncaoComEscapingClosure` no exemplo acima é um erro porque está dentro de uma _mutating func_, então `self` é mutável. Isso viola a regra de que as _escaping closures_ não podem capturar uma referência mutável a `self` para _structs_.
