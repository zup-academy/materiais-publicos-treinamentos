# Classes na linguagem Swift

>Esse material teórico foi traduzido e editado da fonte original disponível em https://docs.swift.org/swift-book/LanguageGuide/ClassesAndStructures.html.

Classes são construções flexíveis de propósito geral que se tornam os blocos de construção do código do seu programa. Você define propriedades e métodos para adicionar funcionalidade às suas classes usando a mesma sintaxe usada para definir constantes, variáveis e funções.

Ao contrário de outras linguagens de programação, o Swift não exige que você crie interfaces e arquivos de implementação separados para classes personalizadas. No Swift, você define uma classe em um único arquivo, e a interface de comunicação externa para essa classe é disponibilizada automaticamente para uso de outro código.

## Características de Classes Swift

As classes no Swift tem uma série de recursos. Com elas você pode:

- Definir propriedades para armazenar valores
- Definir métodos para fornecer funcionalidade
- Definir subscritos para fornecer acesso a seus valores usando a sintaxe de subscritos
- Definir inicializadores para configurar seu estado inicial
- Ser estendido para expandir sua funcionalidade além de uma implementação padrão
- Estar em conformidade com Protocolos para fornecer a funcionalidade padrão de um determinado tipo

Algumas funcionalidades são exclusivas às classes, como: 

- Escrever desinicializadores para que uma instância de uma classe libere quaisquer recursos atribuídos
- Estender uma classe para herdar suas características
- Usar _type casting_ como forma de verificar e interpretar o tipo de uma instância de classe em tempo de execução
- Ter mais de uma referência a uma instância de classe

Esses recursos exclusivos adicionais que as classes suportam têm o custo de maior complexidade. Como orientação geral, é preferível utilizar estruturas mais simples como Enumerações (_Enums_) e Estruturas (_Structs_) porque elas tendem a tornar o raciocínio sobre o que se quer modelar mais fácil. Use classes quando for apropriado ou necessário. Na prática, isso significa que a maioria dos tipos de dados personalizados que você define para o domínio do problema da sua aplicação serão estruturas e enumerações. Nas bordas da aplicação, onde em geral se lida com a integração com frameworks bem definidos - como por exemplo o próprio UIKit - tende-se a trabalhar com classes, pois nestes módulos é fundamental compartilhar de uma série de implementações já prontas para problemas inerentes de alguma infraestrutura.

### Definição de Sintaxe

Classes no Swift têm uma definição de sintaxe semelhante a outras linguagens conhecidas. Você introduz classes com a palavra-chave `class`, com sua definição dentro de um par de chaves:

``` swift
class AlgumaClasse {
    // a definição da classe vai aqui
}
```

> Nota: Sempre que você define uma nova classe (assim como uma enum ou struct), você define um novo tipo Swift. Dê aos tipos nomes seguindo o padrão `UpperCamelCase` (como `AlgumaClasse`) para corresponder à capitalização dos tipos padrão do Swift (como `String`, `Int` e `Bool`). Dê nomes de propriedades e métodos `lowerCamelCase` (como `frameRate` e `incrementCount`) para diferenciá-los dos nomes de tipo.

Aqui está um exemplo de uma definição de classe:

