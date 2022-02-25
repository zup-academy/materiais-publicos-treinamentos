# Mais Tipos de Dados, Operadores e Estruturas de Controle de Fluxo

>Esse material teórico foi traduzido e adaptado da fonte original disponível em https://docs.swift.org/swift-book/LanguageGuide/TheBasics.html

Swift é a linguagem de programação recomendada para o desenvolvimento de aplicativos iOS, macOS, watchOS e tvOS. No entanto, muitas partes do Swift são familiares à experiência anterior de desenvolvimento com C e Objective-C.

Swift fornece suas próprias versões de todos os tipos fundamentais C e Objective-C, incluindo `Int` para inteiros, `Double` e `Float` para valores de ponto flutuante, `Bool` para valores booleanos e `String` para dados textuais. O Swift também fornece versões poderosas dos três tipos de coleção principais, `Array`, `Set` e `Dictionary`, que serão descritos mais a frente ao estudar coleções.

Além dos tipos familiares, o Swift apresenta tipos avançados não encontrados em Objective-C, como tuplas. As tuplas permitem que você crie e passe adiante agrupamentos de valores. Você pode usar uma tupla para retornar vários valores de uma função como um único valor composto.

O Swift também apresenta tipos opcionais (_Optionals_), que lidam com a ausência de um valor. Os opcionais representam “existe um valor e é igual a x” ou “não existe valor algum”. Usar opcionais é semelhante a usar `nil` com ponteiros em Objective-C, mas eles funcionam para qualquer tipo, não apenas classes. Os opcionais não são apenas mais seguros e expressivos do que ponteiros `nil` no Objective-C, eles estão no centro de muitos dos recursos mais poderosos do Swift.

Swift é uma linguagem _type-safe_, o que significa que a linguagem ajuda você a ter clareza sobre os tipos de valores com os quais seu código pode trabalhar. Se parte do seu código requer uma `String`, essa segurança impede que você passe um `Int` por engano. Da mesma forma, ela evita que você passe acidentalmente uma `String` opcional para um trecho de código que requer uma `String` não opcional. Essa característica da linguagem te ajuda a detectar e corrigir erros o mais cedo possível no processo de desenvolvimento.

## Mais Tipos de Dados

### Integers

Integers são números inteiros sem componente fracionário, como 42 e -23. Os inteiros são com sinal (positivo, zero ou negativo) ou sem sinal (positivo ou zero).

Swift fornece inteiros em formatos de 8, 16, 32 e 64 bits. Esses inteiros seguem uma convenção de nomenclatura semelhante a C, em que um inteiro sem sinal de 8 bits é do tipo `UInt8` e um inteiro com sinal de 32 bits é do tipo `Int32`. Como todos os tipos no Swift, esses tipos inteiros têm nomes em maiúsculas.

#### Limites para inteiros

Você pode acessar os valores mínimo e máximo de cada tipo inteiro com suas propriedades min e max:

``` swift
let min = UInt8.min // min é igual a 0 e é do tipo UInt8

let max = UInt8.max // max é igual a 255 e é do tipo UInt8
```

Os valores dessas propriedades são de tipo numérico de tamanho apropriado (como `UInt8` no exemplo acima) e, portanto, podem ser usados em expressões junto com outros valores do mesmo tipo.

#### Int

Na maioria dos casos, você não precisa escolher um tamanho específico de inteiro para usar no seu código. Swift fornece um tipo inteiro adicional, `Int`, que tem tamanho apropriado baseado na plataforma atual:

* Em uma plataforma 32-bits, `Int` tem o mesmo tamanho que `Int32`.
* Em uma plataforma 64-bits, `Int` tem o mesmo tamanho que `Int64`.

A menos que você precise trabalhar com um tamanho específico de inteiro, sempre use `Int` para valores inteiros em seu código. Isso ajuda a consistência e a interoperabilidade do código. Mesmo em plataformas 32-bits, `Int` pode armazenar qualquer valor entre `-2.147.483.648` e `2.147.483.647` e é grande o suficiente para muitos intervalos de números inteiros. 

#### UInt

O Swift também fornece um tipo _unsigned integer_ (sem representação de sinal, logo, não negativo), `UInt`, também com sua representação e tamanho apropriado baseado na plataforma atual:

* Em uma plataforma 32-bits, `UInt` tem o mesmo tamanho que `UInt32`. Pode armazenar valores entre `0` e `4294967295`
* Em uma plataforma 64-bits, `UInt` tem o mesmo tamanho que `UInt64`. Pode armazenar valores entre `0` e `18446744073709551615`

> Nota: Use `UInt` somente quando precisar especificamente de um tipo inteiro sem representação de sinal com a representação apropriada para a plataforma. Se este não for o caso, `Int` é preferível, mesmo quando os valores conhecidos a serem armazenados sejam não negativos. Um uso consistente de `Int` para valores inteiros ajuda na interoperabilidade do código, evita a necessidade de converter entre diferentes tipos de números e corresponde à inferência do tipo inteiro.

### Números de Ponto Flutuante

Números de ponto flutuante são números com um componente fracionário, como `3,14159`, `0,1` e `-273,15`.

Os tipos de ponto flutuante podem representar um intervalo muito maior de valores do que os tipos inteiros e podem armazenar números muito maiores ou menores do que podem ser armazenados em um `Int`. Swift fornece dois tipos de números de ponto flutuante:

* `Double` representa um número de ponto flutuante em 64 bits.
* `Float` representa um número de ponto flutuante em 32 bits.

> Nota: `Double` tem uma precisão de pelo menos 15 dígitos decimais, enquanto a precisão de `Float` pode ser de apenas 6 dígitos decimais. O tipo de ponto flutuante apropriado a ser usado depende da natureza e do intervalo de valores que você precisa para trabalhar em seu código. Em situações em que qualquer tipo seria apropriado, `Double` é preferível.

### Type Alias

Os aliases de tipo definem um nome alternativo para um tipo existente. Você define aliases de tipo com a palavra-chave `typealias`.

Os aliases de tipo são úteis quando você deseja fazer referência a um tipo existente por um nome que seja contextualmente mais apropriado, como ao trabalhar com dados de um tamanho específico de uma fonte externa:

``` swift
typealias AudioSample = UInt16
```

Depois de definir um alias de tipo, você pode usar o alias em qualquer lugar em que possa usar o nome original:

``` swift
var amplitudeMaximaEncontrada = AudioSample.min
// amplitudeMaximaEncontrada agora é 0
```

Aqui, `AudioSample` é definido como um alias para `UInt16`. Por ser um alias, a chamada para `AudioSample.min` na verdade chama `UInt16.min`, que fornece um valor inicial de `0` para a variável `amplitudeMaximaEncontrada`.

### Booleanos

Swift tem um tipo booleano básico, chamado `Bool`. Os valores booleanos são chamados de lógicos, porque só podem ser verdadeiros ou falsos. Swift fornece dois valores de constantes booleanas, `true` e `false`:

``` swift
let laranjasSaoLaranja = true
let chuchusSaoGostosos = false
```

Os tipos de `laranjasSaoLaranja` e `chuchusSaoGostosos` foram inferidos como `Bool` pelo fato de terem sido inicializados com valores literais booleanos. Assim como com um `Int` e um `Double`, você não precisa declarar constantes ou variáveis como `Bool` se as definir como `true` ou `false` assim que as criar. A inferência de tipo ajuda a tornar o código Swift mais conciso e legível quando inicializa constantes ou variáveis com outros valores cujo tipo já é conhecido.

Os valores booleanos são particularmente úteis quando você trabalha com estruturas condicionais, como um `if`:

``` swift
if chuchusSaoGostosos {
    print("😋, I love chuchu!")
} else {
    print("😫, chuchu é muito ruim.")
}
// Imprime "😫, chuchu é muito ruim."
```

Estruturas condicionais da Swift, como `if`, são abordadas com mais detalhes à frente.

O _type safety_ do Swift impede que valores não booleanos sejam substituídos por `Bool`. O exemplo a seguir relata um erro em tempo de compilação: 

``` swift
let i = 1
if i {
    // esse exemplo não compila e um erro será reportado
}
```

