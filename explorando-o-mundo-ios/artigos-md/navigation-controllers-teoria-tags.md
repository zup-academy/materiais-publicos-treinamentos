# Utilizando tags para identificar view objects

A grande maioria das UIs de aplicativos oferece controles a partir dos quais o usuário pode interagir. Esses controles podem ser simples botões ou mesmo sliders, switches, steppers, entre uma série de outras possibilidades incluindo a criação de controler customizados. É através da interação do usuário para com estes diversos tipos de `UIControl` que a aplicação consegue coletar informações importantes sobre as preferências do usuário e executar suas funcionalidades.

Para que seja possível responder às interações é possível utilizar uma série de mecanismos entre os quais podemos citar: _IBActions_ para responder aos eventos naturais, _UIStoryboard segues_ para responder executando uma transição em um fluxo de navegação, ou mesmo a implementação de protocolos _delegate_ para prover implementação para responder a eventos chave do ciclo de vida de diferentes componentes. Na maioria dos casos, é comum que a relação entre o controle e sua implementação de resposta a um determinado evento seja de "um pra um", isto é, cada elemento de view (control) tem uma responsabilidade distinta dos demais objetos em um interface, e para isso existe uma única implementação dedicada para a lógica de resposta. No entanto, nem todos os casos são assim.

Existem alguns tipos de UI específicos onde, para prover uma melhor experiência ao usuário, são utilizados múltiplos controles respondendo à mesma função, ou lógica. Idealize uma tela com uma coleção de botões para facilitar a seleção de uma opção para um mesmo recurso, por exemplo, a tela daquele seu aplicativo favorito de entrega de comida, onde você pode optar por dar uma gorjeta ao entregador. A figura abaixo serve de ilustração para o exemplo.

![Imagem com exemplo de app de entrega de comida contendo um grupo de botões para selecionar uma gorjeta ao entregador. Existem quatro botões na tela: um para uma gorjeta de 10%, outro para 15%, um para 20% e mais um para 25%](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/navigation-controllers-teoria-tags-imagem-exemplo-app-com-colecao-de-botoes.jpg?raw=true)

Imagine que para essa tela devemos construir uma lógica de resposta para a seleção do valor de gorjeta. Como seria possível implementar com o que sabemos até aqui?

## O problema

A primeira hipótese de solução poderia supor a construção de _IBActions_ para cada botão e então escrever o código para a lógica. De certo conseguiríamos propor uma implementação funcional, mas vamos analisar um possível código.

``` swift
import UIKit

class CarinhoViewController: UIViewController {

    // código anterior omitido

    // conectada ao botão 10% através do Interface Builder
    @IBAction func adicionaGorjeta10PorCentoAction(_ sender: UIButton) {
        // lógica para 10% aqui
    }
    
    // conectada ao botão 15% através do Interface Builder
    @IBAction func adicionaGorjeta15PorCentoAction(_ sender: UIButton) {
        // lógica para 15% aqui
    }

    // conectada ao botão 20% através do Interface Builder
    @IBAction func adicionaGorjeta20PorCentoAction(_ sender: UIButton) {
        // lógica para 20% aqui
    }

    // conectada ao botão 25% através do Interface Builder
    @IBAction func adicionaGorjeta25PorCentoAction(_ sender: UIButton) {
        // lógica para 25% aqui
    }

}
```

Esse snippet de código pode revelar um risco de repetição da lógica de resposta. Para impedir isso poderíamos pensar na construção de um método que isole essa responsabilidade de resposta.