``` swift
class Resolucao {
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

O exemplo acima define uma nova classe chamada `Resolucao`, para descrever uma resolução de exibição baseada em pixels. Essa estrutura tem duas propriedades armazenadas chamadas largura e altura. Propriedades armazenadas são constantes ou variáveis que são agrupadas e armazenadas como parte da classe. Essas duas propriedades são inferidas como do tipo `Int`, definindo-as como um valor inteiro inicial de `0`.

O exemplo acima também define uma nova classe chamada `ModoDeVideo`, para descrever um modo específico para exibição de vídeo. Esta classe tem quatro propriedades de variáveis armazenadas. A primeira, `resolucao`, é inicializada com uma nova instância da classe `Resolucao`, que infere seu tipo de propriedade à `Resolucao`. Para as outras três propriedades, novas instâncias de `ModoDeVideo` serão inicializadas com uma configuração `entrelacada` de false (significando “vídeo não entrelaçado”), uma taxa de atualização de quadros `0.0` e um valor opcional `String` chamado `nome`. A propriedade `nome` recebe automaticamente um valor padrão de `nil`, ou “sem valor de nome”, porque é de um tipo opcional.

### Instâncias de classe

A definição das classes `Resolucao` e `ModoDeVideo` descrevem apenas como devem se parecer Resoluções e Modos de Vídeo, modelando o conceito do mundo real. Eles próprios não descrevem uma resolução específica ou modo de vídeo. Para fazer isso, você precisa criar instâncias das classes.

A sintaxe para criar instâncias também é muito semelhante a uma série de outras linguagens:

``` swift
let algumaResolucao = Resolucao()
let algumModoDeVideo = ModoDeVideo()
```

Classes usam a sintaxe do inicializador para novas instâncias. A forma mais simples de sintaxe do inicializador usa o nome do tipo da classe seguido por parênteses vazios, como `Resolucao()` ou `ModoDeVideo()`. Isso cria uma nova instância da classe, com todas as propriedades inicializadas com seus valores padrão.

### Acessando Propriedades

Você pode acessar as propriedades de uma instância usando a sintaxe de ponto (_dot syntax_). Na sintaxe de ponto, você escreve o nome da propriedade imediatamente após o nome da instância, separado por um ponto (.), sem espaços:

``` swift
print("A largura desta resolução é \(algumaResolucao.largura)")
// Imprime: "A largura desta resolução é 0"
```

Neste exemplo, `algumaResolucao.largura` refere-se à propriedade `largura` de `algumaResolucao` e retorna seu valor inicial padrão de 0.

Você pode detalhar as subpropriedades, como a propriedade de `largura` na propriedade de `resolucao` de um `ModeDeVideo`:

``` swift
print("A largura deste modo de vídeo é \(algumaModoDeVideo.resolucao.largura)")
// Imprime "A largura deste modo de vídeo é 0"
```

Você também pode usar a sintaxe de ponto para atribuir um novo valor a uma propriedade variável:

``` swift
algumModoDeVideo.resolucao.largura = 1280
print("A largura deste modo de vídeo agora é \(algumModoDeVideo.resolucao.largura)")
// Imprime "A largura deste modo de vídeo agora é 1280"
```

## Sobre Propriedades 

As propriedades associam valores a uma classe específica. As propriedades armazenadas (_stored properties_) armazenam valores constantes e variáveis como parte de uma instância.

### Propriedades Armazenadas (_Stored Properties_)

Em sua forma mais simples, uma propriedade armazenada é uma constante ou variável que é armazenada como parte de uma instância de uma classe específica. As propriedades armazenadas podem ser propriedades armazenadas de variáveis (introduzidas pela palavra-chave `var`) ou propriedades armazenadas constantes (introduzidas pela palavra-chave `let`).

Você pode fornecer um valor padrão para uma propriedade armazenada como parte de sua definição, como visto no exemplo em Definição de Sintaxe. Você também pode definir e modificar o valor inicial de uma propriedade armazenada durante a inicialização. Isso é verdade mesmo para propriedades armazenadas de constantes.

O exemplo abaixo define uma classe chamada `IntervaloDeComprimentoFixo`, que descreve um intervalo de inteiros cujo comprimento do intervalo não pode ser alterado após sua criação:

``` swift
class IntervaloDeComprimentoFixo {
    var valorInicial: Int
    let comprimento: Int

    init(valorInicial: Int, comprimento: Int) {
        self.valorInicial = valorInicial
        self.comprimento = comprimento
    }
}

var intervaloDeTresItens = IntervaloDeComprimentoFixo(valorInicial: 0, comprimento: 3)
// o intervalo representa os valores inteiros 0, 1 e 2

