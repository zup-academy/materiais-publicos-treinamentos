# Model-View-Controller (MVC)

> Esse material teórico foi traduzido e adaptado da fonte disponível em https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Model-View-Controller/Model-View-Controller.html. Alguns conceitos presentes na fonte original podem não ser aplicáveis em nosso contexto ou mesmo representarem formas de trabalho que já foram atualizadas. No entanto, o conteúdo referente à apresentação do padrão arquitetural MVC é atemporal e pode e deve ser reaproveitado, pois sua aplicação ainda se faz presente nas decisões de design da plataforma iOS.

O padrão de design Model-View-Controller (MVC) é bastante antigo contando com variações existentes desde pelo menos os primeiros dias de Smalltalk. É um padrão de alto nível, pois se preocupa com a arquitetura global da aplicação e classifica objetos de acordo com as funções gerais que eles desempenham no software. É também um padrão composto, pois compreende vários padrões mais elementares.

Os programas orientados a objetos se beneficiam de várias maneiras adaptando o padrão MVC para seus projetos. Muitos objetos nesses programas tendem a ser mais reutilizáveis ​​e suas interfaces tendem a ser mais bem definidas. Os programas em geral são mais adaptáveis ​​às mudanças de requisitos – em outras palavras, eles supostamente são mais facilmente extensíveis do que programas que não são baseados em MVC. Além disso, muitas tecnologias e arquiteturas no Cocoa framework são baseadas em MVC e exigem que seus objetos personalizados desempenhem uma das funções definidas pelo MVC.

## Funções e relacionamentos de objetos MVC

O padrão de projeto MVC considera que existem três tipos de objetos: objetos de modelo, objetos de visualização (_view_) e objetos de controlador. O padrão MVC define as funções que esses tipos de objetos desempenham no aplicativo e suas formas de comunicação. Ao projetar um aplicativo, um passo importante é escolher – ou criar classes personalizadas para – objetos que se enquadram em um desses três grupos. Cada um dos três tipos de objetos é separado dos outros por limites abstratos e se comunicam com objetos dos outros tipos através desses limites.

### Objetos de modelo encapsulam dados e comportamentos básicos do negócio

Objetos de modelo representam conhecimento e expertise do negócio. Eles mantêm os dados de um aplicativo e definem a lógica que manipula esses dados. Um aplicativo MVC bem projetado tem todos os seus dados importantes encapsulados em objetos de modelo. Quaisquer dados que façam parte do estado persistente do aplicativo (seja esse estado persistente armazenado em arquivos ou bancos de dados) devem residir nos objetos do modelo assim que os dados forem carregados no aplicativo. Por representarem conhecimento e expertise relacionados a um domínio de problema específico, tendem a ser reutilizáveis.

Idealmente, um objeto de modelo não tem conexão explícita com a interface do usuário usada para apresentá-lo e editá-lo. Por exemplo, se você tiver um objeto de modelo que representa uma pessoa (digamos que você esteja escrevendo um catálogo de endereços), convém armazenar uma data de nascimento. Ela é uma boa candidata para ser armazenada em seu objeto de modelo `Pessoa`. No entanto, uma string de formato de data ou outras informações sobre como essa data deve ser apresentada provavelmente é melhor ser mantida em outro lugar.

Na prática, essa separação nem sempre é a melhor coisa, e há algum espaço para flexibilidade aqui, mas em geral um objeto de modelo não deve se preocupar com questões de interface e apresentação. Um exemplo em que uma exceção para esta regra é razoável é um aplicativo de desenho que possui objetos de modelo que representam os gráficos exibidos. Faz sentido que os objetos gráficos saibam se desenhar porque a principal razão de sua existência é definir uma coisa visual. Mas, mesmo neste caso, os objetos gráficos não devem depender de viver em quaisquer que sejam as _views_, e não devem ser encarregados de saber quando devem ser desenhados. Eles devem ser solicitados a se desenharem pelo objeto de visualização que deseja apresentá-los. As próprias regras de negócio indicam a necessidade de algo essencialmente visual ser modelado na camada de modelo.

Classes e objetos em seus projetos que você pode incluir nesta camada são, geralmente:

* Código que modela entidades do domínio: você preferencialmente usa _structs_ para representar objetos como `Pessoa`, no exemplo acima, em aplicativos cujo domínio do problema a envolva;
* Código de _network_: você preferencialmente usa apenas uma única classe para comunicação de rede em todo o seu aplicativo. Ele facilita a abstração de conceitos comuns a todas as requisições web, como cabeçalhos HTTP, resposta e tratamento de erros, etc;
* Código de persistência: você usa isso ao persistir dados em um banco de dados, Core Data ou armazenar dados em qualquer outro mecanismo de persistência de um dispositivo.
* Código de _parsing_ para o modelo: você deve incluir objetos que adaptam e/ou convertem respostas da camada de rede para o seu modelo. Por exemplo, em objetos de modelo Swift, você pode usar a codificação/decodificação JSON para lidar com a conversão. Dessa forma, tudo é melhor isolado e sua camada de rede não precisa conhecer os detalhes de implementação de todos os seus objetos de modelo. A lógica de negócios e de _parsing_ é toda autocontida no modelo.
* _Managers_ e outras camadas/classes de abstração: É normal ter os objetos tipicamente tratados por “gerenciador” de algo que geralmente oferecem implementações que ajudam a coordenar alguma operação ou tarefa necessária orquestrando outras classes. Eles também podem ser wrappers em torno de APIs mais robustas a nível de infraestrutura como um wrapper para URLSession no iOS, uma classe para gerenciar o trabalho com geolocalização ou notificações.
* _Delegates_ e _data sources_: algo que é menos usual entre os desenvolvedores, se apoiar em objetos de modelo para serem a fonte de dados ou delegates de componentes como _table views_ ou _collection views_. É comum ver essas implementações viverem na camada de controle, mesmo quando há grande concentração de lógica personalizada que faria mais sentido viver isolada na camada de modelo.

### Objetos de view devem apresentar informações ao usuário

Um objeto de _view_ sabe como exibir os dados e pode permitir que os usuários manipulem os dados do modelo do aplicativo. A _view_ não deve ser responsável por armazenar os dados que está exibindo. (Isso não significa que a _view_ nunca realmente armazena os dados que está exibindo, é claro. Uma _view_ pode armazenar dados em cache ou utilizar outras técnicas semelhantes por motivos de performance). Um objeto de _view_ pode ser responsável por exibir apenas uma parte de um objeto de modelo, ou um objeto de modelo inteiro, ou mesmo muitos objetos de modelo diferentes. As _views_ vêm em muitas variedades diferentes.

Os objetos de _view_ tendem a ser reutilizáveis ​​e configuráveis ​​e fornecem consistência entre os aplicativos. O UIKit framework define um grande número de objetos de _view_ e fornece muitos deles na biblioteca de objetos do Interface Builder. Ao reutilizar os objetos de visualização do UIKit, como objetos de UIButton, você garante que os botões em seu aplicativo se comportem como botões em qualquer outro aplicativo que suporte o framework, garantindo um alto nível de consistência na aparência e no comportamento entre os aplicativos.

Uma _view_ deve garantir que esteja exibindo o modelo corretamente. Consequentemente, ele geralmente precisa saber sobre as mudanças no modelo. Como os objetos de modelo não devem ser vinculados a objetos de exibição específicos, eles precisam de uma maneira genérica de indicar que foram alterados.

Classes e objetos em seus projetos que você pode incluir nesta camada são, geralmente:

* Subclasses de `UIView`: Eles podem variam de um objeto de `UIView` simples a controles complexos e personalizados de UI;
* Classes que são parte dos frameworks UIKit (ou AppKit);
* Classes que envolvam o trabalho com _Core Animation_ e _Core Graphics_.

### Objetos de Controlador ligam o Modelo à _View_

Um objeto controlador atua como agente intermediário entre os objetos de _view_ do aplicativo e seus objetos de modelo. Os controladores geralmente são responsáveis ​​por garantir que as _views_ tenham acesso aos objetos de modelo que precisam exibir e agem como o canal pelo qual as _views_ aprendem sobre as alterações no modelo. Os objetos do controlador também podem executar tarefas de configuração e coordenação para um aplicativo e gerenciar os ciclos de vida de outros objetos.

