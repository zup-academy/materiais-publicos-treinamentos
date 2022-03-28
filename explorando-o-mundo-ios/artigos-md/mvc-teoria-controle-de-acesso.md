# Controle de Acesso

>Esse material teórico foi adaptado da fonte original disponível em https://docs.swift.org/swift-book/LanguageGuide/AccessControl.html

O controle de acesso restringe o acesso a partes de um determinado código do código em outros módulos e arquivos fonte. Esse recurso permite ocultar os detalhes de implementação de seu código e especificar uma interface preferencial por meio da qual esse código pode ser acessado e usado.

Você pode atribuir níveis de acesso específicos a tipos individuais (classes, _structs_ e _enums_), bem como a propriedades, métodos e inicializadores pertencentes a esses tipos.

Além de oferecer vários níveis de controle de acesso, o Swift reduz a necessidade de especificar níveis de controle de acesso explícitos, fornecendo níveis de acesso padrão para cenários típicos. De fato, se você estiver escrevendo um aplicativo de destino único, talvez não seja necessário especificar níveis de controle de acesso explícitos.

> Nota: Os vários aspectos do seu código que podem ter controle de acesso aplicado a eles (propriedades, tipos, funções e assim por diante) são chamados de “entidades” nas seções abaixo, para resumir.

## Módulos e Arquivos Fonte

O modelo de controle de acesso do Swift é baseado no conceito de módulos e arquivos fonte.

Um módulo é uma única unidade de distribuição de código – um framework ou aplicativo que é construído e enviado como uma única unidade e que pode ser importado por outro módulo com a palavra-chave `import` do Swift.

Cada destino de compilação (como um pacote de aplicativos ou framework) no Xcode é tratado como um módulo separado no Swift. Se você agrupar aspectos do código do seu aplicativo como um framework independente - talvez para encapsular e reutilizar esse código em vários aplicativos - tudo o que você definir nesse framework fará parte de um módulo separado quando for importado e usado em um aplicativo, ou quando é usado em outro framework.

Um arquivo fonte é um arquivo único de código-fonte Swift em um módulo (na verdade, um arquivo único em um aplicativo ou framework). Embora seja comum definir tipos individuais em arquivos fonte separados, um único arquivo fonte pode conter definições para vários tipos, funções e assim por diante.

## Níveis de acesso

O Swift fornece cinco níveis de acesso diferentes para entidades em seu código. Esses níveis de acesso são relativos ao arquivo fonte no qual uma entidade está definida e também em relação ao módulo ao qual o arquivo fonte pertence.

* O acesso `open` e o acesso `public` permitem que as entidades sejam usadas em qualquer arquivo fonte de seu módulo de definição e também em um arquivo fonte de outro módulo que importa o módulo de definição. Você normalmente usa acesso `open` ou `public` ao especificar a interface pública para um framework. A diferença entre acesso `open` e `public` é descrita abaixo.
* O acesso `internal` permite que as entidades sejam usadas em qualquer arquivo fonte de seu módulo de definição, mas não em nenhum arquivo fonte fora desse módulo. Você normalmente usa o acesso `internal` ao definir a estrutura interna de um aplicativo ou de um framework.
* O acesso `fileprivate` restringe o uso de uma entidade ao seu próprio arquivo fonte de definição. Use o acesso `fileprivate` para ocultar os detalhes de implementação de uma funcionalidade específica quando esses detalhes forem usados ​​em um arquivo inteiro.
* O acesso `private` restringe o uso de uma entidade à declaração ao qual ela pertence e às extensões dessa declaração que estão no mesmo arquivo. Use o acesso `private` para ocultar os detalhes de implementação de uma funcionalidade específica quando esses detalhes forem usados ​​apenas em uma única declaração.

O acesso `open` é o nível de acesso mais alto (menos restritivo) e o acesso `private` é o nível de acesso mais baixo (mais restritivo).

O acesso `open` se aplica apenas a classes e membros de classe, e difere do acesso `public` ao permitir que código fora do módulo crie subclasses e sobrescritas. Marcar uma classe como `open` indica explicitamente que você considerou o impacto do código de outros módulos usando essa classe como uma superclasse e que projetou o código da sua classe de acordo.

### Guia do Princípio dos Níveis de Acesso

Os níveis de acesso no Swift seguem um princípio geral: nenhuma entidade pode ser definida em termos de outra entidade que tenha um nível de acesso mais baixo (mais restritivo).