Entretanto, o exemplo apresentado abaixo é perfeitamente válido:

``` swift
let i = 1
if i == 1 {
    // vai compilar com sucesso
}
```

O resultado da comparação `i == 1` é do tipo `Bool` e, portanto, este segundo exemplo passa na verificação de tipo. Comparações como `i == 1` são discutidas na seção de Operadores.

### Tuplas

As tuplas agrupam vários valores em um único valor composto. Os valores dentro de uma tupla podem ser de qualquer tipo e não precisam ser do mesmo tipo uns dos outros.

Neste exemplo, `(404, "Not Found")` é uma tupla que descreve um código de status HTTP.

``` swift
let notFoundStatus = (404, "Not Found")
// notFoundStatus é do tipo (Int, String) e igual a (404, "Not Found")
```

A tupla `(404, "Not Found")` agrupa um `Int` e uma `String` para fornecer ao código de status HTTP dois valores separados: um número e uma descrição legível por humanos. Pode ser descrito como “uma tupla do tipo `(Int, String)`”.

Você pode criar tuplas a partir de qualquer permutação de tipos, e elas podem conter quantos tipos diferentes você desejar. Não há nada que impeça você de ter uma tupla do tipo `(Int, Int, Int)`, ou `(String, Bool)`, ou qualquer outra permutação que você precisar.

Você pode decompor o conteúdo de uma tupla em constantes ou variáveis separadas, que você acessa normalmente:

``` swift
let (statusCode, mensagem) = notFoundStatus

print("O Status Code é \(statusCode)")
// Imprime "O Status Code é 404"

print("A mensagem para o status é: \(mensagem)")
// Imprime "A mensagem para o status é: Not Found"
```

Se você precisar apenas de alguns dos valores da tupla, ignore partes da tupla com um sublinhado (_) ao decompor a tupla:

``` swift
let (apenasStatusCode, _) = notFoundStatus

print("O Status Code é \(apenasStatusCode)")
// Imprime "O Status Code é 404"
```

Como alternativa, acesse os valores dos elementos individuais em uma tupla usando números de índice começando em zero:

``` swift
print("O Status Code é \(notFoundStatus.0)")
// Imprime "O Status Code é 404"

print("A mensagem para o status é: \(notFoundStatus.1)")
// Imprime "A mensagem para o status é: Not Found"
```

Você pode nomear os elementos individuais em uma tupla quando a tupla é definida:

``` swift
let okStatus = (statusCode: 200, mensagem: "OK")
```

Se você nomear os elementos em uma tupla, poderá usar os nomes dos elementos para acessar os valores desses elementos:

``` swift
print("O Status Code é \(okStatus.statusCode)")
// Imprime "O Status Code é 200"

print("A mensagem para o status é: \(okStatus.mensagem)")
// Imprime "A mensagem para o status é: OK"
```

Tuplas são particularmente úteis como valores de retorno de funções. Uma função que tenta recuperar uma página da Web pode retornar o tipo de tupla `(Int, String)` para descrever o sucesso ou a falha da operação. Ao retornar uma tupla com dois valores distintos, cada um de um tipo diferente, a função fornece informações mais úteis sobre seu resultado do que se pudesse retornar apenas um único valor de um único tipo.

> Nota: Tuplas são úteis para grupos simples de valores relacionados. Eles não são adequados para a criação de estruturas de dados complexas. Se sua estrutura de dados provavelmente for mais complexa, modele-a como uma classe ou _struct_, em vez de uma tupla.

### Opcionais

Você usa opcionais em situações em que um valor pode não existir. Um opcional representa duas possibilidades: ou existe um valor e você pode desembrulhar o opcional para acessar esse valor, ou não existe nenhum valor.

> Nota: O conceito de opcionais não existe em C ou Objective-C. A coisa mais próxima em Objective-C é a capacidade de retornar `nil` de um método que, de outra forma, retornaria um objeto, com `nil` significando “a ausência de um objeto válido”. No entanto, isso só funciona para objetos - não funciona para estruturas, tipos básicos de C ou valores de enums. Para esses tipos, os métodos Objective-C normalmente retornam um valor especial (como `NSNotFound`) para indicar a ausência de um valor. Essa abordagem pressupõe que quem invoca o método sabe que há um valor especial para testar e se lembra de verificar. Os opcionais do Swift permitem indicar a ausência de um valor para qualquer tipo, sem a necessidade de constantes especiais.

Aqui está um exemplo de como opcionais podem ser usados para lidar com a ausência de um valor. O tipo `Int` do Swift tem um inicializador que tenta converter um valor `String` em um valor `Int`. No entanto, nem toda _string_ pode ser convertida em um inteiro. A _string_ `"123"` pode ser convertida no valor numérico `123`, mas a string `"hello, world"` não tem um valor numérico óbvio para converter.

O exemplo abaixo usa o inicializador para tentar converter uma `String` em um `Int`:

``` swift
let possivelNumero = "123"
let numeroConvertido = Int(possivelNumero)
// numeroConvertido é inferido como do tipo Int?, ou "inteiro opcional"
```

Como o inicializador pode falhar, ele retorna um `Int` opcional, em vez de um `Int`. Um `Int` opcional é escrito como `Int?`, não como `Int`. O ponto de interrogação indica que o valor que ele contém é opcional, o que significa que pode conter algum valor `Int` ou pode não conter nenhum valor. (Ele não pode conter mais nada, como um valor `Bool` ou um valor `String`. É um `Int` ou não é nada.)

#### nil

Você define uma variável opcional para um estado sem valor, atribuindo-lhe o valor especial `nil`:

``` swift
var responseCode: Int? = 404
// responseCode contem um valor Int de 404

responseCode = nil
// responseCode agora não contém qualquer valor
```

> Nota: Você não pode usar `nil` com constantes e variáveis não opcionais. Se uma constante ou variável em seu código precisar funcionar com a ausência de um valor sob certas condições, sempre a declare como um valor opcional do tipo apropriado.

Se você definir uma variável opcional sem fornecer um valor padrão, a variável será definida automaticamente como `nil`:

``` swift
var resposta: String?
// resposta foi setada automaticamente como nil
```

> Nota: O `nil` de Swift não é o mesmo que `nil` em Objective-C. Em Objective-C, `nil` é um ponteiro para um objeto inexistente. Em Swift, `nil` não é um ponteiro – é a ausência de um valor de um certo tipo. Opcionais de qualquer tipo podem ser definidos como `nil`, não apenas tipos de objeto.

#### `If`s e _Forced Unwrapping_

Você pode usar um `if` para descobrir se um opcional contém um valor comparando o opcional com `nil`. Você executa esta comparação com o operador “igual a” (`==`) ou o operador “diferente de” (`!=`).

Se um opcional tem um valor, é considerado “diferente de” `nil`:

``` swift
if numeroConvertido != nil {
    print("numeroConvertido contem algum valor inteiro.")
}
// Imprime "numeroConvertido contem algum valor inteiro."
```

Depois de ter certeza de que o opcional contém um valor, você pode acessar seu valor subjacente adicionando um ponto de exclamação (!) ao final do nome do opcional. O ponto de exclamação efetivamente diz: “Sei que este opcional definitivamente tem um valor; por favor, use-o.” Isso é conhecido como `forced unwrapping` do valor do opcional:

``` swift
if numeroConvertido != nil {
    print("numeroConvertido tem um valor inteiro de \(numeroConvertido!).")
}
// Imprime "numeroConvertido tem um valor inteiro de 123."
```

> Nota: Tentar usar `!` para acessar um valor opcional inexistente aciona um erro de tempo de execução. **Sempre certifique-se de que um opcional contém um valor diferente de `nil` antes de usar `!`** para forçar o _unwrapping_ de seu valor.

#### Optional Binding

Você usa _optional binding_ para descobrir se um opcional contém um valor e, em caso afirmativo, para disponibilizar esse valor como uma constante ou variável temporária. O _optional binding_ pode ser usado com instruções `if`, por exemplo, para verificar um valor dentro de um opcional e para extrair esse valor em uma constante ou variável, como parte de uma única ação. As instruções `if` são descritas com mais detalhes mais à frente.

Escreva uma _optional binding_ para uma instrução `if` da seguinte forma:

``` swift
if let nomeDaConstante = algumOpcional {
    // instruções
}
```

Você pode reescrever o exemplo de `possivelNumero` da seção acima para usar _optional binding_ em vez do _forced unwrapping_:

``` swift
if let numero = Int(possivelNumero) {
    print("A string \"\(possivelNumero)\" tem um valor inteiro de \(numero).")
} else {
    print("A string \"\(possivelNumero)\" não pôde ser convertida para inteiro.")
}
// Imprime "A string "123" tem um valor inteiro de 123."
``` 

Este código pode ser lido como:

“Se o `Int` opcional retornado por `Int(possivelNumero)` contiver um valor, defina uma nova constante chamada `numero` para o valor contido no opcional.”

Se a conversão for bem-sucedida, a constante `numero` ficará disponível para uso dentro do escopo do `if`. Ele já foi inicializado com o valor contido no opcional e, portanto, você não usa o sufixo `!` para acessar seu valor. Neste exemplo, `numero` é usado simplesmente para imprimir o resultado da conversão.

Você pode usar constantes e variáveis com _optional binding_. Se você quisesse manipular o valor de `numero` dentro do escopo do `if`, você poderia escrever `if var numero`, e o valor contido no opcional seria disponibilizado como uma variável em vez de uma constante.

Você pode incluir quantos _optional bindings_ e condições booleanas forem necessárias em um único `if`, separadas por vírgulas. Se qualquer um dos valores nos _optional bindings_ for `nil` ou qualquer condição booleana for avaliada como falsa, toda a condição do `if` será considerada falsa. As seguintes instruções `if` são equivalentes:

``` swift
if let primeiroNumero = Int("4"), let segundoNumero = Int("42"), primeiroNumero < segundoNumero && segundoNumero < 100 {
    print("\(primeiroNumero) < \(segundoNumero) < 100")
}
// Imprime "4 < 42 < 100"

if let primeiroNumero = Int("4") {
    if let segundoNumero = Int("42") {
        if primeiroNumero < segundoNumero && segundoNumero < 100 {
            print("\(primeiroNumero) < \(segundoNumero) < 100")
        }
    }
}
// Imprime "4 < 42 < 100"
```

> Nota: Constantes e variáveis criadas com _optional binding_ em um `if` estão disponíveis apenas no escopo do `if`. Por outro lado, as constantes e variáveis criadas com uma instrução `guard` estão disponíveis nas linhas de código que seguem a instrução `guard`, conforme veremos adiante.

#### Opcionais Implicitamente Desembrulhados _(unwrapped)_

Conforme descrito acima, os opcionais indicam que uma constante ou variável pode ter “nenhum valor”. Os opcionais podem ser verificados com um `if` e podem ser desembrulhados condicionalmente com um _optional binding_ para acessar o valor do opcional, se existir.

Às vezes fica claro pela estrutura de um programa que um opcional sempre terá um valor, depois que esse valor for definido pela primeira vez. Nesses casos, é útil remover a necessidade de verificar e desembrulhar o valor do opcional toda vez que ele for acessado, pois pode-se presumir com segurança que ele tem um valor o tempo todo.

Esses tipos de opcionais são definidos como opcionais desembrulhados implicitamente. Você escreve um _implicit unwrapped optional_ colocando um ponto de exclamação (`String!`) em vez de um ponto de interrogação (`String?`) após o tipo que deseja tornar opcional. Em vez de colocar um ponto de exclamação após o nome do opcional ao **usá-lo**, você coloca um ponto de exclamação após o tipo do opcional ao **declará-lo**.

Opcionais implicitamente desembrulhados são úteis quando o valor de um opcional é confirmado como existente imediatamente após o opcional ser definido pela primeira vez e pode definitivamente ser assumido como existindo em todos os pontos posteriores. O principal uso de opcionais desembrulhados implicitamente no Swift é durante a inicialização de uma classe.

Um _implicit unwrapped optional_ é um opcional normal por baixo dos panos, mas também pode ser usado como um valor não opcional, sem a necessidade de fazer o _unwrap_ do valor opcional toda vez que for acessado. O exemplo a seguir mostra a diferença de comportamento entre uma string opcional e uma string opcional desembrulhada implicitamente ao acessar seu valor encapsulado como uma `String` explícita:

``` swift
let possivelString: String? = "Uma string opcional."
let stringForcada: String = possivelString! // requer uma exclamação

let stringPresumida: String! = "Uma string opcional desembrulhada implicitamente."
let stringImplicita: String = stringPresumida // não requer exclamação
```

Você pode pensar em um opcional desembrulhado implicitamente como dando permissão para que o opcional seja desempacotado à força, se necessário. Quando você usa um valor opcional desembrulhado implicitamente, o Swift primeiro tenta usá-lo como um valor opcional comum; se não puder ser usado como opcional, o Swift desempacotará o valor à força. No código acima, o valor opcional `stringPresumida` é desembrulhado à força antes de atribuir seu valor a `stringImplicita` porque `stringImplicita` tem um tipo `String` explícito e não opcional. No código abaixo, `stringOpcional` não tem um tipo explícito, então é um opcional comum.

``` swift
let stringOpcional = stringPresumida
// O tipo de stringOpcional é definido como String? e stringPresumida não é desembrulhada de forma forçada
```

Se um opcional desembrulhado implicitamente for `nil` e você tentar acessar seu valor encapsulado, você acionará um erro de tempo de execução. O resultado é exatamente o mesmo que se você colocar um ponto de exclamação após um opcional normal que não contém um valor.

Você pode verificar se um opcional desembrulhado implicitamente é `nil` da mesma forma que verifica um opcional normal:

``` swift
if stringPresumida != nil {
    print(stringPresumida!)
}
// Imprime "Uma string opcional desembrulhada implicitamente."
```

Você também pode usar um opcional desembrulhado implicitamente com _optional binding_, para verificar e desempacotar seu valor em uma única instrução:

``` swift
if let string = stringPresumida {
    print(string)
}
// Imprime "Uma string opcional desembrulhada implicitamente."
```

> Nota: Não use _implicit unwrapped optionals_ quando houver a possibilidade de uma variável se tornar nula posteriormente. Sempre use um tipo opcional normal se precisar verificar um valor `nil` durante o tempo de vida de uma variável.

> Nota: _Implicit unwrapped optionals_ são comumente utilizados na declaração de propriedades armazenadas de _View Controllers_ que são conectadas à _UIViews_ definidas em arquivos de Interface Builder através de _outlets_, já que presumem-se sua inicialização e presença de valores pelo carregamento automático das _views_ garantido pelo UIKit _framework_.

## Operadores em Swift

Um operador é um símbolo especial que você usa para verificar, alterar ou combinar valores. Por exemplo, o operador de adição (`+`) adiciona dois números, como em `let i = 1 + 2`, e o operador lógico _AND_ (`&&`) combina dois valores booleanos, como em `codigoSecretoInformado && escaneamentoDeRetinaConfirmado`.

O Swift suporta os operadores que você já conhece de linguagens como C e melhora vários recursos para eliminar erros comuns de codificação. O operador de atribuição (`=`) não retorna um valor, para evitar que seja usado erroneamente quando o operador igual a (`==`) for pretendido. Operadores aritméticos (`+`, `-`, `*`, `/`, `%` e assim por diante) detectam e não permitem _overflow_ de valor, para evitar resultados inesperados ao trabalhar com números que se tornam maiores ou menores do que o intervalo de valores permitido do tipo que os armazena.

Swift também fornece operadores de intervalo que não são encontrados em C, como `a..<b` e `a...b`, como um atalho para expressar um intervalo de valores.

### Terminologia

Os operadores são unários, binários ou ternários:

* Operadores unários operam em um único alvo (como `-a`). Operadores de prefixo unário aparecem imediatamente antes de seu alvo (como `!b`), e operadores de sufixo unários aparecem imediatamente após seu destino (como `c!`).
* Operadores binários operam entre dois alvos (como `2 + 3`).
* Operadores ternários operam em três alvos. Como C, Swift tem apenas um operador ternário, o operador condicional ternário `(a ? b : c)`.

