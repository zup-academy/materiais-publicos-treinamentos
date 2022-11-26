# Manipulação de erros _(Error handling)_

> Esse material teórico foi traduzido e adaptado da fonte original disponível em https://docs.swift.org/swift-book/LanguageGuide/ErrorHandling.html

_Error handling_, ou tratamento/manipulação de erros, é o processo de resposta e recuperação de condições de erro em seu programa. O Swift fornece bom suporte para lançar, capturar, propagar e manipular erros recuperáveis em tempo de execução.

Algumas operações não têm garantia de sempre concluir a execução ou produzir a saída esperada. Opcionais são usados para representar a ausência de um valor, mas quando uma operação falha, geralmente é útil entender o que causou a falha, para que seu código possa responder de acordo.

Como exemplo, considere a tarefa de ler e processar dados de um arquivo em disco. Há várias maneiras pelas quais essa tarefa pode falhar, incluindo o arquivo não existente no caminho especificado, o arquivo sem permissões de leitura ou o arquivo não sendo codificado em um formato compatível. A distinção entre essas diferentes situações permite que um programa resolva alguns erros e comunique ao usuário quaisquer erros que não possa resolver.

> Nota: O tratamento de erros no Swift interopera com padrões de tratamento de erros que usam a classe NSError do Cocoa em Objective-C. Para obter mais informações sobre essa classe, consulte [Manipulando Cocoa Errors no Swift](https://developer.apple.com/documentation/swift/cocoa_design_patterns/handling_cocoa_errors_in_swift).

## Representando e lançando erros

No Swift, os erros são representados por valores de tipos que estão em conformidade com o protocolo `Error`. Este protocolo vazio indica que um tipo pode ser usado para tratamento de erros.

As _enums_ Swift são particularmente adequadas para modelar um grupo de condições de erro relacionadas, com valores associados que permitem a comunicação de informações adicionais sobre a natureza de um erro. Por exemplo, veja como você pode representar as condições de erro ao operar uma máquina de venda automática dentro de um jogo:

``` swift
enum MaquinaDeVendasError: Error {
     case selecaoInvalida
     case saldoInsuficiente(moedasNecessarias: Int)
     case semEstoque
}
```

Lançar um erro permite indicar que algo inesperado aconteceu e o fluxo normal de execução não pode continuar. Você usa uma instrução `throw` para lançar um erro. Por exemplo, o código a seguir gera um erro para indicar que cinco moedas adicionais são necessárias para a máquina de venda automática:

``` swift
throw MaquinaDeVendasError.saldoInsuficiente(moedasNecessarias: 5)
```

## Tratando erros

Quando um erro é lançado, alguma parte do código ao redor da invocação deve ser responsável por lidar com o erro — por exemplo, corrigindo o problema, tentando uma abordagem alternativa ou informando o usuário sobre a falha.

Existem quatro maneiras de lidar com erros no Swift. Você pode propagar o erro de uma função para o código que chama essa função, manipular o erro usando uma instrução _do-catch_, manipular o erro como um valor opcional ou afirmar que o erro não ocorrerá. Cada abordagem é descrita em uma seção abaixo.

Quando uma função gera um erro, ela altera o fluxo do seu programa, por isso é importante que você possa identificar rapidamente locais em seu código que podem gerar erros. Para identificar esses locais em seu código, escreva a palavra-chave `try` — ou uma das variações `try?` ou `try!` — antes de um trecho de código que chama uma função, método ou inicializador que pode gerar um erro. Essas palavras-chave são descritas nas seções abaixo.

> Nota: O tratamento de erros no Swift se assemelha ao tratamento de exceções em outras linguagens, com o uso das palavras-chave `try`, `catch` e `throw`. Ao contrário do tratamento de exceções em muitos idiomas, incluindo Objective-C, o tratamento de erros no Swift não envolve desempilhar a stack de chamadas, um processo que pode ser computacionalmente caro. Assim, as características de desempenho de uma instrução `throw` são comparáveis ​​às de uma instrução `return`.

### Propagando erros usando funções com `throws`

Para indicar que uma função, método ou inicializador pode lançar um erro, você escreve a palavra-chave `throws` na declaração da função após seus parâmetros. Uma função marcada com `throws` é chamada de _throwing function_, ou função de lançamento. Se a função especificar um tipo de retorno, escreva a palavra-chave `throws` antes da seta de retorno _(->)_.

``` swift
func podeLancarErrors() throws -> String

func naoPodeLancarErrors() -> String
```

Uma função de lançamento propaga erros que são lançados dentro dela para o escopo de onde é chamada.

> Nota: Somente funções de lançamento podem propagar erros. Quaisquer erros lançados dentro de uma função nonthrowing devem ser tratados dentro da função.

No exemplo abaixo, a classe `MaquinaDeVendas` tem um método `venda(item:)` que lança um `MaquinaDeVendasError` apropriado se o item solicitado não estiver disponível, estiver esgotado ou tiver um custo que exceda o valor atual depositado:

``` swift
struct Item {
    var preco: Int
    var contagem: Int
}

class MaquinaDeVendas {
    var inventario = [
        "Barra de chocolate": Item(preco: 12, contagem: 7),
        "Fichas": Item(preco: 10, contagem: 4),
        "Pretzels": Item(preco: 7, contagem: 11)
    ]
    var moedasDepositadas = 0

    func venda(item nome: String) throws {
        guard let item = inventario[nome] else {
            throw MaquinaDeVendasError.selecaoInvalida
        }

        guard item.contagem > 0 else {
            throw MaquinaDeVendasError.semEstoque
        }

        guard item.preco <= moedasDepositadas else {
            throw MaquinaDeVendasError.saldoInsuficiente(moedasNecessarias: item.preco - moedasDepositadas)
        }

        moedasDepositadas -= item.preco

        var novoItem = item
        novoItem.contagem -= 1
        inventario[nome] = novoItem

        print("Preparando \(nome)")
    }
}
```

A implementação do método `venda(item:)` usa instruções `guard` para sair do método antecipadamente e gerar erros apropriados se algum dos requisitos para comprar um lanche não for atendido. Como uma instrução `throw` transfere imediatamente o controle do programa, um item será vendido apenas se todos esses requisitos forem atendidos.

Como o método `venda(item:)` propaga todos os erros que lança, qualquer código que chama esse método deve manipular os erros — usando uma instrução _do-catch_, `try?` ou `try!` — ou continuar a propagá-los. Por exemplo, o método `compraSnackFavorito(pessoa:maquinaDeVendas:)` no exemplo abaixo também é uma função de lançamento, e quaisquer erros que o método `venda(item:)` lançar serão propagados até o ponto em que a função `compraSnackFavorito(pessoa:maquinaDeVendas:)` é chamado.

``` swift
let snacksFavoritos = [
    "Alberto": "Acarajé",
    "Rafael": "Pastel de Nata",
    "Matheus": "Açaí",
]

func compraSnackFavorito(pessoa: String, maquinaDeVendas: MaquinaDeVendas) throws {
    let snack = snacksFavoritos[pessoa] ?? "Chocolate"
    try maquinaDeVendas.venda(item: snack)
}
```

Neste exemplo, a função `compraSnackFavorito(pessoa: maquinaDeVendas:)` procura o snack favorito de uma determinada pessoa e tenta comprá-lo chamando o método `venda(item:)`. Como o método `venda(item:)` pode gerar um erro, ele é chamado com a palavra-chave `try` antes dele.

_Throwing initializers_, ou inicializadores de lançamento, podem propagar erros da mesma forma que funções de lançamento. Por exemplo, o inicializador para a estrutura `SnackComprado` na listagem abaixo chama uma função de lançamento como parte do processo de inicialização e lida com quaisquer erros que encontra propagando-os para seu chamador.

``` swift
struct SnackComprado {
    let nome: String
    init(nome: String, maquinaDeVendas: maquinaDeVendas) throws {
        try maquinaDeVendas.venda(item: nome)
        self.nome = nome
    }
}
```

### Manipulando erros usando _do-catch_

Você usa uma instrução _do-catch_ para lidar com erros executando um bloco de código. Se um erro for lançado pelo código na cláusula `do`, ele será comparado com as cláusulas `catch` para determinar qual delas pode lidar com o erro.

Aqui está a forma geral de uma instrução _do-catch_:

<p align="center">
<img alt="Imagem com o exemplo da estrutura base de código utilizando cláusulas do-catch" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-error-handling-exemplo-do-catch.png?raw=true" width="70%" />
</p>

Você escreve um padrão após `catch` para indicar quais erros essa cláusula pode manipular. Se uma cláusula `catch` não tiver um padrão, a cláusula corresponderá a qualquer erro e vinculará o erro a uma constante local chamada `error`. Para obter mais informações sobre correspondência de padrões, consulte [Padrões](https://docs.swift.org/swift-book/ReferenceManual/Patterns.html).

Por exemplo, o código a seguir corresponde a todos os três casos da enumeração MaquinaDeVendasError.

``` swift
var maquinaDeVendas = MaquinaDeVendas()
maquinaDeVendas.moedasDepositadas = 8

do {
    try compraSnackFavorito(pessoa: "Alberto", maquinaDeVendas: maquinaDeVendas)
    print("Sucesso!")

} catch MaquinaDeVendasError.selecaoInvalida {
    print("Seleção inválida.")

} catch MaquinaDeVendasError.semEstoque {
    print("Não tem mais deste item.")

} catch MaquinaDeVendasError.saldoInsuficiente(let moedasNecessarias) {
    print("Saldo insuficiente. Por favor insira \(moedasNecessarias) moedas.")
    
} catch {
    print("Um erro ocorreu: \(error).")
}

// Imprime "Saldo insuficiente. Por favor insira 2 moedas."
```

No exemplo acima, a função `compraSnackFavorito(pessoa:maquinaDeVendas:)` é chamada em uma expressão `try`, porque pode lançar um erro. Se um erro for lançado, a execução será transferida imediatamente para as cláusulas `catch`, que decidem se permitem que a propagação continue. Se nenhum padrão for correspondido, o erro será detectado pela cláusula `catch` final e vinculado a uma constante de erro local. Se nenhum erro for lançado, as instruções restantes na instrução `do` serão executadas.

As cláusulas `catch` não precisam lidar com todos os erros possíveis que o código na cláusula do pode gerar. Se nenhuma das cláusulas `catch` manipular o erro, o erro se propagará para o escopo ao redor. No entanto, o erro propagado deve ser tratado por algum escopo. Em uma função sem lançamento, uma instrução _do-catch_ deve manipular o erro. Em uma função de lançamento, uma instrução _do-catch_ ou o chamador deve tratar o erro. Se o erro se propagar para o escopo de nível superior sem ser tratado, você receberá um erro de tempo de execução encerrando a aplicação.

Por exemplo, o exemplo abaixo pode ser escrito para que qualquer erro que não seja um `MaquinaDeVendasError` seja detectado pela função de chamada:

``` swift
func obtem(item: String) throws {
    do {
        try maquinaDeVendas.venda(item: item)
    } catch is MaquinaDeVendasError {
        print("Não é possível obter este item pela máquina.")
    }
}

do {
    try obtem(item: "Chips de Banana Salgada")
} catch {
    print("Erro não relacionado à MaquinaDeVendasError: \(error)")
}
// Prints "Não é possível obter este item pela máquina."
```

Na função `obtem(item:)`, se `venda(item:)` lançar um erro que é um dos casos da _enum_ `MaquinaDeVendasError`, `obtem(item:)` tratará do erro imprimindo uma mensagem. Caso contrário, `obtem(item:)` propaga o erro para seu ponto de chamada. O erro é então detectado pela cláusula `catch` geral.

Outra forma de capturar vários erros relacionados é listá-los após `catch`, separados por vírgulas. Por exemplo:

``` swift
func pega(item: String) throws {
    do {
        try maquinaDeVendas.venda(item: item)
    } catch MaquinaDeVendasError.selecaoInvalida, MaquinaDeVendasError.saldoInsuficiente, MaquinaDeVendasError.semEstoque {
        print("Seleção inválida, sem estoque, ou dinheiro insuficiente.")
    }
}
```

A função `pega(item:)` lista os erros da máquina de vendas a serem detectados e seu texto de erro corresponde aos itens dessa lista. Se algum dos três erros listados for lançado, essa cláusula `catch` os tratará imprimindo uma mensagem. Quaisquer outros erros são propagados para o escopo de chamada, incluindo quaisquer erros de máquina de venda automática que possam ser adicionados posteriormente.

### Convertendo erros em valores opcionais

Você usa `try?` para lidar com um erro convertendo-o em um valor opcional. Se um erro for lançado durante a avaliação da expressão `try?`, o valor da expressão é nulo. Por exemplo, no código a seguir, `x` e `y` têm o mesmo valor e comportamento:

``` swift
func algumaThrowingFunction() throws -> Int {
    // ...
}
let x = try? algumaThrowingFunction()

let y: Int?
do {
    y = try algumaThrowingFunction()
} catch {
    y = nil
}
```

Se `algumaThrowingFunction()` lançar um erro, o valor de `x` e `y` será nulo. Caso contrário, o valor de `x` e `y` é o valor que a função retornou. Observe que `x` e `y` são opcionais de qualquer tipo que `algumaThrowingFunction()` retorne. Aqui a função retorna um inteiro, então `x` e `y` são inteiros opcionais.

Usando `try?` você pode escrever um código de tratamento de erros conciso quando quiser lidar com todos os erros da mesma maneira. Por exemplo, o código a seguir usa várias abordagens para buscar dados ou retorna `nil` se todas as abordagens falharem.

``` swift
func carregaDados() -> Data? {
    if let dados = try? carregaDadosDoDisco() { return dados }
    if let dados = try? carregaDadosDoServidor() { return dados }
    return nil
}
```

### Desativando a propagação de Erros

Às vezes, você sabe que uma função ou método de lançamento não irá, de fato, lançar um erro em tempo de execução. Nessas ocasiões, você pode escrever `try!` antes da expressão para desabilitar a propagação de erro e agrupar a chamada em uma declaração de tempo de execução de que nenhum erro será gerado. Se um erro realmente for lançado, você receberá um erro de tempo de execução.

Por exemplo, o código a seguir usa uma função `carregaImagem(em:), que carrega o recurso de imagem em um determinado caminho ou lança um erro se a imagem não puder ser carregada. Nesse caso, como a imagem é local, empacotada junto com o aplicativo, nenhum erro será lançado em tempo de execução, portanto, é apropriado desabilitar a propagação de erros.

``` swift
let foto = try! carregaImagem(em: "./Resources/Rafael Rollo.jpg")
```

## Especificando ações de cleanup

Você usa uma instrução `defer` para executar um conjunto de instruções antes que a execução do código saia do bloco de código atual. Essa instrução permite que você faça qualquer limpeza necessária que deve ser executada, independentemente de como a execução sai do bloco de código atual — se ela sai porque um erro foi lançado ou por causa de uma instrução como `return` ou `break`. Por exemplo, você pode usar uma instrução `defer` para garantir que os descritores de arquivo sejam fechados e a memória alocada manualmente seja liberada.

Uma instrução `defer` adia a execução até que o escopo atual seja encerrado. Essa instrução consiste na palavra-chave `defer` e nas instruções a serem executadas posteriormente. As instruções adiadas não podem conter nenhum código que transfira o controle das instruções, como uma quebra ou uma instrução de retorno, ou lançando um erro. As ações adiadas são executadas na ordem inversa da ordem em que foram escritas em seu código-fonte. Ou seja, o código na primeira instrução `defer` é executado por último, o código na segunda instrução `defer` é executado da penúltima e assim por diante. A última instrução `defer` na ordem do código-fonte é executada primeiro.

``` swift
func processaArquivo(nomeDoArquivo: String) throws {
    if exists(nomeDoArquivo) {
        let arquivo = open(nomeDoArquivo)
        
        defer {
            close(arquivo)
        }
        
        while let ln = try arquivo.readline() {
            // Trabalha com o arquivo.
        }
        
        // close(arquivo) é chamado aqui, no final deste escopo.
    }
}
```

O exemplo acima usa uma instrução `defer` para garantir que a função `open(_:)` tenha uma chamada correspondente para `close(_:)`.

> Nota: Você pode usar uma instrução `defer` mesmo quando nenhum código de tratamento de erro estiver envolvido.