Por exemplo:

* Uma variável `public` não pode ser definida como tendo um tipo `internal`, `fileprivate` ou `private`, porque o tipo pode não estar disponível em todos os lugares em que a variável pública é usada.
* Uma função não pode ter um nível de acesso mais alto do que seus tipos de parâmetro e tipo de retorno, porque a função pode ser usada em situações em que seus tipos constituintes não estão disponíveis para o código ao redor.

As implicações específicas deste princípio orientador para diferentes aspectos da linguagem são abordadas em detalhes abaixo.

### Níveis de acesso padrão

Todas as entidades em seu código (com algumas exceções específicas, conforme descrito posteriormente) têm um nível de acesso padrão `internal` se você não especificar um nível de acesso explícito. Como resultado, em muitos casos, você não precisa especificar um nível de acesso explícito em seu código.

### Níveis de acesso para single-target apps

Quando você escreve um aplicativo simples de artefato único, o código em seu aplicativo normalmente é autocontido dentro do aplicativo e não precisa ser disponibilizado fora do módulo do aplicativo. O nível de acesso padrão `internal` já corresponde a esse requisito. Portanto, você não precisa especificar um nível de acesso personalizado. Você pode, no entanto, querer marcar algumas partes do seu código como `fileprivate` ou `private` para ocultar seus detalhes de implementação de outro código dentro do módulo do aplicativo.

### Níveis de acesso para frameworks

Ao desenvolver um framework, marque a interface pública para esse framework como `open` ou `public` para que possa ser visualizada e acessada por outros módulos, como um aplicativo que importa o framework. Essa interface pública constitui a interface de programação de aplicativos (ou API) do framework.

> Nota: Quaisquer detalhes de implementação interna de seu framework ainda podem usar o nível de acesso padrão `internal` ou podem ser marcados como `private` ou `fileprivate` se você quiser ocultá-los de outras partes do código interno do framework. Você precisa marcar uma entidade como `open` ou `public` somente se quiser que ela se torne parte da API do seu framework.

## Sintaxe para controle de acesso

Defina o nível de acesso para uma entidade colocando um dos modificadores `open`, `public`, `internal`, `fileprivate` ou `private` no início da declaração da entidade.

``` swift
public class UmaClassePublica {}
internal class UmaClasseInterna {}
fileprivate class UmaClassePrivadaAoArquivoFonte {}
private class UmaClassePrivada {}

public var umaVariavelPublica = 0
let internal umaConstanteInterna = 0
fileprivate func umaFuncaoPrivadaAoArquivoFonte() {}
private func umaFuncaoPrivada() {}
```

A menos que especificado de outra forma, o nível de acesso padrão é `internal`, conforme descrito acima. Isso significa que `UmaClasseInterna` e `umaConstanteInterna` podem ser escritos sem um modificador de nível de acesso explícito e ainda terão um nível de acesso `internal`:

``` swift
class UmaClasseInterna {} // implicitamente interna
let umaConstanteInterna = 0 // implicitamente interna
```

## Tipos customizados

Se você quiser especificar um nível de acesso explícito para um tipo customizado, faça isso no momento em que definir o tipo. O novo tipo pode ser usado sempre que seu nível de acesso permitir. Por exemplo, se você definir uma classe `fileprivate`, essa classe só poderá ser usada como o tipo de uma propriedade, ou como um parâmetro de função ou tipo de retorno, no arquivo de origem no qual a classe `fileprivate` está definida.

O nível de controle de acesso de um tipo também afeta o nível de acesso padrão dos membros desse tipo (suas propriedades, métodos e inicializadores). Se você definir o nível de acesso de um tipo como `private` ou `fileprivate`, o nível de acesso padrão de seus membros também será `private` ou `fileprivate`. Se você definir o nível de acesso de um tipo como `internal` ou `public` (ou usar o nível de acesso padrão `internal` sem especificar um nível de acesso explicitamente), o nível de acesso padrão dos membros do tipo será `internal`.

> Importante: Um tipo `public` por padrão tem membros definidos sob o nível de acesso `internal`, e não públicos como poderia se esperar. Se você quiser que um membro de tipo seja `public`, você deve marcá-lo explicitamente como tal. Esse requisito garante que a API pública para um tipo seja algo que você opte por publicar e evita apresentar o funcionamento interno de um tipo como API pública por engano.