Os valores que os operadores afetam são operandos. Na expressão `1 + 2`, o símbolo `+` é um operador infixo e seus dois operandos são os valores `1` e `2`.

### Operador de Atribuição: uma revisão

O operador de atribuição (`a = b`) inicializa ou atualiza o valor de `a` com o valor de `b`:

``` swift
let b = 10
var a = 5
a = b
// a agora é igual a 10
```

Se o lado direito da atribuição for uma tupla com vários valores, seus elementos podem ser decompostos em várias constantes ou variáveis de uma só vez:

``` swift
let (x, y) = (1, 2)
// x é igual a 1 e y é igual a 2
```

Ao contrário do operador de atribuição em C e Objective-C, o operador de atribuição em Swift não retorna um valor. A seguinte afirmação não é válida:

```swift
if x = y {
    // Isso não é válido, porque x = y não retorna um valor como o if statement espera.
}
```

Esse recurso evita que o operador de atribuição (`=`) seja usado acidentalmente quando o operador igual a (`==`) for o que realmente se pretende usar. Ao tornar `if x = y` inválido, o Swift ajuda a evitar esses tipos de erros em seu código.

### Operadores aritméticos

O Swift suporta os quatro operadores aritméticos padrão para todos os tipos de números:

* Adição (`+`)
* Subtração (`-`)
* Multiplicação (`*`)
* Divisão (`/`)

``` swift
1 + 2 // igual a 3
5 - 3 // igual a 2
2 * 3 // igual a 6
10,0 / 2,5 // igual a 4,0
```

Ao contrário dos operadores aritméticos em C e Objective-C, os operadores aritméticos Swift não permitem que os valores estourem por padrão.

O operador de adição também é compatível com a concatenação de String:

``` swift
"olá, " + "mundo" // é igual a "olá, mundo"
```

#### Operador de resto

O operador de resto (`a % b`) calcula quantos múltiplos de `b` caberão dentro de `a` e retorna o valor que sobrou (conhecido como resto).

> Nota: O operador de resto (`%`) também é conhecido como operador de módulo em outras linguagens. No entanto, seu comportamento em Swift para números negativos significa que, estritamente falando, é um resto e não uma operação de módulo.

Veja como o operador de resto funciona. Para calcular `9 % 4`, você primeiro calcula quantos `4` cabem dentro de 9:

<p align="center">
<img alt="imagem de exemplo analisando graficamente multiplos de 4 dentro de 9" src="https://docs.swift.org/swift-book/_images/remainderInteger_2x.png" width="40%" />
</p>

Você pode encaixar dois `4`s dentro de `9`, e o restante é `1` (mostrado em laranja).

Em Swift, isso seria escrito como: 

``` swift
9 % 4    // igual a 1
```

Para determinar a resposta para `a % b`, o operador `%` calcula a seguinte equação e retorna resto como sua saída:

a = (b x algum multiplicador) + resto

onde "algum multiplicador" é o maior número de múltiplos de `b` que cabem dentro de `a`.

A inserção de `9` e `4` nesta equação produz:

9 = (4 x 2) + 1

O mesmo método é aplicado ao calcular o restante para um valor negativo de a:

``` swift
-9 % 4   // igual a -1
```

A inserção de `-9` e `4` na equação produz:

-9 = (4 x -2) + -1

dando um valor de resto de -1.

O sinal de `b` é ignorado para valores negativos de `b`. Isso significa que `a % b` e `a % -b` sempre produzem a mesma resposta.

#### Operador Unário de Menos

O sinal de um valor numérico pode ser alternado usando um prefixo `-`, conhecido como operador de menos unário:

``` swift
let tres = 3
let menosTres = -tres // menosTres é igual a -3
let maisTres = -menosTres // maisTres é igual a 3, ou "menos menos três"
```

O operador unário de menos (`-`) é prefixado diretamente antes do valor em que opera, sem nenhum espaço em branco.

#### Operador Unário de Mais

O operador unário de mais (`+`) simplesmente retorna o valor em que opera, sem nenhuma alteração:

``` swift
let menosSeis = -6
let tambemMenosSeis = +menosSeis // tambemMenosSeis é igual a -6
```

Embora o operador unário de mais não faça nada, você pode usá-lo para fornecer simetria em seu código para números positivos ao usar também o operador unário menos para números negativos.

### Operadores de Atribuição Compostos

Assim como C, Swift fornece operadores de atribuição compostos que combinam atribuição (`=`) com outra operação. Um exemplo é o operador de atribuição de adição (`+=`):

``` swift
var a = 1
a += 2
// a agora é igual a 3
```

A expressão `a += 2` é um atalho para `a = a + 2`. Efetivamente, a adição e a atribuição são combinadas em um operador que executa as duas tarefas ao mesmo tempo.

> Nota: Os operadores de atribuição compostos não retornam um valor. Por exemplo, você não pode escrever `let b = a += 2`.

### Operadores de Comparação

Swift suporta os seguintes operadores de comparação:

* Igual a (`a == b`)
* Não igual a (`a != b`)
* Maior que (`a > b`)
* Menor que (`a < b`)
* Maior ou igual a (`a >= b`)
* Menor ou igual a (`a <= b`)

> Nota: O Swift também fornece dois operadores de identidade (`===` e `!==`), que você usa para testar se duas referências de objeto se referem à mesma instância de objeto.

Cada um dos operadores de comparação retorna um valor `Bool` para indicar se a declaração é verdadeira ou não:

``` swift
1 == 1 // verdadeiro porque 1 é igual a 1
2 != 1 // verdadeiro porque 2 não é igual a 1
2 > 1 // verdadeiro porque 2 é maior que 1
1 < 2 // verdadeiro porque 1 é menor que 2
1 >= 1 // verdadeiro porque 1 é maior ou igual a 1
2 <= 1 // falso porque 2 não é menor ou igual a 1
```

Os operadores de comparação são frequentemente usados em instruções condicionais, como em um `if`:

``` swift
let nome = "mundo"

if nome == "mundo" {
    print("Olá, mundo")
} else {
    print("Foi mal \(nome), mas não te reconheço")
}
// Imprime "Olá, mundo", porque nome é realmente igual a "mundo".
```

Você pode comparar duas tuplas se elas tiverem o mesmo tipo e o mesmo número de valores. As tuplas são comparadas da esquerda para a direita, um valor por vez, até que a comparação encontre dois valores que não sejam iguais. Esses dois valores são comparados e o resultado dessa comparação determina o resultado geral da comparação de tuplas. Se todos os elementos são iguais, então as próprias tuplas são iguais. Por exemplo:

``` swift
(1, "zebra") < (2, "apple") // verdadeiro porque 1 é menor que 2; "zebra" e "apple" não são comparados
(3, "apple") < (3, "bird") // verdadeiro porque 3 é igual a 3, e "apple" é menor que "bird"
(4, "dog") == (4, "dog") // verdadeiro porque 4 é igual a 4, e "dog" é igual a "dog"
```

No exemplo acima, você pode ver o comportamento de comparação da esquerda para a direita na primeira linha. Como `1` é menor que `2`, `(1, "zebra")` é considerado menor que `(2, "apple")`, independentemente de quaisquer outros valores nas tuplas. Não importa que `"zebra"` não seja menor que `"apple"`, pois a comparação já é determinada pelos primeiros elementos das tuplas. No entanto, quando os primeiros elementos das tuplas são os mesmos, seus segundos elementos são comparados – isso é o que acontece na segunda e na terceira linha.

As tuplas podem ser comparadas com um determinado operador somente se o operador puder ser aplicado a cada valor nas respectivas tuplas. Por exemplo, conforme demonstrado no código abaixo, você pode comparar duas tuplas do tipo `(String, Int)` porque os valores `String` e `Int` podem ser comparados usando o operador `<`. Por outro lado, duas tuplas do tipo `(String, Bool)` não podem ser comparadas com o operador `<` porque o operador `<` não pode ser aplicado a valores `Bool`.

```swift
("azul", -1) < ("roxo", 1) // funciona e avalia como verdadeiro
("azul", false) < ("roxo", true) // dá erro porque < não pode comparar valores booleanos
```