Em um design típico do MVC, quando os usuários inserem um valor ou indicam uma escolha por meio de um objeto de _view_, esse valor ou escolha é comunicado a um objeto controlador. O objeto controlador pode interpretar a entrada do usuário de alguma maneira específica do aplicativo e, em seguida, pode dizer a um objeto do modelo o que fazer com essa entrada - por exemplo, "adicionar um novo valor" ou "excluir o registro atual" - ou pode dizer ao objeto de modelo para refletir um valor alterado em uma de suas propriedades. Com base nessa mesma entrada do usuário, alguns objetos do controlador também podem dizer a um objeto de _view_ para alterar um aspecto de sua aparência ou comportamento, como dizer a um botão para desabilitar a si mesmo. Por outro lado, quando um objeto de modelo é alterado - digamos, uma nova fonte de dados é acessada - o objeto de modelo geralmente comunica essa alteração a um objeto de controlador, que então solicita que um ou mais objetos de _view_ se atualizem adequadamente.

Os objetos do controlador podem ser reutilizáveis ​​ou não reutilizáveis, dependendo de seu tipo geral.

### Combinando papéis

Pode-se mesclar os papéis do MVC desempenhados por um objeto, fazendo com que um objeto, por exemplo, cumpra as funções de controlador e de _view_ - nesse caso, ele seria chamado de controlador de visualização, ou _View Controller_. Da mesma forma, você também pode ter objetos model-controller. Para alguns aplicativos, combinar funções como essa é um design aceitável.

Um model-controller é um controlador que se preocupa principalmente com a camada de modelo. Ele “possui” o modelo; suas principais responsabilidades são gerenciar o modelo e comunicar-se com os objetos de _view_. Os métodos de ação que se aplicam ao modelo como um todo são normalmente implementados em um model-controller.

Um controlador de visualização é um controlador que se preocupa principalmente com a camada de visualização. Ele “possui” a interface (as _views_); suas principais responsabilidades são gerenciar a interface e comunicar-se com o modelo. Os métodos de ação relacionados aos dados exibidos em uma exibição são normalmente implementados em um _view controller_. Um objeto UIAlertController (ou qualquer especialização de UIViewController) é um exemplo de controlador de visualização.