intervaloDeTresItens.valorInicial = 6
// o intervalo agora representa os valores inteiros 6, 7 e 8
```

As instâncias de `IntervaloDeComprimentoFixo` têm uma propriedade armazenada variável chamada `valorInicial` e uma propriedade armazenada constante chamada `comprimento`. No exemplo acima, o comprimento é inicializado quando o novo intervalo é criado e não pode ser alterado depois, porque é uma propriedade constante.

>Nota: O exemplo acima conta com um inicializador (`init`) para prover a lógica executada no momento da construção de novas instâncias para esta classe. Como é possível perceber a sintaxe também é bastante simples e clara, se assemelhando de qualquer função. De fato, muitos recursos como vistos em _Funções na linguagem Swift_ são aplicáveis aos inicializadores, como veremos mais a frente.

## Sobre Métodos

Métodos são funções associadas a um tipo específico. Classes, estruturas e enumerações podem definir métodos de instância, que encapsulam tarefas e funcionalidades específicas para trabalhar com uma instância de um determinado tipo.

### Métodos de Instância

Métodos de instância são funções que pertencem a instâncias de uma determinada classe, estrutura ou enumeração. Eles oferecem suporte à funcionalidade dessas instâncias, fornecendo maneiras de acessar e modificar as propriedades da instância ou fornecendo funcionalidade relacionada à finalidade da instância. Os métodos de instância têm exatamente a mesma sintaxe que as funções, conforme descrito em _Funções na linguagem Swift_.

Você escreve um método de instância dentro das chaves de abertura e fechamento do tipo ao qual ele pertence. Um método de instância tem acesso implícito a todos os outros métodos de instância e propriedades desse tipo. Um método de instância pode ser chamado apenas em uma instância específica do tipo ao qual pertence. Ele não pode ser chamado isoladamente sem uma instância existente.

Aqui está um exemplo que define uma classe `Contador` simples, que pode ser usada para contar o número de vezes que uma ação ocorre:

``` swift
class Contador {
    var valor = 0
    
    func incrementa() {
        valor += 1
    }
    
    func incrementa(por quantidade: Int) {
        valor += quantidade
    }
    
    func redefine() {
        valor = 0
    }
}
```

A classe `Contador` define três métodos de instância:

- `incrementa()` - incrementa o contador em 1;
- `incrementa(por: Int)` incrementa o contador por um valor inteiro especificado;
- `redefine()` - redefine o contador para zero;

A classe `Contador` também declara uma propriedade variável, `valor`, para acompanhar o valor atual do contador.

Você chama métodos de instância com a mesma sintaxe de ponto das propriedades:

``` swift
let contador = Contador()
// o valor inicial para o contador é 0

contador.incrementa()
// o valor do contador agora é 1

contador.incrementa(por: 5)
// o valor do contador agora é 6

contador.redefine()
// o valor do contador agora é 0
```

Os parâmetros de função podem ter um nome (para uso no corpo da função) e um rótulo de argumento (para uso ao chamar a função), conforme descrito em _Funções na linguagem Swift_. O mesmo vale para parâmetros de método, porque métodos são apenas funções associadas a um tipo.

## A propriedade Self

Cada instância de um tipo tem uma propriedade implícita chamada `self`, que é exatamente equivalente à própria instância. Você usa a propriedade `self` para fazer referência à instância atual em seus próprios métodos de instância.

O método `incrementa()` no exemplo acima poderia ter sido escrito assim:

``` swift
func incrementa() {
    self.valor += 1
}
```

Na prática, você não precisa escrever `self` em seu código com muita frequência. Se você não escrever `self` explicitamente, o Swift assume que você está se referindo a uma propriedade ou método da instância atual sempre que usar uma propriedade ou nome de método conhecido dentro de um método. Essa suposição é demonstrada pelo uso de `valor` (em vez de `self.valor`) dentro dos três métodos de instância para `Contador`.

A principal exceção a essa regra ocorre quando um nome de parâmetro para um método de instância tem o mesmo nome de uma propriedade dessa instância. Nessa situação, o nome do parâmetro tem precedência e torna-se necessário fazer referência à propriedade de forma mais qualificada. Você usa então a propriedade `self` para distinguir entre o nome do parâmetro e o nome da propriedade.

Aqui, `self` remove a ambiguidade entre um parâmetro de método chamado `x` e uma propriedade de instância que também é chamada de `x`:

``` swift
class Ponto {
    var x = 0.0 
    var y = 0.0

    init(x: Double, y: Double) {
        self.x = x
        self.y = y
    }

