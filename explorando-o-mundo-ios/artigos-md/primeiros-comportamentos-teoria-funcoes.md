# Funções na linguagem Swift

>Esse material teórico foi traduzido e editado da fonte original disponível em https://docs.swift.org/swift-book/LanguageGuide/Functions.html.

As funções são pedaços de código independentes que executam uma tarefa específica. Você dá a uma função um nome que identifica o que ela faz, e esse nome é usado para “chamar” a função para executar sua tarefa quando necessário.

A sintaxe de função unificada do Swift é flexível o suficiente para expressar qualquer coisa, desde uma função simples no estilo C sem nomes de parâmetros até um método complexo no estilo Objective-C com nomes e rótulos de argumento para cada parâmetro. Os parâmetros podem também fornecer valores padrão para simplificar as chamadas de função.

Cada função em Swift tem um tipo, que consiste nos tipos de parâmetros da função e no tipo de retorno. Você pode usar esse tipo como qualquer outro tipo no Swift.

## Definindo e Chamando Funções

Ao definir uma função, você pode definir opcionalmente um ou mais valores com nome e tipo que a função usa como entrada, conhecidos como parâmetros. Você também pode definir opcionalmente um tipo de valor que a função retornará como saída quando terminar, conhecido como tipo de retorno.

Cada função tem um nome de função, que descreve a tarefa que a função executa. Para usar uma função, você “chama” essa função com seu nome e passa valores de entrada (conhecidos como argumentos) que correspondem aos tipos de parâmetros da função. Os argumentos de uma função devem sempre ser fornecidos na mesma ordem que a lista de parâmetros da função.

A função no exemplo abaixo é chamada `greet(person:)`, porque é isso que ela faz — pega o nome de uma pessoa como entrada e retorna uma saudação para essa pessoa. Para fazer isso, você define um parâmetro de entrada - um valor String chamado pessoa - e um tipo de retorno String, que conterá uma saudação para essa pessoa:

``` swift
func greet(person: String) -> String {
    let greeting = "Hello, " + person + "!"
    return greeting
}
```

Todas essas informações são agrupadas na definição da função, que tem como prefixo a palavra-chave `func`. Você indica o tipo de retorno da função com a seta de retorno `->` (um hífen seguido por um colchete angular reto), que é seguido pelo nome do tipo a ser retornado.

A definição descreve o que a função faz, o que ela espera receber e o que ela retorna quando termina. A definição facilita que a função seja chamada de forma inequívoca de outro lugar em seu código:

``` swift
print(greet(person: "Rafael"))
// Imprime "Hello, Rafael!"

print(greet(person: "Alberto"))
// Imprime "Hello, Alberto!"
```

Você chama a função `greet(person:)` passando um valor `String` após o rótulo do argumento `person`, como `greet(person: "Alberto")`. Como a função retorna um valor `String`, `greet(person:)` pode ser agrupado em uma chamada para a função `print(_:separator:terminator:)` para imprimir essa _string_ e ver seu valor de retorno, conforme mostrado acima.

>Nota: A função `print(_:separator:terminator:)` não possui um rótulo para seu primeiro argumento, e seus outros argumentos são opcionais porque possuem um valor padrão. Essas variações na sintaxe da função são discutidas mais a frente.

O corpo da função `greet(person:)` começa definindo uma nova constante `String` chamada saudação (_greeting_) e configurando-a como uma mensagem de saudação simples. Essa saudação é então passada de volta para fora da função usando a palavra-chave `return`. Na linha de código que diz `return greeting`, a função termina sua execução e retorna o valor atual de saudação.

Você pode chamar a função `greet(person:)` várias vezes com diferentes valores de entrada. O exemplo acima mostra o que acontece se for chamado com um valor de entrada de `"Alberto"` e um valor de entrada de `"Rafael"`. A função retorna uma saudação personalizada em cada caso.

Para tornar o corpo desta função mais curto, você pode combinar a criação da mensagem e a instrução de retorno em uma linha:

``` swift
func greet(person: String) -> String {
    return "Hello, " + person + "!"
}

print(greet(person: "Leonardo"))
// Imprime "Hello, Leonardo!"
```

## Parâmetros de Função e Valores de Retorno

Parâmetros de função e valores de retorno são extremamente flexíveis no Swift. Você pode definir qualquer coisa, desde uma função utilitária simples com um único parâmetro sem nome até uma função complexa com nomes de parâmetros expressivos e diferentes opções de parâmetros.

### Funções sem Parâmetros