A seção [Diretrizes de design para aplicações MVC](#diretrizes-de-design-para-aplicações-mvc) oferece alguns conselhos de design sobre objetos com papéis do MVC combinados.

## Tipos de objetos de controlador

A seção [Objetos de controlador ligam o modelo à view](#objetos-de-controlador-ligam-o-modelo-à-view) esboça a definição abstrata de um objeto do controlador, mas na prática a ideia pode ser mais complexa. Existem algumas possíveis implementações para tipos gerais de objetos controladores, e cada implementação pode estar associada a um conjunto diferente de vantagens e desvantagens do ponto de vista da aplicação.

Atualmente, no desenvolvimento de aplicações iOS, a estratégia mais usual para a implementação de objetos de controlador é a de *mediação* (estratégia herdada pelo subclassing de `UIViewController`, por exemplo). Objetos de controlador de mediação facilitam — ou mediam — o fluxo de dados entre objetos de _view_ e objetos de modelo.

Os controladores de mediação geralmente são objetos prontos que você arrasta da biblioteca do Interface Builder. Você pode configurar esses objetos para estabelecer as conexões entre as propriedades dos objetos de _view_ e as propriedades do objeto do controlador e, em seguida, entre essas propriedades do controlador e as propriedades específicas de um objeto de modelo. Como resultado, quando os usuários alteram um valor exibido em um objeto de _view_, o novo valor pode ser comunicado automaticamente a um objeto de modelo para armazenamento — por meio do controlador de mediação; e quando uma propriedade de um modelo altera seu valor, essa alteração é comunicada a uma _view_ para exibição. A classe UIViewController e suas subclasses fornecem recursos de suporte, como a capacidade de confirmar e descartar alterações e o gerenciamento de seleções e valores.

## MVC como um padrão de projeto composto

Model-View-Controller é um padrão de projeto que é composto de vários padrões de projeto mais básicos. Esses padrões básicos trabalham juntos para definir a separação funcional e os caminhos de comunicação que são característicos de uma aplicação MVC. No entanto, a noção tradicional de MVC atribui um conjunto de padrões básicos diferentes daqueles que a implementação padrão na plataforma iOS atribui. A diferença está principalmente nas funções atribuídas ao controlador e aos objetos de _view_ de um aplicativo.

Na concepção original (Smalltalk), o MVC é composto pelos padrões _Composite_, _Strategy_ e _Observer_.

* Composite: Os objetos de _view_ em um aplicativo são, na verdade, uma composição de _views_ aninhadas que funcionam juntas de maneira coordenada (ou seja, em uma hierarquia de visualizações). Esses componentes de _view_ vão desde de uma janela inteira com uma composição complexa de _views_, como uma _Table View_, até _views_ individuais, como botões. As entradas do usuário e a exibição podem ocorrer em qualquer nível da estrutura hierárquica.

* Strategy: O objeto controlador implementa a estratégia para um ou mais objetos de _view_. O objeto de _view_ se limita a manter seus aspectos visuais encapsulados e delega ao controlador todas as decisões sobre o comportamento da interface (as regras de visualização que dão sentido ao aplicativo).

* Observer: Um objeto de modelo mantém objetos do aplicativo interessados ​​— geralmente objetos de _view_ — avisados ​​sobre as mudanças em seu estado.

A forma tradicional como os padrões _Composite_, _Strategy_ e _Observer_ trabalham juntos é representada pela imagem abaixo: O usuário manipula uma _view_ em algum nível da estrutura hierárquica e, como resultado, um evento é gerado. Um objeto controlador recebe o evento e o interpreta de uma maneira específica para a aplicação, ou seja, aplica uma estratégia. Essa estratégia pode ser solicitar a um objeto de modelo (via mensagem por invocação de função) que altere seu estado ou mesmo solicitar que um objeto de _view_ (em algum nível da estrutura composta) altere seu comportamento ou aparência. O objeto modelo, por sua vez, notifica todos os objetos que se registraram como observadores quando seu estado muda; se o observador for um objeto de _view_, ele poderá atualizar sua aparência de acordo.

<p align="center">
<img alt="Imagem com diagrama contendo os componentes do padrão mvc e suas relações segundo a implementação tradicional" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-teoria-imagem-mvc-composto.jpg?raw=true" width="75%"/>
</p>

A versão do MVC como padrão composto segundo a visão da plataforma iOS tem algumas semelhanças com a versão tradicional e, de fato, é bem possível construir um aplicativo funcional baseado no diagrama apresentado acima. Usando algumas técnicas para fazer o _binding_, você pode criar um aplicativo MVC cujas _views_ observam diretamente os objetos do modelo para receber notificações de alterações de estado. No entanto, há um problema teórico com este projeto. Objetos de _view_ e objetos de modelo devem ser os objetos mais reutilizáveis ​​em um aplicativo. Objetos de _view_ representam a "aparência" de um sistema operacional e os aplicativos que o sistema suporta; consistência na aparência e no comportamento é essencial, e isso requer objetos altamente reutilizáveis. Objetos de modelo, por definição, encapsulam os dados associados a um domínio de problema e executam operações sobre esses dados. Em termos de design, é melhor manter os objetos de modelo e de exibição separados uns dos outros, pois isso aumenta sua capacidade de reutilização.

Na maioria dos aplicativos iOS, as notificações de mudanças de estado em objetos de modelo são comunicadas aos objetos de _view_ por meio do controlador. A figura abaixo mostra essa configuração diferente, que parece mais limpa apesar do envolvimento de mais dois padrões de projeto básicos.

<p align="center">
<img alt="Imagem com diagrama contendo os componentes do padrão mvc e suas relações segundo a implementação da estratégia mediador da plataforma iOS" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-teoria-mvc-mediador.jpg?raw=true" width="75%"/>
</p>


O objeto controlador nesta implementação composta incorpora o padrão _Mediator_, bem como o padrão _Strategy_; ele medeia o fluxo de dados entre objetos de modelo e _view_ em ambas as direções. As alterações no estado do modelo são comunicadas aos objetos de _view_ por meio dos objetos do controlador de um aplicativo. Além disso, os objetos de _view_ incorporam o padrão `Command` por meio de sua implementação do mecanismo _target-action_.

## Diretrizes de design para aplicações MVC

As diretrizes a seguir se aplicam às considerações do padrão Model-View-Controller no design de aplicativos:

* Embora você possa combinar funções MVC em um objeto, a melhor estratégia geral é manter a separação entre os papéis. Essa separação aumenta a reutilização de objetos e a extensibilidade do software em que eles são usados.

* O objetivo de uma aplicação MVC bem projetado deve ser usar tantos objetos quanto possível que sejam (teoricamente, pelo menos) reutilizáveis. Em particular, objetos de _view_ e objetos de modelo devem ser altamente reutilizáveis. (Os objetos controladores de mediação cujas implementações são fornecidas pelo UIKit framework, é claro, são reutilizáveis). O comportamento específico do aplicativo é frequentemente concentrado o máximo possível em objetos controladores.

* Embora seja possível fazer com que as _views_ observem diretamente os modelos para detectar mudanças no estado, é melhor não fazê-lo. Um objeto de _view_ deve sempre passar por um objeto controlador de mediação para aprender sobre as alterações em um objeto de modelo. As razões são:
    
    * Se você usar o _binding_ para que os objetos de _view_ observem diretamente as propriedades dos objetos de modelo, você ignorará todas as vantagens que o UIViewController e suas subclasses oferecem ao seu aplicativo.

    * Se você não usar algum mecanismo de binding entre o modelo e a _view_, precisará criar uma subclasse de uma classe de _view_ existente para adicionar a capacidade de observar as notificações de alteração postadas por um objeto de modelo.

* Se esforce para limitar dependências no código das classes de seu aplicativo. Quanto maior o número de dependências que uma classe tem de outras classes, menos reutilizável ela tende a ser. As recomendações específicas variam de acordo com os papéis do MVC das duas classes envolvidas:

    * Uma _view_ não deve depender de uma classe de modelo (embora isso possa ser inevitável com algumas exibições personalizadas).
    
    * Uma _view_ não deve depender de uma classe de controlador de mediação.

    * Uma classe de modelo não deve depender de nada além de outras classes de modelo.

    * Um controlador não deve depender de uma classe de modelo (embora, como as visualizações, isso possa ser necessário se for uma classe de controlador personalizada). OBS: Depender de abstrações definidas em termos de coisas *apresentáveis ao usuário* poderia ser um caminho imaginável, embora não seja muito usual.

* Se o UIKit framework oferece uma arquitetura que resolve um problema de programação e essa arquitetura atribui papéis MVC a objetos de tipos específicos, use essa arquitetura. Será muito mais fácil construir seu projeto dessa maneira. Em outras palavras, não brigue com seu framework. (A menos que em determinadas situações essa briga seja por uma boa causa 🫢)

## Uma aplicação prática da separação de papéis

Considere uma aplicação simples como a apresentada pela figura abaixo. Ela sugere uma simulação de um jogar de dados, que entre suas especificações de negócio, requer: (I) que a aplicação ofereça um controle ao usuário para que seja acionado o jogar de um dado virtual; (II) que ao acionar o controle a aplicação sorteie um valor numérico randômico entre `1` e `6`; e (III) que o aplicativo informe o número corrente sorteado, mas que além disso mantenha o valor anterior apresentado ao usuário à título de comparação com o novo valor.

<p align="center">
<img alt="Imagem com modelo da tela para o app exemplo de jogo de dados" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-teoria-imagem-mvc-aplicacao-pratica-1.jpg?raw=true" width="75%"/>
</p>

Para tal aplicação temos as seguintes implementações de _view_ e controle, camadas com as quais já temos certa intimidade, como ponto de partida.

* Implementação para a _view_

    <p align="center">
    <img alt="Imagem com implementação da view para o exemplo no storyboard" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-teoria-imagem-mvc-aplicacao-pratica-2.png?raw=true" width="75%"/>
    </p>

* Implementação para o _controller_

    <p align="center">
    <img alt="Imagem com implementação do controller Swift para o exemplo" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-teoria-imagem-mvc-aplicacao-pratica-3.png?raw=true" width="75%"/>
    </p>

O estado atual do projeto com as devidas ligações entre os elementos da camada de visualização e da camada de controle já satisfaz o objetivo proposto pela aplicação. Entretanto, é possível fazer uma série de considerações sobre a separação de código do ponto de vista da aplicação de padrão arquitetural sugerido pela plataforma.

Considerando que as condições para o jogo de dados e comparação dos valores sorteados são definidos em termos do domínio do problema que a aplicação tende a resolver, é razoável pensar que os detalhes de implementação do jogo de dados como nas instruções em `valorDoTurnoAnterior = valorDoTurnoAtual` e `valorDoTurnoAtual = Int.random(in: 1...6)` (e a própria existência das propriedades armazenadas no controlador) poderiam ser destacadas da camada atual e isoladas na camada do modelo. A aplicação da técnica poderia trazer ganhos ao expressar de forma mais compreensível o domínio, privilegiar separação de responsabilidades e favorecer reuso e testabilidade.

O caso pode ainda ficar mais visível à medida que imaginemos outros casos de uso para o aplicativo, como no caso de haver a representação de turnos para o jogo em partidas e aferição de um ganhador, trazendo a ideia de dois ou mais _players_ para o domínio. Toda a implementação dessa lógica na camada de controle poderia representar sérios riscos para o entendimento e manutenção do código.

> Nota: Vale destacar que `valorDoTurnoAnterior` e `valorDoTurnoAtual` (e sua gestão) já representam os dados que dão sentido à existência do aplicativo, ou em outras palavras, já configuram seu modelo, mesmo que trabalhados através de tipos simples providos pelo Swift e sem uma separação visível desta camada. Se o domínio do problema contemplado for tão simples e fechado como o exemplo, é perfeitamente aceitável que a implementação permaneça da forma como está atualmente, desde que isso seja acompanhado por completo conhecimento dos trade-offs que ela possa oferecer. É importante ter em mente que a adoção de qualquer padrão de design ou arquitetura para um projeto deve ser feita de modo a resolver um problema, e para tanto, deve ser verificado como um problema real alguma situação da implementação. Aplicar padrões de design sem a mínima reflexão sobre ganhos e perdas, pode tornar mais difícil o processo de desenvolvimento e aumentar a complexidade do código produzido, com pouco resultado do outro lado.
>
> Isso posto, para a maior parte das aplicações simples ainda é aconselhável uma separação mínima de responsabilidades, e como objetivo do material teórico, faremos isso pela perspectiva do padrão arquitetural MVC.

Podemos construir um tipo customizado para representar o jogo de dados, atingindo uma separação mais clara para a camada de modelo.

``` swift

// JogoDeDados.swift

import Foundation

fileprivate class Dado {
    static func joga() -> Int {
        return Int.random(in: 1...6)
    }
}

struct JogoDeDados {
    private(set) var valorDoTurnoAnterior: Int = 0
    private(set) var valorDoTurnoAtual: Int = 0
    
    mutating func executa() {
        valorDoTurnoAnterior = valorDoTurnoAtual
        valorDoTurnoAtual = Dado.joga()
    }
}
```

Com a classe de modelo acima, o código para o controlador pode ser atualizado para o que segue abaixo.

``` swift

import UIKit

class JogoDeDadosViewController: UIViewController {

    @IBOutlet weak var valorAnteriorLabel: UILabel!
    @IBOutlet weak var valorAtualLabel: UILabel!
    
    var jogo: JogoDeDados = .init() {
        didSet {
            atualizaView()
        }
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        atualizaView()
    }

    @IBAction func botaoPlayPressionado(_ sender: UIButton) {
        jogo.executa()
    }
    
    func atualizaView() {
        valorAnteriorLabel.text = String(describing: jogo.valorDoTurnoAnterior)
        valorAtualLabel.text = String(describing: jogo.valorDoTurnoAtual)
    }
}

```

A implementação atual demonstra com clareza a distinção dos papéis do padrão arquitetural MVC e o posição do controlador como um agente mediador entre as atualizações necessárias nas camadas de _view_ e modelo. Os objetos de _view_ (UILabel, no exemplo) são por natureza totalmente reutilizáveis, assim como a representação de modelo construída.

> Nota: A implementação para o controlador acima pode sugerir uma dependência do controlador para com o modelo. Por mais necessária e inevitável que ela seja, em certas situações se faz necessário suavizar a forma como a dependência é resolvida. Veremos no material teórico Injeção de Dependências via Contrutores e Propriedades desta seção, formas de gerir esse acoplamento entre módulos.