``` swift
public class UmaClassePublica { // classe explicitamente pública
    public var umaPropriedadePublica = 0 // membro da classe explicitamente público
    var umaVariavelInterna = 0 // membro de classe implicitamente interno
    fileprivate func umMetodoFileprivate() {} // membro da classe explicitamente restrito ao arquivo
    private func umMetodoPrivado() {} // membro de classe explicitamente privado
}

class UmaClasseInterna { // classe interna implicitamente
    var umaPropriedadeInterna = 0 // membro de classe implicitamente interno
    fileprivate func umMetodoFileprivate() {} // // membro da classe explicitamente restrito ao arquivo
    private func umMetodoPrivado() {} // membro de classe explicitamente privado
}

fileprivate class UmaClasseFileprivate { // classe explicitamente restrita ao arquivo
    func umMetodoFileprivate() {} // membro da classe explicitamente restrito ao arquivo
    private func umMetodoPrivado() {} // membro de classe explicitamente privado
}

private class UmaClassePrivada { // classe privada explicitamente
    func umMetodoPrivado() {} // membro de classe implicitamente privado
}
```

### Tuplas

O nível de acesso para um tipo de tupla é o nível de acesso mais restritivo de todos os tipos usados ​​nessa tupla. Por exemplo, se você compor uma tupla de dois tipos diferentes, um com acesso `internal` e outro com acesso `private`, o nível de acesso para esse tipo de tupla composta será `private`.

> Nota: Os tipos de tupla não têm uma definição particular da forma como têm as classes, _structs_, _enums_ e funções. O nível de acesso de um tipo de tupla é determinado automaticamente a partir dos tipos que compõem o tipo de tupla e não pode ser especificado explicitamente.

## Tipos de função

O nível de acesso para um tipo de função é calculado como o nível de acesso mais restritivo dos tipos de parâmetro da função e do tipo de retorno. Você deve especificar o nível de acesso explicitamente como parte da definição da função se o nível de acesso calculado da função não corresponder ao padrão contextual.

O exemplo abaixo define uma função global chamada `umaFuncao()`, sem fornecer um modificador de nível de acesso específico para a função em si. Você pode esperar que esta função tenha o nível de acesso padrão de `internal`, mas esse não é o caso. Na verdade, `umaFuncao()` não irá compilar conforme escrito abaixo:

``` swift
func umaFuncao() -> (UmaClasseInterna, UmaClassePrivada) {
    // implementação da função vai aqui
}
```

O tipo de retorno da função é um tipo de tupla composto por duas das classes personalizadas. Uma dessas classes é definida como interna e a outra é definida como privada. Portanto, o nível de acesso geral do tipo de tupla composta é `private` (o nível de acesso mais restritivo dos tipos constituintes da tupla).

Como o tipo de retorno da função é `private`, você deve marcar o nível de acesso geral da função com o modificador `private` para que a declaração da função seja válida:

``` swift
private func umaFuncao() -> (UmaClasseInterna, UmaClassePrivada) {
    // implementação da função vai aqui
}
```

Não é válido marcar a definição de `umaFuncao()` com os modificadores `public` ou `internal`, ou usar a configuração padrão de `internal`, porque os usuários públicos ou internos da função podem não ter acesso apropriado à classe privada usada no tipo de retorno da função .

### _Enums_

Os casos individuais de uma _enum_ recebem automaticamente o mesmo nível de acesso que a _enum_ a que pertencem. Você não pode especificar um nível de acesso diferente para casos de _enum_ individuais.

No exemplo abaixo, a _enum_ `PontoDaBussola` tem um nível de acesso explícito `public`. Os casos de _enum_ `.norte`, `.sul`, `.leste` e `.oeste`, portanto, também têm um nível de acesso público:

``` swift
public enum PontoDaBussola {
    case norte
    case sul
    case leste
    case oeste
}
```

#### Valores brutos e valores associados _(raw values e associated values)_

Os tipos usados ​​para quaisquer valores brutos ou valores associados em uma definição de _enum_ devem ter um nível de acesso pelo menos tão alto quanto o nível de acesso da _enum_. Por exemplo, você não pode usar um tipo `private` como o tipo de valor bruto de uma _enum_ com um nível de acesso `internal`.

### Tipos aninhados

O nível de acesso de um tipo aninhado é o mesmo do tipo que o contém, a menos que o tipo que o contém seja público. Os tipos aninhados definidos em um tipo `public` têm um nível de acesso automático `internal`. Se desejar que um tipo aninhado em um tipo público esteja disponível publicamente, você deve declarar explicitamente o tipo aninhado como `public`.