> Nota: A biblioteca padrão do Swift inclui operadores de comparação de tuplas para tuplas com menos de sete elementos. Para comparar tuplas com sete ou mais elementos, você mesmo deve implementar os operadores de comparação.

### Operador Condicional Ternário

O operador condicional ternário é um operador especial com três partes, que assume a forma `pergunta ? resposta1 : resposta2`. É um atalho para avaliar uma das duas expressões com base em se a pergunta é verdadeira ou falsa. Se `pergunta` for `true`, ele avalia `resposta1` e retorna seu valor; caso contrário, ele avalia `resposta2` e retorna seu valor.

O operador condicional ternário é uma abreviação para o código abaixo:

``` swift
if pergunta {
    resposta1
} else {
    resposta2
}
```

Aqui está um exemplo, que calcula a altura de uma linha da tabela. A altura da linha deve ser 50 pontos mais alta do que a altura do conteúdo se a linha tiver um cabeçalho e 20 pontos mais alta se a linha não tiver um cabeçalho:

``` swift
let alturaDoConteudo = 40
let temCabecalho = true
let alturaDaLinha = alturaDoConteudo + (temCabecalho ? 50 : 20)
// alturaDaLinha é igual a 
```

O exemplo acima é uma abreviação para o código abaixo:

``` swift
let alturaDoConteudo = 40
let temCabecalho = true

let alturaDaLinha: Int

if temCabecalho {
    alturaDaLinha = alturaDoConteudo + 50
} else {
    alturaDaLinha = alturaDoConteudo + 20
}
// alturaDaLinha é igual a 90
```

O uso do operador condicional ternário no primeiro exemplo significa que `alturaDaLinha` pode ser definido com o valor correto em uma única linha de código, que é mais conciso do que o código usado no segundo exemplo.

O operador condicional ternário fornece um atalho eficiente para decidir qual das duas expressões considerar. No entanto, use o operador condicional ternário com cuidado. Sua concisão pode levar a um código difícil de entender se usado sem alguma reflexão a respeito. Evite combinar vários _statements_ do operador condicional ternário em uma instrução composta.

### Operador Nil-Coalescing

O operador _nil-coalescing_ (`a ?? b`) desembrulha um opcional `a` se contiver um valor ou retorna um valor _default_ `b` se `a` for `nil`. A expressão `a` é sempre de um tipo opcional. A expressão `b` deve corresponder ao tipo armazenado dentro de `a`.

O operador _nil-coalescing_ é uma abreviação para o código abaixo:

``` swift
a != nil ? a! : b
```

O código acima usa o operador condicional ternário e o _forced unwrapping_ (`a!`) para acessar o valor encapsulado em `a` quando `a` não for nulo e retornar `b` caso contrário. O operador _nil-coalescing_ fornece uma maneira mais elegante de encapsular essa verificação condicional e _unwrapping_ de uma forma concisa e legível.

> Nota: Se o valor de `a` for diferente de `nil`, o valor de `b` não será avaliado. Isso é conhecido como avaliação de curto-circuito.

O exemplo abaixo usa o operador _nil-coalescing_ para escolher entre um nome de cor padrão e um nome de cor opcional definido pelo usuário:

``` swift
let corPadrao = "vermelho"
var corDefinidaPeloUsuario: String? // define para nil

var cor = corDefinidaPeloUsuario ?? corPadrao
// corDefinidaPeloUsuario é nil, então cor é definido como o padrão "vermelho"
```

A variável `corDefinidaPeloUsuario` é definida como uma `String` opcional, com um valor padrão de `nil`. Como `corDefinidaPeloUsuario` é de um tipo opcional, você pode usar o operador _nil-coalescing_ para considerar seu valor. No exemplo acima, o operador é usado para determinar um valor inicial para uma variável `String` chamada `cor`. Como `corDefinidaPeloUsuario` é `nil`, a expressão `corDefinidaPeloUsuario ?? corPadrao` retorna o valor de `corPadrao` ou `"vermelho"`.

Se você atribuir um valor não nulo a `corDefinidaPeloUsuario` e executar a verificação do operador _nil-coalescing_ novamente, o valor encapsulado dentro de `corDefinidaPeloUsuario` será usado em vez do padrão:

``` swift
corDefinidaPeloUsuario = "preto"
cor = corDefinidaPeloUsuario ?? corPadrao
// corDefinidaPeloUsuario não é nil, então cor é definido como "preto"
```

### Operadores Lógicos

Os operadores lógicos modificam ou combinam os valores lógicos booleanos `true` e `false`. O Swift suporta os três operadores lógicos padrão encontrados em linguagens baseadas em C:

* _NOT_ Lógico (`!a`)
* _AND_ Lógico (`a && b`)
* _OR_ Lógico (`a || b`)

#### Operador _NOT_ Lógico

O operador lógico _NOT_ (`!a`) inverte um valor booleano para que `true` se torne `false` e `false` se torne `true`.

O operador lógico _NOT_ é um operador de prefixo e aparece imediatamente antes do valor em que opera, sem nenhum espaço em branco. Pode ser lido como _“not a”_ ou _“não a”_, como pode ser visto no exemplo a seguir:

``` swift
let entradaPermitida = false

if !entradaPermitida {
     print("ACESSO NEGADO")
}
// Imprime "ACESSO NEGADO"
```

A frase `if !entradaPermitida` pode ser lida como _“se não for permitida a entrada”_. A linha subsequente só é executada se _“entrada não permitida”_ for verdadeira; isto é, se `entradaPermitida` for falsa.

Como neste exemplo, a escolha cuidadosa de nomes de constantes e variáveis booleanas pode ajudar a manter o código legível e conciso, evitando duplas negativas ou declarações lógicas confusas.

#### Operador _AND_ Lógico

O operador lógico _AND_ (`a && b`) cria expressões lógicas em que ambos os valores devem ser verdadeiros para que a expressão geral também seja verdadeira.

Se um dos valores for falso, a expressão geral também será falsa. Na verdade, se o primeiro valor for false, o segundo valor nem será avaliado, porque não pode fazer com que a expressão geral seja igual a `true`. Isso é conhecido como avaliação de curto-circuito.

Este exemplo considera dois valores `Bool` e só permite o acesso se ambos os valores forem verdadeiros:

codigoSecretoInformado && escaneamentoDeRetinaConfirmado

``` swift
let codigoSecretoInformado = true
let escaneamentoDeRetinaConfirmado = false

if codigoSecretoInformado && escaneamentoDeRetinaConfirmado {
     print("Bem-vindo!")
} else {
     print("ACESSO NEGADO")
}
// Imprime "ACESSO NEGADO"
```

#### Operador _OR_ Lógico

O operador OR lógico (`a || b`) é um operador feito de dois caracteres pipe que atua entre dois operandos alvo. Você o usa para criar expressões lógicas nas quais apenas um dos dois valores precisa ser verdadeiro para que a expressão geral seja verdadeira.

Como o operador lógico _AND_ acima, o operador lógico _OR_ usa avaliação de curto-circuito para considerar suas expressões. Se o lado esquerdo de uma expressão lógica _OR_ for verdadeiro, o lado direito não será avaliado, pois não pode alterar o resultado da expressão geral.

No exemplo abaixo, o primeiro valor `Bool` (hasDoorKey) é false, mas o segundo valor (knowsOverridePassword) é true. Como um valor é verdadeiro, a expressão geral também é avaliada como verdadeira e o acesso é permitido:

``` swift
let temAChaveDaPorta = false
let sabeASenhaDeSubstituicaoEmergencial = true

if temAChaveDaPorta || sabeASenhaDeSubstituicaoEmergencial {
     print("Bem-vindo!")
} else {
     print("ACESSO NEGADO")
}
// Imprime "Bem-vindo!"
```

#### Combinando Operadores Lógicos

Você pode combinar vários operadores lógicos para criar expressões compostas mais longas:

``` swift
if codigoSecretoInformado && escaneamentoDeRetinaConfirmado || temAChaveDaPorta || sabeASenhaDeSubstituicaoEmergencial {
    print("Bem-vindo!")
} outro {
    print("ACESSO NEGADO")
}
// Imprime "Bem-vindo!"
```

