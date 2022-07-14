# Detalhes da instanciação de objetos a partir de Storyboards

Você já parou pra pensar sobre como conseguimos obter as referências para cada objeto cujas definições enconstram-se apenas nos arquivos storyboard? Para obter a referência de cada instância de uma classe precisamos explícitamente invocar um de seus inicializadores válidos, como por exemplo: `UIView()`, `Decimal(1)` ou `String(describing: Date.now)`. Como é então possível obter todos esses objetos de _view_ que declaramos ao desenhar nossa interface pelo Interface Builder?

Se você pensou "a implementação por trás do framework de UIKit deve fazer isso pra nós", você tem toda a razão. Mas é importante entender, ainda que não na totalidade, alguns detalhes sobre como esse mecanismo funciona. Assim, podemos tirar proveito de alguns momentos importantes do ciclo de vida de objetos construídos a partir do Interface Builder.

## Inicialização

Mesmo que ainda não tenhamos dado o devido foco para a construção de _views_ programaticamente, em alguns dos tópicos desse treino já foi possível ter algum contato com a prática. Nestes momentos, você provavelmente percebeu as definições de objetos de _view_ e sua configuração sem feita sem a necessidade do suporte do Interface Builder. 

Quanto à inicialização dos objetivos programaticamente e via Interface Builder (arquivos storyboard ou nib), já temos uma primeira diferença. Ao trabalhar programaticamente, na grande maioria das vezes, é comum utilizar um inicializador padrão para a classe desejada, e nestes casos, toda a configuração do estado inicial para o objeto fica por conta do seu código. Já ao utilizar o Interface Builder como mecanismo de design das _views_, prática comum neste treino, na verdade não temos controle sobre este momento do ciclo de vida dos objetos.

A implementação do UIKit _framework_ provê todo o código de _loading_ necessário para interpretar em tempo de execução a definição dos nossos objetos de _view_ definidos nos arquivos storyboard (que a nível de arquivo detém algo equivalente a conteúdo xml). Esse processo é iniciado através da identificação do tipo de objeto declarado e invocação de seu inicializador `init?(coder:)`. É assim, por exemplo, que são construídos todos os objetos que compõem a hierarquia de _views_ de um _view controller_.

Mas um ponto importe a se pensar é: trabalhando com orientação a objetos, e relembrando da utilidade de construtores que é configurar adequadamente o estado inicial para a instância, o que fazemos nos casos onde a definição presente no conteúdo do storyboard não for suficiente para configurar este estado?

Diversas são as situações onde isso pode ser necessário. Basta lembrar que nem todas as propriedades visuais de componentes são configuráveis através do _Attributes Inspector_ do Interface Builder.

## Ajustando o estado de uma view após a instanciação pelo Interface Builder

Para ter essa capacidade o UIKit _framework_ oferece a possibilidade de adicionar implementação após o momento de instanciação de _views_ definidas em _nib files_. Para isso, é possível sobrescrever o método `awakeFromNib()` de `NSObject`, que segundo sua própria documentação "prepara o objeto para serviço depois de ter sido carregado de um arquivo do Interface Builder (ou arquivo nib)".

É através deste método que é recomendável a configuração adicional de qualquer propriedade adicional ou execução de comportamento desejável antes do trabalho com a _view_ no contexto de sua utilização. Por exemplo, ainda nas atividades desta seção, em alguns momentos, você pode precisar configurar seus objetos de _table view cell_ para terem uma _backgroundView_ para o estado de seleção com uma cor customizada. Este nível de detalhe na customização dos objetos pode ser impossível de obter ao se apoio unicamente no suporte do Interface Builder.

O código abaixo exemplifica essa possibilidade.

``` swift
class MinhaCustomTableViewCell: UITableViewCell {

    // implementação anterior omitida
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        let bgColorView = UIView()
        bgColorView.backgroundColor = .systemRed.withAlphaComponent(0.3)
        selectedBackgroundView = bgColorView
    }
    
}
```

A seção abaixo traz a discussão sobre o papel do método pela visão do _framework_ na documentação oficial em https://developer.apple.com/documentation/objectivec/nsobject/1402907-awakefromnib.

## awakeFromNib() discussion

O código de infraestrutura para carregamento de arquivos _nib_ envia uma mensagem `awakeFromNib` para cada objeto recriado de um arquivo nib, mas somente depois que todos os objetos no arquivo forem carregados e inicializados. Quando um objeto recebe uma mensagem `awakeFromNib`, é garantido que todas as suas conexões _outlets_ e _actions_ já estejam estabelecidas.

Você deve chamar a superimplementação de `awakeFromNib`(`super.awakeFromNib()`) para dar às classes mãe a chance de realizar qualquer inicialização adicional necessária. Embora a implementação padrão desse método não faça nada, muitas classes do UIKit fornecem implementações não vazias. Você pode chamar a superimplementação a qualquer momento durante seu próprio método `awakeFromNib`.

Durante o processo de instanciação, cada objeto no arquivo é desarquivado e inicializado com o método adequado ao seu tipo. Objetos que estão em conformidade com o protocolo `NSCoding` (incluindo todas as subclasses de `UIView` e `UIViewController`) são inicializados usando seu método `initWithCoder:` (`init(coder:)` em Swift). Todos os objetos que não estão em conformidade com o protocolo `NSCoding` são inicializados usando seu método `init`. Depois que todos os objetos foram instanciados e inicializados, o código de carregamento de arquivos nib restabelece as conexões _outlets_ e _actions_ para todos esses objetos. Em seguida, ele chama o método `awakeFromNib` dos objetos.

Normalmente, você implementa o `awakeFromNib` para objetos que exigem configuração adicional que não pode ser feita em tempo de design. Por exemplo, você pode usar esse método para personalizar a configuração padrão de qualquer controle para corresponder às preferências do usuário ou aos valores em outros controles.

> Nota: Você pode obter um nível maior de detalhes através da documentação oficial em https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/LoadingResources/CocoaNibs/CocoaNibs.html#//apple_ref/doc/uid/10000051i-CH4-SW8
>
> Importante notar que toda a infraestrutura para o trabalho com arquivos do Interface Builder remonta às implementações de base desenvolvidas com Objective-C.