## Subclasses

Você pode definir subclasses de qualquer classe que possa ser acessada no contexto de acesso atual e que esteja definida no mesmo módulo que a subclasse. Você também pode definir subclasses de qualquer classe `open` definida em um módulo diferente. Uma subclasse não pode ter um nível de acesso mais alto que sua superclasse — por exemplo, você não pode escrever uma subclasse `public` de uma superclasse `internal`.

Além disso, para classes definidas no mesmo módulo, você pode sobrescrever qualquer membro de classe (método, propriedade ou inicializador) visível em um determinado contexto de acesso. Para classes definidas em outro módulo, você pode sobrescrever qualquer membro de classe com nível de acesso `open`.

Uma sobrescrita pode tornar um membro de classe herdado mais acessível do que sua versão de superclasse. No exemplo abaixo, a classe `A` é uma classe `public` com um método `fileprivate` chamado `umMetodo()`. A classe `B` é uma subclasse de `A`, com um nível de acesso reduzido de `internal`. No entanto, a classe `B` fornece uma sobrescrita de `umMetodo()` com um nível de acesso `internal`, que é maior que a implementação original de `umMetodo()`:

``` swift
public class A {
    fileprivate func umMetodo() {}
}

internal class B: A {
    override internal func umMetodo() {}
}
```

É até válido para um membro da subclasse chamar um membro da superclasse que tenha permissões de acesso menores do que o membro da subclasse, desde que a chamada para o membro da superclasse ocorra dentro de um contexto de nível de acesso permitido (ou seja, dentro do mesmo arquivo de origem que a superclasse para uma chamada de membro `fileprivate` ou dentro do mesmo módulo que a superclasse para uma chamada de membro `internal`):

``` swift
public class A {
    fileprivate func umMetodo() {}
}

internal class B: A {
    override internal func umMetodo() {
        super.umMetodo()
    }
}
```

Como a superclasse `A` e a subclasse `B` são definidas no mesmo arquivo fonte, é válido para a implementação `B` de `umMetodo()` chamar `super.umMetodo()`.

## Constantes, Variáveis e Propriedades

Uma constante, variável ou propriedade não pode ser mais pública que seu tipo. Não é válido escrever uma propriedade `public` com um tipo `private`, por exemplo. Se uma constante, variável ou propriedade fizer uso de um tipo `private`, a constante, variável ou propriedade também deverá ser marcada como `private`:

``` swift
private var instanciaPrivada = UmaClassePrivada()
```

### Getters e Setters

_Getters_ e _setters_ para constantes, variáveis e propriedades recebem automaticamente o mesmo nível de acesso que a constante, variável ou propriedade a que pertencem.

Você pode dar a um _setter_ um nível de acesso mais baixo do que seu _getter_ correspondente, para restringir o escopo de leitura/gravação dessa variável ou propriedade. Você atribui um nível de acesso mais baixo escrevendo `fileprivate(set)`, `private(set)` ou `internal(set)` antes da palavra chave de definição `var`, por exemplo.

> Nota: Esta regra se aplica a propriedades armazenadas, bem como propriedades computadas. Mesmo que você não escreva um _getter_ e um _setter_ explícitos para uma propriedade armazenada, o Swift ainda sintetiza um _getter_ e um _setter_ implícitos para você fornecer acesso à propriedade armazenada. Use `fileprivate(set)`, `private(set)` e `internal(set)` para alterar o nível de acesso desse _setter_ sintetizado exatamente da mesma maneira que para um _setter_ explícito em uma propriedade computada.

O exemplo abaixo define uma _struct_ chamada `StringTrackeada`, que acompanha o número de vezes que uma propriedade de string é modificada:

``` swift
struct StringTrackeada {
    private(set) var numeroDeEdicoes = 0
    var valor: String = "" {
        didSet {
            numeroDeEdicoes += 1
        }
    }
}
```

A _struct_ `StringTrackeada` define uma propriedade armazenada do tipo `String` chamada `value`, com um valor inicial de `""` (uma string vazia). A _struct_ também define uma propriedade armazenada inteira chamada `numeroDeEdicoes`, que é usada para rastrear o número de vezes que esse valor é modificado. Esse rastreamento de modificação é implementado com um observador de propriedade didSet na propriedade `valor`, que incrementa `numeroDeEdicoes` toda vez que a propriedade `valor` é definida para um novo valor.