Este exemplo usa vários operadores `&&` e `||` para criar uma expressão composta mais longa. No entanto, os operadores `&&` e `||` ainda operam em apenas dois valores, portanto, na verdade, são três expressões menores encadeadas. O exemplo pode ser lido como:

    "Se informamos o código secreto correto e passamos no escaneamento de retina, ou se temos uma chave para a porta válida, ou se sabemos a senha de substituição de emergência, permita o acesso."

Com base nos valores de `codigoSecretoInformado`, `escaneamentoDeRetinaConfirmado` e `temAChaveDaPorta`, as duas primeiras subexpressões são falsas. No entanto, a senha de substituição de emergência é conhecida, portanto, a expressão composta geral ainda é avaliada como verdadeira.

> NOTA: Os operadores lógicos Swift `&&` e `||` são associativas à esquerda, o que significa que expressões compostas com vários operadores lógicos avaliam primeiro a subexpressão mais à esquerda.

##### Combinando Operadores com Parênteses Explícitos

Às vezes, é útil incluir parênteses quando não são estritamente necessários, para facilitar a leitura da intenção de uma expressão complexa. No exemplo de acesso à porta acima, é útil adicionar parênteses em torno da primeira parte da expressão composta para tornar sua intenção explícita:

``` swift
if (codigoSecretoInformado && escaneamentoDeRetinaConfirmado) || temAChaveDaPorta || sabeASenhaDeSubstituicaoEmergencial {
     print("Bem-vindo!")
} outro {
     print("ACESSO NEGADO")
}
// Imprime "Bem-vindo!"
```

Os parênteses deixam claro que os dois primeiros valores são considerados como parte de um estado possível separado na lógica geral. A saída da expressão composta não muda, mas a intenção geral é mais clara para o leitor. A legibilidade é sempre preferível à brevidade; use parênteses onde eles ajudam a tornar suas intenções claras.

## Estruturas de Controle de Fluxo

O Swift fornece uma variedade de instruções de fluxo de controle. Isso inclui laços de repetição para executar uma tarefa várias vezes; instruções `if`, `guard` e `switch` para executar diferentes ramificações de código com base em certas condições; e instruções como `return` para interromper o fluxo de execução em um ponto em seu código.

A instrução `switch` do Swift é consideravelmente mais poderosa do que suas semelhantes em muitas linguagens de programação _C-like_. Os _cases_ podem corresponder a muitos padrões diferentes, incluindo _matching_ de intervalos, tuplas e _castings_ para um tipo específico. Os valores que dão _match_ com algum padrão em um `case` podem ser vinculados a constantes ou variáveis temporárias (_value bindings_) para uso no corpo do `case`, e padrões de _match_ complexos podem se utilizar de uma cláusula `where` para cada caso.

### Estruturas Condicionais

Muitas vezes é útil executar diferentes partes de código com base em certas condições. Você pode querer executar um código extra quando ocorrer um erro ou exibir uma mensagem quando um valor se tornar muito alto ou muito baixo. Para fazer isso, você condiciona partes do seu código.

O Swift fornece duas maneiras principais de adicionar ramificações condicionais ao seu código: `if` e `switch`. Normalmente, você usa a instrução `if` para avaliar condições simples com apenas alguns resultados possíveis. A instrução `switch` é mais adequada para condições mais complexas com várias permutações possíveis e é útil em situações em que _pattern matching_ pode ajudar a selecionar uma ramificação de código apropriada para execução.

#### If

Em sua forma mais simples, um `if` tem uma única condição. Ele executa um conjunto de instruções somente se essa condição for verdadeira.

``` swift
var temperaturaEmCelsius = 7
if temperaturaEmCelsius <= 10 {
    print("Tá bem frio. Se for sair leva um casaco.")
}
// Imprime "Tá bem frio. Se for sair leva um casaco."
```

O exemplo acima verifica se a temperatura é menor ou igual a 10 graus Celsius. Se estiver, uma mensagem é impressa. Caso contrário, nenhuma mensagem é impressa e a execução do código continua após a chave de fechamento do `if`.

A instrução `if` pode fornecer um conjunto alternativo de instruções, conhecido como cláusula _senão_, para situações em que a condição do `if` é falsa. Essas instruções são indicadas pela palavra-chave `else`.

``` swift
temperaturaEmCelsius = 15
if temperaturaEmCelsius <= 10 {
    print("Tá bem frio. Se for sair leva um casaco.")
} else {
    print("Talvez já dê pra arriscar uma camisetinha, hein!?")
}
// Imprime "Talvez já dê pra arriscar uma camisetinha, hein!?"
```

Uma dessas duas ramificações é sempre executada. Como a temperatura aumentou para 15 graus Celsius, talvez já não esteja mais tão frio para aconselhar o uso de um casaco, portanto, a outra _branch_ é acionada.

Você pode encadear várias instruções `if` juntas para considerar cláusulas adicionais.

``` swift
temperaturaEmCelsius = 30
if temperaturaEmCelsius <= 10 {
    print("Tá bem frio. Se for sair leva um casaco.")
} else if temperaturaEmCelsius >= 26 {
    print("Tá calor. Se for se expor ao sol, não se esqueça do protetor solar.")
} else {
    print("Talvez já dê pra arriscar uma camisetinha, hein!?")
}
// Imprime "Tá calor. Se for se expor ao sol, não se esqueça do protetor solar."
```

Aqui, uma instrução `if` adicional foi utilizada para responder a temperaturas particularmente altas. A cláusula final `else` permanece e imprime uma resposta para quaisquer temperaturas que não sejam nem muito frias nem muito quentes.

A cláusula final `else` é opcional, no entanto, e pode ser excluída se o conjunto de condições não precisar ser completo.

``` swift
temperaturaEmCelsius = 17
if temperaturaEmCelsius <= 10 {
    print("Tá bem frio. Se for sair leva um casaco.")
} else if temperaturaEmCelsius >= 26 {
    print("Tá calor. Se for se expor ao sol, não se esqueça do protetor solar.")
}
```

Como a temperatura não é nem muito fria nem muito quente para acionar as condições `if` ou `else if`, nenhuma mensagem é impressa.

#### Switch

Uma estrutura `switch` considera um valor e o compara com vários padrões de correspondência possíveis. Em seguida, ele executa um bloco de código apropriado, com base no primeiro padrão que corresponde com sucesso. Uma instrução `switch` fornece uma alternativa à instrução `if` para responder a vários estados potenciais.

Em sua forma mais simples, uma instrução switch compara um valor com um ou mais valores do mesmo tipo.

``` swift
switch algum-valor-a-ser-considerado {
case valor1:
    implementação respondendo ao valor1
case valor2, 
     valor3:
    implementação respondendo ao valor 2 ou 3
default:
    caso contrário, faça outra coisa aqui
}
```

Cada instrução `switch` consiste em vários casos possíveis, cada um dos quais começa com a palavra-chave `case`. Além de comparar com valores específicos, o Swift fornece várias maneiras para cada _case_ especificar padrões de reconhecimento mais complexos. Essas opções são descritas posteriormente abaixo.

Assim como o corpo de uma instrução `if`, cada `case` é uma _branch_ separada da execução do código. A instrução `switch` determina qual _branch_ deve ser selecionada. Este procedimento é conhecido como _switching_ para o valor que está sendo considerado.

Cada instrução `switch` deve ser completa. Ou seja, cada valor possível do tipo que está sendo considerado deve dar _match_ com algum dos _cases_. Se não for apropriado fornecer um caso para cada valor possível, você pode definir um caso padrão para cobrir quaisquer valores que não sejam endereçados explicitamente. Esse caso padrão é indicado pela palavra-chave `default` e deve sempre aparecer por último.

Este exemplo usa uma instrução `switch` para considerar um único caractere minúsculo chamado `algumCaractere`:

``` swift
let algumCaractere: Character = "z"

switch algumCaractere {
case "a":
    print("A primeira letra do alfabeto")
case "z":
    print("A última letra do alfabeto")
default:
    print("Algum outro caractere")
}
// Imprime "A última letra do alfabeto"
```

