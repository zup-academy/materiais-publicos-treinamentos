# Constantes e Variáveis na Swift

>Esse material teórico foi traduzido e editado da fonte original disponível em https://docs.swift.org/swift-book/LanguageGuide/TheBasics.html.

Constantes e variáveis associam um nome - como `numeroMaximoDeTentativas` ou `mensagemDeBoasVindas`) a um valor de um tipo específico (como o número `10` ou a string `"Hello"`). O valor de uma constante não pode ser alterado depois de definido, enquanto uma variável pode ser definida para um valor diferente no futuro.

## Declaração de constantes e variáveis

Constantes e variáveis devem ser declaradas antes de serem usadas. Você declara constantes com a palavra-chave `let` e variáveis com a palavra-chave `var`. Aqui está um exemplo de como constantes e variáveis podem ser usadas para rastrear o número de tentativas de login que um usuário fez:

``` swift
let numeroMaximoDeTentativasDeLogin = 10
var tentativaAtual = 0
```

Este código pode ser lido como:

“Declare uma nova constante chamada `numeroMaximoDeTentativasDeLogin` e dê a ela um valor de 10. Em seguida, declare uma nova variável chamada `tentativaAtual` e dê a ela um valor inicial de 0.”

Neste exemplo, o número máximo de tentativas de login permitidas é declarado como uma constante, porque o valor máximo nunca muda. O contador de tentativas de login atual é declarado como uma variável, porque esse valor deve ser incrementado após cada tentativa de login com falha.

Swift permite declarar um conjunto de constantes ou de variáveis em uma única linha, separadas por vírgulas:

``` swift
var x = 0.0, y = 0.0, z = 0.0
```

> Nota: Se um valor armazenado em seu código não mudar, sempre o declare como uma constante com a palavra-chave `let`. Use variáveis apenas para armazenar valores que precisam ser alterados.

## Anotações de tipo (Type Annotations)

Você pode fornecer uma anotação de tipo _(type annotation)_ ao declarar uma constante ou variável, para ser claro sobre o tipo de valores que a constante ou variável pode armazenar. Escreva uma anotação de tipo colocando dois pontos após o nome da constante ou variável, seguido por um espaço, seguido pelo nome do tipo a ser usado.

Este exemplo fornece uma anotação de tipo para uma variável chamada `mensagemDeBoasVindas`, para indicar que a variável pode armazenar valores de String:

``` swift
var mensagemDeBoasVindas: String
```

Os dois pontos na declaração significam “... do tipo ...”, portanto, o código acima pode ser lido como:

“Declare uma variável chamada `mensagemDeBoasVindas` que é do tipo String.”

A frase “do tipo String” significa “pode armazenar qualquer valor de String”. A variável `mensagemDeBoasVindas` agora pode ser definida como qualquer valor de string sem erros:

``` swift
mensagemDeBoasVindas = "Olá"
```

É possível definir múltiplas variáveis relacionadas do mesmo tipo em uma única linha, separadas por vírgulas, com uma única anotação de tipo após o nome da variável final:

``` swift
var red, green, blue: Double
```

>Nota: Na prática, você não precisa escrever anotações de tipo para as constantes ou variáveis. Se você fornecer um valor inicial para uma constante ou variável no ponto em que está definida, o Swift quase sempre pode inferir o tipo a ser usado para essa constante ou variável. No exemplo `mensagemDeBoasVindas` acima, nenhum valor inicial é fornecido e, portanto, o tipo da variável `mensagemDeBoasVindas` é especificado com uma anotação de tipo em vez de ser inferido de um valor inicial. Na dúvida sobre qual abordagem usar, siga sempre o especificado pelos padrões de escrita de código acordado com sua equipe de projeto.

## Nomeando Constantes and Variáveis

Nomes de constantes e variáveis podem conter quase qualquer caractere, incluindo caracteres Unicode:

``` swift
let π = 3.14159
let 你好 = "你好世界"
let 🐶🐮 = "dogcow"
```

Os nomes de constantes e variáveis não podem conter caracteres de espaço em branco, símbolos matemáticos, setas, valores escalares Unicode de uso privado ou caracteres de desenho de linhas e caixas. Nem podem começar com um número, embora os números possam ser incluídos em outras partes do nome.

Depois de declarar uma constante ou variável de um determinado tipo, você não pode declará-la novamente com o mesmo nome ou alterá-la para armazenar valores de um tipo diferente. Nem você pode transformar uma constante em uma variável ou uma variável em uma constante.

Você pode alterar o valor de uma variável existente para outro valor de um tipo compatível. Neste exemplo, o valor de `mensagemDeBoasVindas` foi alterado de "Olá!" para "Hello!":

``` swift
var mensagemDeBoasVindas = "Olá!"
mensagemDeBoasVindas = "Hello!"
// mensagemDeBoasVindas agora é "Hello!"
```

Ao contrário de uma variável, o valor de uma constante não pode ser alterado depois de definido. A tentativa de fazer isso é relatada como um erro quando seu código é compilado:

``` swift
let nomeDaLinguagem = "Swift"
nomeDaLinguagem = "Swift++" // Esse é um erro em tempo de compilação: nomeDaLinguagem cannot be changed.
```

## Imprimindo Constantes e Variáveis

Você pode imprimir o valor atual de uma constante ou variável com a função `print(_:separator:terminator:)`:

``` swift
print(mensagemDeBoasVindas)
// Imprime "Hello!"
```

A função `print(_:separator:terminator:)` é uma função global que imprime um ou mais valores em uma saída apropriada. No Xcode, por exemplo, a função `print(_:separator:terminator:)` imprime sua saída no painel “console” do Xcode. Os parâmetros separador e terminador têm valores padrão, portanto, você pode omiti-los ao chamar essa função. Por padrão, a função encerra a linha que imprime adicionando uma quebra de linha. Para imprimir um valor sem uma quebra de linha depois dele, passe uma string vazia como o terminador — por exemplo, `print(algumValor, terminator: "")`.

O Swift usa a interpolação de _strings_ para incluir o nome de uma constante ou variável como um espaço reservado em uma _string_ mais longa e para solicitar que o Swift o substitua pelo valor atual dessa constante ou variável. Coloque o nome entre parênteses e escape com uma barra invertida antes do parêntese de abertura:

``` swift
print("O valor atual de mensagemDeBoasVindas é \(mensagemDeBoasVindas)")
// Imprime "O valor atual de mensagemDeBoasVindas é Hello!"
```