A _struct_ `StringTrackeada` e a propriedade `valor` não fornecem um modificador de nível de acesso explícito e, portanto, ambas recebem o nível de acesso padrão `internal`. No entanto, o nível de acesso para a propriedade `numeroDeEdicoes` é marcado com um modificador `private(set)` para indicar que o _getter_ da propriedade ainda tem o nível de acesso padrão `internal`, mas a propriedade é configurável apenas de dentro do código que faz parte da _struct_ `StringTrackeada`. Isso permite que `StringTrackeada` modifique a propriedade `numeroDeEdicoes` internamente, mas apresente a propriedade como uma propriedade somente leitura quando usada fora da definição da _struct_.

Se você criar uma instância `StringTrackeada` e modificar seu valor de string algumas vezes, poderá ver a atualização do valor da propriedade `numeroDeEdicoes` para corresponder ao número de modificações:

``` swift
var string = StringTrackeada()
string.valor = "Esta string será rastreada."
string.valor += "Esta edição incrementará numeroDeEdicoes."
string.valor += " E este também."
print("O número de edições é \(string.numeroDeEdicoes)")
// Imprime "O número de edições é 3"
```

Embora você possa consultar o valor atual da propriedade `numeroDeEdicoes` de outro arquivo fonte, não pode modificar a propriedade de outro arquivo fonte. Essa restrição protege os detalhes de implementação da funcionalidade de acompanhamento de edição de `StringTrackeada`, enquanto ainda fornece acesso conveniente a um aspecto dessa funcionalidade.

Observe que você pode atribuir um nível de acesso explícito para um _getter_ e um _setter_, se necessário. O exemplo abaixo mostra uma versão da _struct_ `StringTrackeada` na qual a _struct_ é definida com um nível de acesso explícito de `public`. Os membros da _struct_ (incluindo a propriedade `numeroDeEdicoes`), portanto, têm um nível de acesso interno por padrão. Você pode tornar público o _getter_ de propriedade `numeroDeEdicoes` da _struct_ e privado seu _setter_ de propriedade, combinando os modificadores de nível de acesso `public` e `private(set)`:

``` swift
public struct StringTrackeada {
    public private(set) var numeroDeEdicoes = 0
    public var valor: String = "" {
        didSet {
            numeroDeEdicoes += 1
        }
    }
    public init() {}
}
```

## Inicializadores

Inicializadores personalizados podem receber um nível de acesso menor ou igual ao tipo que eles inicializam. A única exceção é para inicializadores _required_. Um inicializador _required_ deve ter o mesmo nível de acesso que a classe a que pertence.

Assim como os parâmetros de função e método, os tipos de parâmetros de um inicializador não podem ser mais privados do que o próprio nível de acesso do inicializador.

### Inicializadores padrão

O Swift fornece automaticamente um inicializador padrão sem argumentos para qualquer _struct_ ou classe base que não forneça pelo menos um inicializador, provendo valores padrão para todas as suas propriedades 

Um inicializador padrão tem o mesmo nível de acesso que o tipo que inicializa, a menos que esse tipo seja definido como `public`. Para um tipo definido como `public`, o inicializador padrão é considerado `internal`. Se você deseja que um tipo público seja inicializável com um inicializador sem argumentos quando usado em outro módulo, você deve fornecer explicitamente um inicializador público sem argumentos como parte da definição do tipo.

### Inicializadores padrão para _structs_

O inicializador padrão para um tipo _struct_ é considerado `private` se qualquer uma das propriedades armazenadas da _struct_ for privada. Da mesma forma, se qualquer uma das propriedades armazenadas da _struct_ for `fileprivate`, o inicializador será `fileprivate`. Caso contrário, o inicializador tem um nível de acesso `internal`.

Assim como com o inicializador padrão acima, se você quiser que um tipo de _struct_ pública seja inicializável com um inicializador de membro quando usado em outro módulo, você deve fornecer um inicializador de membro público como parte da definição do tipo.

## Type aliases

Quaisquer _type aliases_ que você definir são tratados como tipos distintos para fins de controle de acesso. Um _type alias_ pode ter um nível de acesso menor ou igual ao nível de acesso do tipo que ele denomina. Por exemplo, um _type alias_ privado pode denominar um tipo `private`, `fileprivate`, `internal`, `public` ou `open`, mas um _type alias_ público não pode denominar um tipo `internal`, `fileprivate` ou `private`.