``` swift
import UIKit

class CarinhoViewController: UIViewController {

    // código anterior omitido

    // função que implementa a lógica de resposta à adição de gorjeta
    func adiciona(_ gorjeta: Gorjeta) {
        // código aqui
    }

    // conectada ao botão 10% através do Interface Builder
    @IBAction func adicionaGorjeta10PorCentoAction(_ sender: UIButton) {
        // lógica para 10% aqui
        adiciona(Gorjeta.dezPorCento)
    }
    
    // conectada ao botão 15% através do Interface Builder
    @IBAction func adicionaGorjeta15PorCentoAction(_ sender: UIButton) {
        // lógica para 15% aqui
        adiciona(Gorjeta.quinzePorCento)
    }

    // conectada ao botão 20% através do Interface Builder
    @IBAction func adicionaGorjeta20PorCentoAction(_ sender: UIButton) {
        // lógica para 20% aqui
        adiciona(Gorjeta.vintePorCento)
    }

    // conectada ao botão 25% através do Interface Builder
    @IBAction func adicionaGorjeta25PorCentoAction(_ sender: UIButton) {
        // lógica para 25% aqui
        adiciona(Gorjeta.vinteECincoPorCento)
    }

}
```

Mas algo fica ainda estranho com o código. A carga cognitiva que o código exerce pode crescer de forma indefinida e dificultar o entendimento com o tempo. Pensando na lógica de resposta facilmente podemos perceber que ele é a mesma para todo botão do grupo. E faz sentido! Usar o grupo de botões para a interface foi uma decisão de UX mas isso não precisa pressionar o nosso código. Conceitualmente, independentemente do botão pressionado estamos executando **uma** lógica, a mesma lógica, e para tanto é natural supor que deveríamos conter um método de resposta.

## Uma possível solução: tags

O UIKit _framework_ oferece um recurso comum para qualquer `UIView` que pode ajudar em situações como essa. Trata-se da propriedade `tag` das _views_, que segundo a própria documentação de `UIView`, é um número inteiro que você pode usar para identificar objetos de _view_ em seu aplicativo. Definir um valor  customizado para esse atributo tem o efeito de colocar uma etiqueta para uma determinada `UIView`. Podemos tirar proveito disso.

> Nota: O uso de _tags_ deve ser feito com parcimônia. Seu uso deve ser feito no intuito de definir um identificador para o objeto. Não o utilize como meio de guardar dados ou rastrear o estado da aplicação.

A animação abaixo ilustra a definição de tags para objetos através do Interface Builder.

<p align="center">
<img alt="Animação mostrando como selecionar um elemento de view no Interface Builder e definir um valor inteiro identificador através do campo para a propriedade tag no Attributes Inspector" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/navigation-controllers-teoria-tags-gif-adicionando-tags.gif?raw=true" width="80%"/>
</p>

Uma vez que tenhamos identificadores para os elementos, é possível associar os identificadores às opções adequadas para o conceito modelado de gorjeta, ou por qualquer implementação que se julgue mais simples para o caso. Neste exemplo teórico estamos utilizando uma _enum_ `Gorjeta` para expressar as opções.

``` swift
enum Gorjeta: Int, CaseIterable {
    case dezPorCento = 1
    case quizePorCento = 2
    case vintePorCento = 3
    case vinteECincoPorCento = 4
    
    var valor: Decimal {
        switch self {
        case .dezPorCento:
            return 0.1
        case .quizePorCento:
            return 0.15
        case .vintePorCento:
            return 0.2
        case .vinteECincoPorCento:
            return 0.25
        }
    }
}
```

Note que fizemos da _enum_ um tipo passível de ser representado a partir de um valor base inteiro _(raw representable)_ para fins de facilidade de implementação. Assim, ganhamos a possibilidade de acessar o valor através da propriedade `rawValue`, além de ganhar um inicializador implícito que pode nos ajudar a construir uma instância deste tipo passando este inteiro. Também fizemos com que o tipo entre em conformidade com o protocolo `CaseIterable` o que garante que o tipo terá um propriedade `allCases` que guarda um array com valores para todos os casos da _enum_ (isso também vai nos ajudar facilitando o código).

Com a representação modelada e as tags configuradas, já temos tudo o que é necessário pra reduzir aquela carga de código antes presente no _view controller_ que gerencia os botões. Podemos reduzir o número de action para um. A animação abaixo ilustra como podemos criar actions e adicionar multiplos objetos como _senders_ (acionadores do evento desejado).