    func estaAhDireitaDe(x: Double) -> Bool {
        return self.x > x
    }
}

let ponto = Ponto(x: 4.0, y: 5.0)

print("Este ponto está à direita da linha onde x = 1.0 ?: \(ponto.estaAhDireitaDe(x: 1.0))")
// Imprime "Este ponto está à direita da linha onde x = 1.0 ?: true"
```

Sem o prefixo `self`, o Swift assumiria que ambos os usos de `x` se referiam ao parâmetro do método chamado `x`. A mesma ideia fica clara ao analizar os exemplos de inicializadores demonstrados nos trechos de código acima.

## Sobre Herança: Subclassing e Overriding

Uma classe pode herdar métodos, propriedades e outras características de outra classe. Quando uma classe herda características de outra, ela é conhecida como subclasse (_subclass_ de _subclassing_), e a classe da qual ela herda é conhecida como sua superclasse (_superclass_). A herança é um comportamento fundamental que diferencia classes de outros tipos em Swift.

As classes em Swift podem chamar e acessar métodos e propriedades pertencentes à sua superclasse e podem fornecer suas próprias versões de substituição (_override_) desses métodos e propriedades para especializar ou modificar seu comportamento. O Swift ajuda a garantir que seus _overridings_ estejam corretos verificando se a definição de substituição tem uma definição de superclasse correspondente.

### Definindo uma classe base

Qualquer classe que não herda de outra classe é conhecida como classe base.

> Nota: As classes Swift não herdam de uma classe base universal. As classes que você define sem especificar uma superclasse automaticamente se tornam classes base para você construir.

O exemplo abaixo define uma classe base chamada `Veiculo`. Essa classe base define uma propriedade armazenada chamada `velocidadeAtual`, com um valor padrão de `0.0` (inferindo um tipo de propriedade `Double`). O valor da propriedade `velocidadeAtual` é usado por um método `imprimeStatus()` para registrar a informação atual sobre a locomoção.

A classe base `Veiculo` também define um método chamado `geraRuido()`. Este método não faz nada para uma instância base `Veículo`, mas será customizado por subclasses de `Veiculo` posteriormente:

``` swift
class Veiculo {
    var velocidadeAtual = 0.0
    
    func imprimeStatus() {
        print("viajando a \(velocidadeAtual) km/H")
    }
    
    func geraRuido() {
        // não faz nada - um veículo arbitrário não precisa emitir ruído relativo ao seu funcionamento
    }
}
```

Você cria uma nova instância de `Veiculo` com a sintaxe do inicializador, que é escrita como um nome de tipo seguido por parênteses vazios:

``` swift
let veiculo = Veiculo()
```

Tendo criado uma nova instância `Veiculo`, você pode acessar seu método `imprimeStatus()` para imprimir uma descrição legível da velocidade atual do veículo:

``` swift
veiculo.imprimeStatus()
// Imprime: "viajando a 0.0 km/H"
```

A classe `Veiculo` define características comuns para um veículo arbitrário, mas não é muito útil por si só. Para torná-lo mais útil, você precisa refiná-lo para descrever tipos mais específicos de veículos.

### Criando Subclasses (_Subclassing_)

Criar uma subclasse é o ato de basear uma nova classe em uma classe existente. A subclasse herda características da classe existente, que você pode especializar. Você também pode adicionar novas características à subclasse.

Para indicar que uma subclasse tem uma superclasse, escreva o nome da subclasse antes do nome da superclasse, separados por dois pontos:

``` swift
class UmaSubclass: SuaSuperclass {
    // definição da subclasse vai aqui
}
```

O exemplo a seguir define uma subclasse chamada `Bicicleta`, com uma superclasse de `Veiculo`:

``` swift
class Bicicleta: Veiculo {
    var temCesta = false
}
```

A nova classe `Bicicleta` ganha automaticamente todas as características de `Veiculo`, como sua propriedade `velocidadeAtual` e seus métodos `imprimeStatus()` e  `geraRuido()`.

Além das características que herda, a classe `Bicicleta` define uma nova propriedade armazenada, `temCesta`, com um valor padrão false (inferindo um tipo `Bool` para a propriedade).

Por padrão, qualquer nova instância de `Bicicleta` que você criar não terá uma cesta. Você pode definir a propriedade `temCesta` como `true` para uma instância de `Bicicleta` específica depois que essa instância for criada:

``` swift
let bicicleta = Bicicleta()
bicicleta.temCesta = true
```

Você também pode modificar a propriedade herdada `velocidadeAtual` de uma instância `Bicicleta` e consultar o método `imprimeStatus()` também herdado:

``` swift
bicicleta.velocidadeAtual = 15.0
bicicleta.imprimeStatus()
// Imprime: "viajando a 15.0 km/H"