As funções não obrigam a definição de parâmetros de entrada. Por exemplo, aqui está uma função sem parâmetros de entrada, que sempre retorna a mesma mensagem String sempre que é chamada:

``` swift
func sayHello() -> String {
    return "hello, world"
}

print(sayHello())
// Imprime "hello, world"
```

A definição da função ainda precisa de parênteses após o nome da função, mesmo que não receba nenhum parâmetro. O nome da função também é seguido por um par de parênteses vazio quando a função é chamada.

### Funções com Vários Parâmetros

As funções podem ter vários parâmetros de entrada, que são escritos entre parênteses da função, separados por vírgulas.

Esta função pega o nome de uma pessoa e se ela já foi saudada como entrada e retorna uma saudação apropriada para essa pessoa:

``` swift
func greet(person: String, withTitle desiredTitle: String) -> String {
    return "Hello, " + desiredTitle + " " + person + "!"
}

// possível leitura: saúda a pessoa "Leonardo" com o titulo "Sir"
print(greet(person: "Leonardo", withTitle: "Sir"))

// Imprime "Hello, Sir Leonardo!"
```

Você chama a função `greet(person:withTitle:)` passando a ela um valor de argumento `String` rotulado `person` e outro valor de argumento String rotulado `withTitle` (associado à `desiredTitle`) entre parênteses, separados por vírgulas. Observe que esta função é diferente da função `greet(person:)` mostrada em uma seção anterior. Embora ambas as funções tenham nomes que começam com _greet_, a função `greet(person:withTitle:)` recebe dois argumentos, enquanto a função `greet(person:)` recebe apenas um.

>Nota: O código acima dá uma mostra do poder de expressividade da linguagem com os possíveis rótulos de argumento (_argument labels_) associados aos nomes de parâmetros. Mais a frente falaremos especificamente sobre esse importante recurso.

### Funções sem Valores de Retorno

As funções também não obrigam a definição de um tipo de retorno. Aqui está uma versão da função `greet(person:)`, que imprime seu próprio valor String em vez de retorná-lo:

``` swift
func greet(person: String) {
    print("Hello, \(person)!")
}

greet(person: "Edu")
// Imprime "Hello, Edu!"
```

Por não precisar retornar um valor, a definição da função não inclui a seta de retorno (`->`) ou um tipo de retorno.

>Nota: Estritamente falando, esta versão da função `greet(person:)` ainda retorna um valor, mesmo que nenhum valor de retorno seja definido. Funções sem um tipo de retorno definido retornam um valor especial do tipo `Void`. Esta é simplesmente uma tupla vazia, que é escrita como `()`.

O valor de retorno de uma função pode ser ignorado quando é chamado:

``` swift
func printaEConta(string: String) -> Int {
    print(string)
    return string.count
}

func printaSemContar(string: String) {
    let _ = printaEConta(string: string)
}

printaEConta(string: "hello, world")
// Imprime "hello, world" e retorna o valor 12

printaSemContar(string: "hello, world")
// Imprime "hello, world" mas não retorna qualquer valor
```

A primeira função, `printaEConta(string:)`, imprime uma _string_ e retorna sua contagem de caracteres como um inteiro. A segunda função, `printaSemContar(string:)`, chama a primeira função, mas ignora seu valor de retorno. Quando a segunda função é chamada, a mensagem ainda é impressa pela primeira função, mas o valor retornado não é usado.

>Nota: Os valores de retorno podem ser ignorados, mas uma função que diz que retornará um valor deve sempre fazê-lo. Uma função com um tipo de retorno definido não pode permitir que o controle saia da parte inferior da função sem retornar um valor, e tentar fazer isso resultará em um erro de tempo de compilação.

### Tipos de Retorno de Opcionais

Se o tipo de dado a ser retornado por uma função tem o potencial de não ter “nenhum valor”, você pode usar um tipo de retorno de opcional para refletir o fato de que o retorno pode ser nulo. Você escreve um tipo de retorno opcional colocando um ponto de interrogação após o tipo de dado, como `Int?` ou `String?`.

``` swift
func conta(string: String) -> Int? {
    if string.isEmpty {
        return nil
    }

    return string.count
}

conta(string: "hello, world")
// Retorna 12

conta(string: "")
//Retorn nil
```

### Funções com  Retorno Implícito

Se todo o corpo da função for uma única expressão, a função retornará implicitamente o resultado dessa expressão. Por exemplo, ambas as funções abaixo têm o mesmo comportamento:

``` swift
// 1
func greeting(for person: String) -> String {
    "Hello, " + person + "!"
}
print(greeting(for: "Alberto"))
// Imprime "Hello, Alberto!"

// 2 
func greeting(for person: String) -> String {
    return "Hello, " + person + "!"
}
print(greeting(for: "Edu"))
// Imprime "Hello, Edu!"
```

A definição completa da função `greeting(for:)` é a mensagem de saudação que ela retorna, o que significa que ela pode usar essa forma mais curta. O exemplo 2 da função `greeting(for:)` retorna a mesma mensagem de saudação, usando a palavra-chave `return` como uma função mais longa. Qualquer função que você escreve como apenas uma linha de retorno pode omitir a cláusula `return`.

## Rótulos de Argumentos de Função e Nomes de Parâmetros

Cada parâmetro de função tem um rótulo de argumento (_argument label_) e um nome de parâmetro propriamente. O rótulo do argumento é usado ao chamar a função; cada argumento é escrito na chamada de função com seu rótulo de argumento antes dele. O nome do parâmetro é usado na implementação da função. Quando não definido um rótulo de argumento, por padrão, os parâmetros usam seu nome de parâmetro como rótulo.

``` swift
func umaFuncao(primeiroNomeDeParametro: Int, segundoNomeDeParametro: Int) {
    // No corpo da função, primeiroNomeDeParametro and segundoNomeDeParametro
    // referem-se aos valores de argumento para o primeiro e segundo parâmetros.
}

umaFuncao(primeiroNomeDeParametro: 1, primeiroNomeDeParametro: 2)
```

Todos os parâmetros devem ter nomes exclusivos. Embora seja possível que vários parâmetros tenham o mesmo rótulo de argumento, rótulos de argumento exclusivos ajudam a tornar seu código mais legível.

### Especificando Rótulos de Argumento

Você escreve um rótulo de argumento antes do nome do parâmetro, separado por um espaço:

``` swift
func umaFuncao(rotuloDeArgumento nomeDoParamentro: Int) {
    // No corpo da função, nomeDoParamentro se refere ao valor do argumento
    // para o parâmentro.
}
```

Novamente, aqui está uma variação da função `greet(person:)` que recebe o nome e o título com o qual a pessoa deseja  ser tratada e retorna uma saudação:

``` swift
func greet(person: String, withTitle desiredTitle: String) -> String {
    return "Hello, \(desiredTitle) \(person)!"
}

// possível leitura: saúda a pessoa "Leonardo" com o titulo "Sir"
print(greet(person: "Leonardo", withTitle: "Sir"))

// Imprime "Hello, Sir Leonardo!"
```

O uso de rótulos de argumento pode permitir que uma função seja chamada de maneira expressiva, semelhante a uma frase, enquanto ainda fornece um corpo de função que é legível e claro na intenção.

### Omitindo Rótulos de Argumentos

Se você não quiser um rótulo de argumento para um parâmetro, escreva um sublinhado (`_`) em vez de um rótulo de argumento explícito para esse parâmetro.

``` swift 
func umaFuncao(_ primeiroNomeDeParametro: Int, segundoNomeDeParametro: Int) {
    // No corpo da função, primeiroNomeDeParametro and segundoNomeDeParametro
    // referem-se aos valores de argumento para o primeiro e segundo parâmetros.
}

umaFuncao(1, segundoNomeDeParametro: 2)
```

Se um parâmetro tiver um rótulo de argumento, o argumento deverá ser rotulado quando você chamar a função.

### Valores Padrão para Parâmetros

Você pode definir um valor padrão para qualquer parâmetro em uma função atribuindo um valor ao parâmetro após o tipo desse parâmetro. Se um valor padrão for definido, você poderá omitir esse parâmetro ao chamar a função.

``` swift
func umaFuncao(parametroComum: Int, parametroComValorPadrao: Int = 12) {
    // Se você omite o segundo argumento ao chamar essa função
    // o valor de parametroComValorPadrao é definido para 12 dentro do corpo da função
}

umaFuncao(parametroComum: 3, parametroComValorPadrao: 6) // parametroComValorPadrao é 6

umaFuncao(parametroComum: 4) // parametroComValorPadrao é 12
```

Coloque os parâmetros que não possuem valores padrão no início da lista de parâmetros de uma função, antes dos parâmetros que possuem valores padrão. Parâmetros que não têm valores padrão geralmente são mais importantes para o significado da função - escrevê-los primeiro facilita o reconhecimento de que a mesma função está sendo chamada, independentemente de quaisquer parâmetros padrão serem omitidos.