O primeiro caso da instrução `switch` corresponde à primeira letra do alfabeto, `a`, e seu segundo caso corresponde à última letra, `z`. Como o `switch` deve ter maiúsculas e minúsculas para cada caractere possível, não apenas para cada caractere alfabético, essa instrução `switch` usa um caso padrão para corresponder a todos os caracteres diferentes de `a` e `z`. Essa disposição garante que a instrução switch seja completa.

##### Sem _Fallthrough_ Implícito

Em contraste com as instruções _switch_ em C e Objective-C, as instruções _switch_ em Swift não caem para o próximo caso por padrão (_fallthrough_) ao fim da implementação decada caso. Em vez disso, toda a instrução `switch` termina sua execução assim que o primeiro caso do _switch_ correspondente é concluído, sem exigir uma instrução `break` explícita. Isso torna a instrução `switch` mais segura e fácil de usar do que a de C e evita a execução de mais de um `switch` por engano.

> Nota: Embora `break` não seja necessário no Swift, você pode usar uma instrução `break` para corresponder e ignorar um caso específico ou para quebrar um caso correspondido antes que o caso conclua sua execução. veremos mais adiante

O corpo de cada _case_ deve conter pelo menos uma instrução executável. Não é válido escrever o código a seguir, porque o primeiro _case_ está vazio:

``` swift
let outroCaractere: Character = "a"

switch outroCaractere {
case "a": // Inválido, o caso tem um corpo vazio
case "A":
    print("A letra A")
default:
    print("Não é a letra A")
}
// Isso reportará um erro em tempo de compilação.
```

Ao contrário de uma instrução `switch` em C, esta instrução `switch` não corresponde a `"a"` e `"A"`. Em vez disso, ele relata um erro em tempo de compilação que informa `case "a": não contém nenhuma instrução executável`. Essa abordagem evita falhas acidentais de um caso para outro e cria um código mais seguro e mais claro em sua intenção.

Para fazer uma troca com um único caso que corresponda a `"a"` e `"A"`, combine os dois valores em um caso composto, separando os valores com vírgulas.

``` swift
let outroCaractere: Character = "a"

switch outroCaractere {
case "a", "A":
    print("A letra A")
default:
    print("Não é a letra A")
}
// Imprime "A letra A"
```

Para facilitar a leitura, um caso composto também pode ser escrito em várias linhas, como no exemplo de estrutua logo ao início dessa seção.

> Nota: Para passar explicitamente para o caso abaixo, ao final de um caso de `switch` específico, use a palavra-chave `fallthrough`, conforme descrito mais adiante.

##### Matching com Tuplas

Você pode usar tuplas para testar vários valores na mesma instrução `switch`. Cada elemento da tupla pode ser testado em relação a um valor ou intervalo de valores diferente. Como alternativa, use o caractere sublinhado (`_`), também conhecido como _wildcard pattern_ (curinga), para corresponder a qualquer valor possível.

O exemplo abaixo pega um ponto `(x, y)`, expresso como uma tupla simples do tipo `(Int, Int)`, e o categoriza no gráfico que segue o exemplo.

``` swift
let ponto = (1, 1)

switch ponto {
case (0, 0):
    print("\(ponto) está na origem")
case (_, 0):
    print("\(ponto) está no eixo x")
case (0, _):
    print("\(ponto) está no eixo y")
case (-2...2, -2...2):
    print("\(ponto) está dentro da caixa")
default:
    print("\(ponto) está fora da caixa")
}
// Imprime "(1, 1) está dentro da caixa"
```

<p align="center">
<img alt="imagem com gráfico no plano cartesiano relacionado ao exemplo de código para o switch, com uma caixa azul 4 por 4 tendo como centro a origem" src="https://docs.swift.org/swift-book/_images/coordinateGraphSimple_2x.png" width="50%" />
</p>

A instrução `switch` determina se o ponto está na origem `(0, 0)`, no eixo x vermelho, no eixo y verde, dentro da caixa azul 4 por 4 centrada na origem ou fora da caixa.

Ao contrário de C, Swift permite que vários casos de `switch` considerem o mesmo valor ou valores. Na verdade, o ponto `(0, 0)` pode corresponder a todos os quatro casos neste exemplo. No entanto, se várias correspondências forem possíveis, o primeiro caso de correspondência será sempre usado. O ponto `(0, 0)` corresponderia ao primeiro caso `(0, 0)` e, portanto, todos os outros casos correspondentes seriam ignorados.

##### Value Bindings

Um caso de `switch` pode nomear o valor ou valores correspondentes a constantes ou variáveis temporárias, para uso no corpo do _case_. Esse comportamento é conhecido como _value binding_ (algo como vinculação de valor), porque os valores são vinculados à constantes ou variáveis temporárias no escopo do caso.

O exemplo abaixo pega um ponto `(x, y)`, expresso como uma tupla do tipo `(Int, Int)`, e o categoriza no gráfico a seguir:

``` swift
let outroPonto = (2, 0)

switch outroPonto {
case (let x, 0):
     print("no eixo x com um valor x de \(x)")
case (0, let y):
     print("no eixo y com um valor y de \(y)")
case let (x, y):
     print("em outro lugar em (\(x), \(y))")
}
// Imprime "no eixo x com um valor x de 2"
```

<p align="center">
<img alt="imagem com gráfico no plano cartesiano relacionado ao exemplo de código para o switch, com o eixo x destacado em vermelho e o eixo y destacado em verde" src="https://docs.swift.org/swift-book/_images/coordinateGraphMedium_2x.png" width="50%" />
</p>

A instrução `switch` determina se o ponto está no eixo x vermelho, no eixo y verde ou em outro lugar (em nenhum dos eixos).

Os três casos declaram constantes placeholder `x` e `y`, que assumem temporariamente um ou ambos os valores de tupla de `outroPonto`. O primeiro caso, `case (let x, 0)`, corresponde a qualquer ponto com um valor y de 0 e atribui o valor x do ponto à constante temporária `x`. Da mesma forma, o segundo caso, `case (0, let y)`, corresponde a qualquer ponto com um valor x de 0 e atribui o valor y do ponto à constante temporária `y`.

Depois que as constantes temporárias são declaradas, elas podem ser usadas dentro do bloco de código do caso. Aqui, eles são usados para imprimir a categorização do ponto.

Esta instrução `switch` não tem um caso padrão (`default`). O case final, `case let (x, y)`, declara uma tupla de duas constantes placeholder que podem corresponder a qualquer valor. Como `outroPonto` é sempre uma tupla de dois valores, esse caso corresponde a todos os valores restantes possíveis e um _default case_ não é necessário para tornar a instrução `switch` completa.

##### Where

Um _switch case_ pode usar uma cláusula `where` para verificar condições adicionais.

O exemplo abaixo categoriza um ponto `(x, y)` no gráfico a seguir:

``` swift
let maisUmPonto = (1, -1)

switch maisUmPonto {
case let (x, y) where x == y:
     print("(\(x), \(y)) está na linha x == y")
case let (x, y) where x == -y:
     print("(\(x), \(y)) está na linha x == -y")
case let (x, y):
     print("(\(x), \(y)) é apenas um ponto arbitrário")
}
// Imprime "(1, -1) está na linha x == -y"
```

<p align="center">
<img alt="imagem com gráfico no plano cartesiano relacionado ao exemplo de código para o switch, com duas linhas diagonais destacadas em verde e roxo" src="https://docs.swift.org/swift-book/_images/coordinateGraphComplex_2x.png" width="50%" />
</p>

A instrução `switch` determina se o ponto está na linha diagonal verde onde `x == y`, na linha diagonal roxa onde `x == -y`, ou nenhum dos dois.

Os três casos do `switch` declaram constantes de placeholder `x` e `y`, que assumem temporariamente os dois valores de tupla de `maisUmPonto`. Essas constantes são usadas como parte de uma cláusula `where`, para criar um filtro dinâmico. O _switch case_ da match com o valor atual do ponto somente se a condição da cláusula `where` for avaliada como verdadeira para esse valor.

Como no exemplo anterior, o caso final corresponde a todos os valores restantes possíveis e, portanto, um _default case_ não é necessário para tornar a instrução `switch` completa.

#### Estruturas de Transferência de Controle