<p align="center">
<img alt="Animação mostrando como adicionar uma ib action e depois configurar multiplos senders para a mesma ação através do marcador de conexão da ib action no editor de código. O especialista usa a referência visual de ponto de conexão para arrastar uma seta a cada novo elemento que se deseja relacionar à mesma ação. Ao final da animação é possível perceber os quatro botões com as opções de gorjeta sendo relacionados com a única função de lógica relacionada." src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/navigation-controllers-teoria-tags-gif-action-com-multiplos-senders.gif?raw=true" width="80%"/>
</p>

Com todo o grupo de botões relacionado à _IB Action_ criada conseguimos responder com a lógica através de uma só referência de função. Podemos então acessar a propriedade `tag` do `UIButton`, cuja referência recemos como parâmetro na função (nomeado como `sender`), para descobrir qual foi a seleção do usuário com o código do exemplo abaixo:

``` swift
class CarinhoViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }

    @IBAction func adicionaGorjeta(_ sender: UIButton) {
        guard let gorjetaSelecionada = Gorjeta.allCases.filter({ $0.rawValue == sender.tag }).first else {
            fatalError("Não foi possível determinar a opção para a gorjeta.")
        }
        
        print(gorjetaSelecionada)
        
        // lógica para adição da gorjeta aqui
        // carrinho.adiciona(gorjetaSelecionada)
    }

}
```

A animação abaixo ilustra o funcionamento da implementação aplicada.

<p align="center">
<img alt="Animação mostrando como a action funcionando para todo o grupo de botões. O especialista clica na opção de gorjeta e o nome do caso adequado para a gorjeta aparece na saída padrão do sistema" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/navigation-controllers-teoria-tags-gif-action-funcionando.gif?raw=true" width="80%"/>
</p>

## Outras possíveis aplicações

O exemplo deste material teórico trata de situações onde conseguimos minimizar a carga de código necessária ao lidar com _IB Actions_. Mas como a própria introdução deste texto menciona é possível que essa estratégia te ajude em outras situações. Um exemplo pode ser ao planejar transições/navegações com _segues_ através do Interface Builder. Imagine a situação hipotética onde, ao selecionar uma gorjeta com um valor, o aplicativo apresentasse outra cena com transição modal com uma mensagem com o agradecimento pelos R$ XX,xx (valor exato adicionado como gorjeta para o pedido). O exemplo talvez não seja muito usual, mas para fins didáticos vai servir.

A esta altura você já sabe como adicionar referências de _segues_ para planejar transições entre cenas no storyboard (se não sabe consulte o material teórico de navigation controllers). Você então deve estar pensando num problema parecido como vimos acima. Seria preciso adicionar uma série de _segues_, com suas respectivas identificações, e preparar-se para a transição (no `prepare(for:sender:)` do _view controller_) passando as informações corretas para a próxima cena em cada caso específico verificado. 

Caso você tenha de fato associado este tipo de solução, não precisamos nem ir a isso. O código tenderia a ficar muito verboso, assim como o exemplo com as várias _actions_. 

Assim como fizemos no exemplo anterior, podemos reduzir também o problema a uma única _segue_ (e transição), o que, de fato, faz sentido ao pensar que independentemente da escolha, a navegação para a tela com o agradecimento é essencialmente a mesma coisa. Poderíamos endereçar a solução com um código tão simples quanto, como o exemplo abaixo nos mostra.

``` swift
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        // identificador único para quatro segues diferentes configuradas através do Interface Builder
        guard segue.identifier == "verAgradecimentoGorjeta" else { return }
        
        // certificando-se de ter todo o necessário para executar a transição
        guard let botaoGorjeta = sender as? UIButton,
              let controller = segue.destination as? AgradecimentoGorjetaViewController else {
            fatalError("Não foi possível executar segue \(segue.identifier!)")
        }
        
        // obtendo a escolha exata para a opção de gorjeta
        guard let gorjetaSelecionada = Gorjeta.allCases
                .filter({ $0.rawValue == botaoGorjeta.tag }).first else {
            fatalError("Não foi possível determinar a gorjeta selecionada ao processar a segue \(segue.identifier!)")
        }
        
        // possível configuração do estado do controlador que será apresentado
        controller.subtotal = carrinho.subtotal
        controller.gorjeta = gorjetaSelecionada
    }
```
