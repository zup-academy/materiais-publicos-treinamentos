# UIStepper

Um controle para incrementar ou decrementar um valor.

> Esse material foi traduzido e adaptado da fonte original em https://developer.apple.com/documentation/uikit/uistepper

## Declaração 

``` swift
@MainActor class UIStepper : UIControl
```

## Visão Geral

Por padrão, pressionar e segurar o botão de um stepper aumenta ou diminui o valor do stepper repetidamente. A taxa de mudança depende de quanto tempo o usuário continua pressionando o controle. Para desativar esse comportamento, defina a propriedade [autorrepeat](https://developer.apple.com/documentation/uikit/uistepper/1624079-autorepeat) como `false`.

O valor máximo deve ser maior ou igual ao valor mínimo. Se você definir um valor máximo ou mínimo que quebraria essa invariante, ambos os valores serão definidos para o novo valor. Por exemplo, se o valor mínimo for 200 e você definir um valor máximo de 100, o mínimo e o máximo se tornarão 200.

> Importante: UIStepper não está disponível quando o idioma de interface do usuário é `UIUserInterfaceIdiom.mac`.

## Exemplo de utilização básico

1. Em um projeto exemplo, contendo um label para um valor numérico em seu arquivo `Main.storyboard`, junto à biblioteca de objetos do Interface Builder, adicione um novo objeto arrastando para o canvas um novo elemento UIStepper.

1. Através do _Attributes Inspector_, define valores para algumas propriedades como: `value`, `minimum`, `maximum` e `step`.

    <p align="center">
    <img alt="Animação com a demonstração da adição e configuração do objeto" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-teoria-doc-uistepper-1.gif?raw=true" width="75%" />
    </p>

1. No controller associado à tela, adicione _outlets_ para obter as referências do _label_ e do _stepper_ e uma _action_ para o evento padrão _value changed_.

    <p align="center">
    <img alt="Animação com a demonstração da conexão dos outlets e action" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/mvc-teoria-doc-uistepper-2.gif?raw=true" width="75%" />
    </p>

1. Adicione código para prover uma função cuja implementação atualize a view para refletir uma alteração do valor numérico

    ``` swift
        func atualizaView(para valor: Int) {
            valorLabel.text = String(describing: valor)
            stepper.value = Double(valor)
        }
    ```

1. Adicione uma propriedade armazenada chamada `valor` para conter um numero inteiro: `private var valor: Int = 1`. Em seguida, adicione um observador `didSet` que invoque a função `atualizaView(valor:)` para o novo valor, além de garantir um chamada inicial para `atualizaView(valor:)` no corpo da função `viewDidLoad()`.

    <gif-3>

    ``` swift
        private var valor: Int = 1 {
            didSet {
                atualizaView(para: valor)
            }
        }

        override func viewDidLoad() {
            super.viewDidLoad()
        
            // Do any additional setup after loading the view.
            atualizaView(para: valor)
        }    
    ```

1. Na função conectada através de `@IBAction`, adicione implementação para alterar o valor numérico do _label_ apresentando o novo valor selecionado no _stepper_.

    ``` swift
        @IBAction func valorStepperMudou(_ sender: UIStepper) {
            valor = Int(sender.value)
        }
    ```

    <gif-4>

1. Teste a aplicação exemplo e perceba seu funcionamento.

    <gif-5>

1. Adicionalmente, consulte o item [Tópicos](#tópicos) abaixo e pesquise sobre outras propriedades e método úteis para um ajuste fino de sua funcionalidade. Teste as possibilidades.

## Tópicos

### Configurando o Stepper

``` swift 
var isContinuous: Bool
```
Um valor booleano que determina se as alterações de valor devem ser enviadas durante a interação do usuário ou após o término da interação do usuário.

``` swift 
var autorepeat: Bool
```
Um valor booleano que determina se o valor do stepper deve ser alterado repetidamente à medida que o usuário pressiona e mantém pressionado um botão de stepper.

``` swift 
var wraps: Bool
```
Um valor booleano que determina se o stepper pode reiniciar a contagem ao decrementar ou incrementar através dos limites.

``` swift 
var minimumValue: Double
```
O menor valor numérico possível para o _stepper_.

``` swift 
var maximumValue: Double
```
O valor numérico mais alto possível para o _stepper_.

``` swift
var stepValue: Double
```
O valor dp passo, ou incremento, para o _stepper_.

### Acessando o valor do Stepper

``` swift
var value: Double
```
O valor numérico do stepper.

### Personalizando a aparência

``` swift
func backgroundImage(for: UIControl.State) -> UIImage?
```
Retorna a imagem de fundo associada ao estado especificado do controle.

``` swift
func setBackgroundImage(UIImage?, for: UIControl.State)
```
Define a imagem de fundo do controle quando ele está no estado especificado.

``` swift
func decrementImage(for: UIControl.State) -> UIImage?
```
Retorna a imagem usada para o símbolo de decremento do controle.

``` swift
func setDecrementImage(UIImage?, for: UIControl.State)
```
Define a imagem a ser usada para o símbolo de decremento do controle.

``` swift
func dividerImage(forLeftSegmentState: UIControl.State, rightSegmentState: UIControl.State) -> UIImage?
```
Retorna a imagem do divisor para a combinação fornecida de estados esquerdo e direito.

``` swift
func setDividerImage(UIImage?, forLeftSegmentState: UIControl.State, rightSegmentState: UIControl.State)
```
Define a imagem de divisor a ser usada para uma determinada combinação de estados esquerdo e direito.

``` swift
func incrementImage(for: UIControl.State) -> UIImage?

```
Retorna a imagem usada para o símbolo de incremento do controle.

``` swift
func setIncrementImage(UIImage?, for: UIControl.State)
```
Define a imagem a ser usada para o símbolo de incremento do controle.

## Relações

### Herda de

`UIControl`