As instruções de transferência de controle alteram a ordem na qual seu código é executado, transferindo o controle de um pedaço de código para outro. Swift tem cinco instruções de transferência de controle, entre as quais citaremos aqui:

* Break
* Fallthrough
* Return

As instruções `break` e `fallthrough` são descritas abaixo. A instrução `return` já foi descrita no material teórico introdutório sobre Funções.

##### Break

A instrução `break` encerra a execução de uma estrutura de controle de fluxo inteira imediatamente. A instrução `break` pode ser usada dentro de uma instrução `switch` ou laços de repetição quando você deseja encerrar a execução antes do que seria normalmente.

###### Break em uma instrução de switch

Quando usado dentro de uma instrução `switch`, `break` faz com que a instrução `switch` termine sua execução imediatamente e transfira o controle para o código após a chave de fechamento da instrução `switch` (`}`).

Esse comportamento pode ser usado para corresponder e ignorar um ou mais casos em uma instrução `switch`. Como a instrução `switch` do Swift é exaustiva e não permite casos vazios, às vezes é necessário combinar e ignorar deliberadamente um caso para tornar suas intenções explícitas. Você faz isso escrevendo a instrução `break` representando todo o corpo do caso que deseja ignorar. Quando esse _case_ é correspondido pela instrução `switch`, a instrução `break` dentro do _case_ encerra a execução da instrução `switch` imediatamente.

> Nota: Um _switch case_ que contém apenas um comentário é relatado como um erro em tempo de compilação. Comentários não são instruções e não fazem com que um _switch case_ seja ignorado. Sempre use uma instrução `break` para ignorar um _switch case_.

O exemplo a seguir avalia o valor de um `Character` e determina se ele representa um símbolo numérico em um dos quatro idiomas. Por questões de brevidade, vários valores são cobertos em um único _switch case_.

``` swift
let simboloNumerico: Character = "三" // Símbolo chinês para o número 3
var possivelValorInteiro: Int?

switch simboloNumerico {
case "1", "١", "一", "๑":
    possivelValorInteiro = 1
case "2", "٢", "二", "๒":
    possivelValorInteiro = 2
case "3", "٣", "三", "๓":
    possivelValorInteiro = 3
case "4", "٤", "四", "๔":
    possivelValorInteiro = 4
default:
    break
}

if let valorInteiro = possivelValorInteiro {
    print("O valor inteiro de \(simboloNumerico) é \(valorInteiro).")
} else {
    print("Não foi possível encontrar um valor inteiro para \(simboloNumerico).")
}
// Imprime "O valor inteiro de 三 é 3."
```

Este exemplo verifica `simboloNumerico` para determinar se é um símbolo latino, árabe, chinês ou tailandês para os números de `1` a `4`. Se um _match_ ocorrer, um dos casos da instrução `switch` define uma variável do tipo inteiro opcional (`Int?`) chamada `possivelValorInteiro` para um valor inteiro apropriado.

Depois que a instrução `switch` conclui sua execução, o exemplo usa o _optional binding_ para determinar se um valor foi encontrado. A variável `possivelValorInteiro` tem um valor inicial implícito de `nil` em virtude de ser um tipo opcional e, portanto, o _optional binding_ terá sucesso somente se `possivelValorInteiro` tiver sido definido como um valor real por um dos quatro primeiros casos da instrução `switch`.

Como não é prático listar todos os valores de caracteres possíveis no exemplo acima, um _default case_ trata de quaisquer caracteres que não correspondam. Esse _default case_ não precisa executar nenhuma ação e, portanto, é escrito com uma única instrução `break` como seu corpo. Assim que o _default case_ é correspondido, a instrução `break` encerra a execução da instrução `switch` e a execução do código continua a partir da instrução `if let`.

##### Fallthrough

No Swift, as instruções `switch` não caem por padrão para o próximo caso (_fallthrough_). Ou seja, toda a instrução `switch` conclui sua execução assim que o primeiro caso correspondente é concluído. Por outro lado, C exige que você insira uma instrução `break` explícita no final de cada _switch case_ para evitar falhas. Evitar o _fallthrough_ padrão significa que as instruções de `switch` Swift são muito mais concisas e previsíveis do que suas equivalentes em C e, portanto, evitam a execução de vários casos de `switch` por engano.

Se você precisar de um comportamento de _fallthrough_ no estilo C, poderá optar por esse comportamento caso a caso com a palavra-chave `fallthrough`. O exemplo abaixo usa `fallthrough` para criar uma descrição textual de um número.

``` swift
let inteiroParaDescrever = 5
var descricao = "O número \(inteiroParaDescrever) é"

switch inteiroParaDescrever {
case 2, 3, 5, 7, 11, 13, 17, 19:
    descricao += " um número primo, e também"
    fallthrough
default:
    descricao += " um inteiro."
}

print(descrição)
// Imprime "O número 5 é um número primo e também um inteiro."
```

Este exemplo declara uma nova variável `String` chamada `descricao` e atribui a ela um valor inicial. A função então considera o valor de `inteiroParaDescrever` usando uma instrução `switch`. Se o valor de `inteiroParaDescrever` for um dos números primos na lista, a função anexará texto ao final da descrição, para observar que o número é primo. Em seguida, ele usa a palavra-chave `fallthrough` para “cair” no _default case_ também. O _default case_ adiciona algum texto extra ao final da descrição e a instrução `switch` está completa.

A menos que o valor de `inteiroParaDescrever` esteja na lista de números primos conhecidos, ele não corresponde ao primeiro caso do `switch`. Como não há outros casos específicos, `inteiroParaDescrever` corresponde ao _default case_.

Após a conclusão da execução da instrução `switch`, a descrição do número é impressa usando a função `print(_:separator:terminator:)`. Neste exemplo, o número 5 é identificado corretamente como um número primo.

> Nota: A palavra-chave `fallthrough` não verifica as condições para o _switch case_ em que ela faz com que a execução caia. A palavra-chave `fallthrough` simplesmente faz com que a execução do código se mova diretamente para as instruções dentro do próximo bloco `case` (ou _default case_ no exemplo), como no comportamento padrão da instrução `switch` do C.

#### **Guard** e os Early Returns

Uma instrução `guard`, como uma instrução `if`, executa instruções dependendo do valor booleano de uma expressão. Você usa uma instrução `guard` para exigir que uma condição seja verdadeira para que o código após a instrução `guard` seja executado. Ao contrário de uma instrução `if`, uma instrução `guard` sempre tem uma cláusula `else` — o código dentro da cláusula else é executado se a condição não for verdadeira.

``` swift
func sauda(_ nomeDaPessoa: String, de localizacao: String?) {
    print("Olá \(nomeDaPessoa)!")

    guard let localizacao = localizacao else {
        print("Espero que o clima esteja legal por aí.")
        return
    }

    print("Espero que o clima esteja legal em \(localizacao).")
}

let pessoa = "Rafael"
var localizacao: String?

sauda(pessoa, de: localizacao)
// Imprime "Olá Rafael!"
// Imprime "Espero que o clima esteja legal por aí."

let outraPessoa = "Hamilton"

sauda(outraPessoa, de: "Brasília")
// Imprime "Olá Hamilton!"
// Imprime "Espero que o clima esteja legal em Brasília"
```

Se a condição da instrução `guard` for atendida, a execução do código continua após a chave de fechamento da instrução `guard`. Quaisquer variáveis ou constantes aos quais foram atribuídos valores usando um _optional binding_ como parte da condição estão disponíveis para o restante do bloco de código em que a instrução `guard` aparece.

Se essa condição não for atendida, o código dentro da _branch_ do `else` é executado. Essa _branch_ deve transferir o controle para sair do bloco de código no qual a instrução `guard` aparece. Ele pode fazer isso com uma instrução de transferência de controle, como `return`, `break`, e outras, ou pode chamar uma função ou método terminal, como `fatalError(_:file:line:)`.

Usar uma instrução `guard` para requisitos melhora a legibilidade do seu código, em comparação com fazer a mesma verificação com uma instrução `if`. Ele permite que você escreva o código que normalmente é executado sem envolvê-lo em um bloco `else` e permite que você mantenha o código que trata de um requisito violado ao lado do requisito.