As subclasses podem ser subclasses. O próximo exemplo cria uma subclasse de `Bicicleta` para uma bicicleta de dois lugares conhecida como “tandem”:

``` swift
class Tandem: Bicicleta {
    var numeroDePassageiros = 0
}
```

`Tandem` herda todas as propriedades e métodos de `Bicicleta`, que por sua vez herda todas as propriedades e métodos de `Veiculo`. A subclasse `Tandem` também adiciona uma nova propriedade armazenada chamada `numeroDePassageiros`, com um valor padrão de `0`.

Se você criar uma instância de `Tandem`, poderá trabalhar com qualquer uma de suas propriedades novas e herdadas e consultar a propriedade de descrição somente leitura herdada de `Veiculo`:

``` swift
let tandem = Tandem()
tandem.temCesta = true
tandem.numeroDePassageiros = 2
tandem.velocidadeAtual = 22.0

tandem.imprimeStatus()
// viajando a 22.0 km/H
```

### Sobrescrita (_Overriding_)

Uma subclasse pode fornecer sua própria implementação personalizada de um método ou propriedade de instância que, de outra forma, herdaria de uma superclasse. Isso é conhecido como **sobrescrita**.

Para substituir uma característica que de outra forma seria herdada, você prefixa sua definição de sobrescrita com a palavra-chave `override`. Isso esclarece que você pretende fornecer uma substituição e não forneceu uma definição correspondente por engano. A substituição por acidente pode causar um comportamento inesperado, e quaisquer sobrescritas sem a palavra-chave `override` são diagnosticadas como um erro quando seu código é compilado.

A palavra-chave `override` também solicita que o compilador Swift verifique se a superclasse de sua subclasse (ou uma de suas mães) tem uma declaração que corresponde à que você forneceu para a sobrescrita. Essa verificação garante que sua definição de sobrescrita esteja correta.

#### Acessando Métodos e Propriedades da Superclasse

Quando você fornece um método ou propriedade sobrescrita para uma subclasse, às vezes é útil usar a implementação da superclasse existente como parte de sua substituição. Por exemplo, você pode especializar o comportamento dessa implementação existente ou armazenar um valor modificado em uma variável herdada existente.

Onde for apropriado, você acessa a versão da superclasse de um método ou propriedade usando o prefixo `super`:

- Um método sobrescrito chamado `algumMetodo()` pode chamar a versão da superclasse de `algumMetodo()` chamando `super.algumMetodo()` dentro da implementação do método sobrescrito.
- Uma propriedade sobrescrita chamada `algumaPropriedade` pode acessar a versão da superclasse de `algumaPropriedade` como `super.algumaPropriedade` dentro da implementação getter* ou setter* de sobrescrita.

> \* Veremos no futuro a sintaxe de facilitação para _getters_ e _setters_.

#### Sobrescrevendo Métodos (_Method Overriding_)

Você pode sobrescrever um método herdado na sua subclasse para fornecer uma implementação personalizada ou alternativa do método.

O exemplo a seguir define uma nova subclasse de `Veiculo` chamada `Trem`, que substitui o método `geraRuido()` que `Trem` herda de `Veiculo`:

``` swift
class Trem: Veiculo {
     override func geraRuido() {
         print("Tic Tic Tic Piuii")
     }
}
```

Se você criar uma nova instância de `Trem` e chamar seu método `geraRuido()`, poderá ver que a versão da subclasse `Trem` do método é chamada:

``` swift
let trem = Trem()
trem.geraRuido()
// Imprime "Tic Tic Tic Piuii"
```
