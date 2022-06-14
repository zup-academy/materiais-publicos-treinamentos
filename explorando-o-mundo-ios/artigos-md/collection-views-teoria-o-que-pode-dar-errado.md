# Collection Views: O que pode dar errado?

Neste material teórico são apresentados possíveis problemas que podem ocorrer no cálculo do layout ao utilizar UICollectionViews em situações específicas e como podemos corrigí-los.

## O código certo, no momento certo

Em certos momentos nos deparamos com especificações de design que, num primeiro momento não se mostram muito desafiadoras, mas que ao desenvolver código no sentido de atingí-la, podem requerer um entendimento maior de detalhes de funcionamento do framework de UI utilizado. É o caso do exemplo apresentado a seguir e que é também tema de um dos desafios da seção do treino.

<p align="center">
<img alt="Imagem contendo um exemplo de layout com uma coleção de raças de cachorros cada um com sua imagem com shape arredondado e com o nome da raça logo abaixo" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/collection-views-teoria-o-que-pode-dar-errado-exemplo-de-layout.jpg?raw=true" width="100%" />
</p>

Temos uma coleção organizada de valores para raças de cachorro. O destaque fica por conta do shape das imagens das raças. Temos uma identidade visual arredondada. Como mencionado anteriormente, pode não parecer algum problema, afinal basta a aplicação de `cornerRadius` na camada (_layer_) para a ImageView com o valor adequado e atingimos o resultado.

Mas qual o valor adequado para a propriedade? Sabemos que, para que consigamos este efeito é preciso que o valor para `cornerRadius`, considerando que o frame da ImageView seja um quadrado, seja equivalente a metade da largura (ou mesmo altura) deste elemento - ou mais precisamente igual ao raio da circunferência inscrita no quadrado que compreende este frame.

<p align="center">
<img alt="Imagem contendo o modelo visual da circunferência inscrita no quadrado do frame de cada imageView" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/collection-views-teoria-o-que-pode-dar-errado-shape-explicado.jpg?raw=true" width="40%" />
</p>

Podemos então calcular este valor e, através do _Identity Inspector_ do Interface Buider, definir um _Runtime Attribute_ de `layer.cornerRadius`, como ja fizemos algumas vezes antes. O problema dessa abordagem fica por conta da falta de flexibilidade da solução. Como chegaríamos ao valor adequado para um item? A partir da Interface Builder, apenas trabalhamos sobre um aspecto de largura, que em runtime, dados os diversos dispositivos e suas características de tela, poderia mudar.

É preciso configurar esta propriedade a partir de um momento mais preciso do nosso código, onde as dimensões para o objeto de layout da _Collection View_ e o tamanho para os itens (células) já tenham sido calculadas.

### Noções sobre o ciclo de vida de uma UIView

Antes de adiantar a discussão sobre ciclo de vida é importante fazermos uma revisão sobre o conceit fundamental de uma UIView.

> Nota: Este trecho do documento foi adaptado da documentação original em https://developer.apple.com/documentation/uikit/uiview

#### UIView: Um objeto que gerencia o conteúdo de uma área retangular na tela

As _views_ são os blocos de construção fundamentais da UI do seu aplicativo, e a classe `UIView` define os comportamentos comuns a todas elas. Um objeto de _view_ renderiza o conteúdo dentro de seu retângulo de limites (_bounds_) e trata de quaisquer interações com esse conteúdo. 

A classe `UIView` é uma classe concreta que você pode instanciar e usar para exibir um background e conteúdo fixos. Você também pode sobrescrevê-la para desenhar algum conteúdo mais customizado, quando utilizar suas próprias subclasses providas pelo UIKit - que é sempre recomendável - não for suficiente.

Algumas das principais responsabilidades de objetos de _view_ são:

* Desenho e animação
    * As _views_ desenham conteúdo em seu retangulo usando UIKit ou Core Graphics.
    * Você pode animar algumas propriedades de visualização para novos valores.

* Layout e gerenciamento de _subviews_
    * As _views_ podem conter zero ou mais _subviews_.
    * As _views_ podem ajustar o tamanho e a posição de suas _subviews_.
    * Use o Auto Layout para definir as regras para redimensionamento e reposicionamento de suas _views_ em resposta a alterações na hierarquia de _views_.

* Manipulação de eventos
    * Uma _view_ é uma subclasse de `UIResponder` e pode responder a toques e outros tipos de eventos.
    * As _views_ podem instalar _gesture recognizers_ para lidar com gestos comuns.

As _views_ podem ser aninhadas dentro de outras _views_ para criar hierarquias de _views_, que oferecem uma maneira adequada de organizar conteúdo relacionado. 

As propriedades `frame` e `bounds` definem a geometria de cada _view_. A propriedade `frame` define a origem e as dimensões da _view_ no sistema de coordenadas de sua _superview_. Já a propriedade _bounds_ define as dimensões internas da _view_ como ela mesma as vê, e seu uso é voltado para código que orienta a renderização de sua própria perspectiva.

#### Criando uma _view_ 

Normalmente, você cria _views_ em seus storyboards arrastando-os da biblioteca de componentes para o canvas. Mas você também pode criar _views_ programaticamente. 

Ao criar uma _view_, você normalmente especifica seu tamanho inicial e posição em relação à sua _superview_ futura. Por exemplo, o código a seguir cria uma _view_ e coloca seu canto superior esquerdo no ponto (`10`, `10`) do sistema de coordenadas da sua _superview_ (quando da sua adição a esta _superview_).

``` swift
let rect = CGRect(x: 10, y: 10, width: 100, height: 100)
let view = UIView(frame: rect)
```

Para adicionar uma _subview_ a outra _view_, basta chamar o método `addSubview(_:)` na _superview_.

Depois de criar uma _view_, crie regras de Auto Layout para controlar como o tamanho e a posição da exibição mudam em resposta às alterações no restante da hierarquia de _views_.

### Ciclo de vida de UIView

Todo objeto em um sistema orientado a objetos possui um ciclo de vida, que compreende da sua construção, passando por todas as transições de estados programadas, até o momento onde a informação que detém não se faz mais necessária e é então desalocado. 

Alguns objetos, como os dos frameworks base das plataformas, são tão centrais para o design das aplicações que nesse sentido se destacam e têm seu ciclo de vida guiando boa parte do funcionamento de uma aplicação. Esse é o caso dos objetos de `UIView`, entre outros objetos fundamentais, do framework UIKit. Conhecer seu funcionamento em mais detalhes pode diferenciar entre escrever códigos precisos na resolução de problemas específicos ou tender a implementações que ofereçam problemas para o sistema por alguma interpretação equivocada.

Conheceremos mais detalhes sobre o ciclo de vida das _UIViews_ através da separação de três grandes fases que podem se repetir como um ciclo durante a utilização de uma _view_. 

<p align="center">
<img alt="Imagem contendo as três fases do ciclo de vida de uma UIView: Update, Layout, Render" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/collection-views-teoria-o-que-pode-dar-errado-ciclo-de-vida-view.jpeg?raw=true" width="70%" />
</p>

Toda `UIView` passa por estes três grandes momentos após sua alocação: _Update_, _Layout_ e _Render_.

#### **Update**

Esta etapa tem como objetivo o cálculo do _frame_ para a _view_ tendo como base suas _constraints_. O sistema percorre a hierarquia de _views_ dos nós mais profundos para cima (_bottom-up_), ou seja, das _subviews_ para a _superview_, invocando o método `updateConstraints()` para cada _view_.

Este processo ocorre naturalmente ao se utilizar de uma _view_. No entanto, existem situações onde você precisará executá-lo por sua conta intervindo com código. Por exemplo, em casos onde algum evento em sua aplicação ou uma alteração em seu estado requeira recalcular imediatamente as _contraints_ para responder visualmente de forma adequadamente.

Existem outros métodos relacionados a esta fase do ciclo das _views_ que merecem atenção especial. É o caso dos métodos `setNeedsUpdateConstraints()` e `updateConstraintsIfNeeded()`. O método `setNeedsUpdateConstraints()` marca como invalidas as _constraints_ da _view_ e agenda uma atualização para o próximo ciclo. Já o método `updateConstraintsIfNeeded()` invoca `updateConstraints()`, se as _constraints_ foram invalidadas anteriormente.

> Nota: A documentação sugere não sobrescrever o método `updateConstraints()` a menos que você precise otimizar performance no cálculo das _constraints_ de layout.

#### **Layout**

Durante esta etapa, os _frames_ de cada _vista_ são atualizados com os retângulos calculados na fase anterior. O sistema percorre as _views_ da _view_ para as _subviews_ (_top-down_) invocando `layoutSubviews()` para cada uma.

O método `layoutSubviews()` é comumente sobrescrito no ciclo de vida da _view_ e em geral você faz isso quando:

* As _constraints_ não são suficientes para configurar o layout da _view_ adequadamente.
* Os _frames_ precisam ser calculados programaticamente.

Também existem dois métodos complementares nesta etapa. São eles `setNeedsLayout()` e `layoutIfNeeded()`. O método `setNeedsLayout()` invalida o _layout_ da _view_, enquanto `layoutIfNeeded()` se encarrega de invocar `layoutSubviews()` diretamente se o _layout_ foi invalidado anteriormente.

> Nota: É importante seguir algumas diretrizes ao sobrescrever o método `layoutSubviews()`.
>
>* Certifique-se de invocar `super.layoutSubviews()`;
>* Certifique-se de **não** invocar `setNeedsLayout()` ou `setNeedsUpdateConstraints()`, caso contrário um loop infinito será criado no ciclo de vida do objeto;
>* Certifique-se de **não** alterar as _constraints_ das _views_ fora da hierarquia atual;
>* Tenha cuidado ao alterar as _constraints_ de _views_ da hierarquia atual. A alteração acionará a etapa de atualização seguida por outra etapa de layout, criando potencialmente um loop infinito.

#### **Render**

Esta etapa é responsável por definir os pixels da tela. Por padrão, a `UIView` delega todo o trabalho para um objeto auxiliar de `CALayer` que contém um pixel bitmap com o estado de _view_ atual. Esta etapa é independente do Auto Layout em si.

O principal método desta etapa é o método `drawRect()`. A menos que você esteja desenhando algo personalizado com OpenGL ES, Core Graphics ou UIKit, não há necessidade de sobrescrever esse método. Todas as alterações, como para definição de background e adição de _subviews_, tem seu desenho executados automaticamente. Na maioria das vezes, você pode compor sua UI através de diferentes _views_ e camadas e não precisar sobrescrever o método `drawRect()`.

> Nota: Existem diversos métodos que compõem o ciclo de vida dos objetos de _view_ aos quais você pode recorrer para prover implementações para momentos específicos, no entanto, eles talvez sejam menos relevantes que os demonstrados neste documento. Sinta-se à vontade para explorar mais possibilidades consultando a documentação de [`UIView`](https://developer.apple.com/documentation/uikit/uiview).

### Voltando ao problema original

Agora que já temos suporte teórico podemos nos debruçar novamente ao problema introduzido anteriormente. Algo que ainda permanecia oculto sobre a implementação necessária, é onde escrever o código para definir o valor de `cornerRadius`. Sabendo agora mais detalhes sobre o ciclo de vida de uma _view_ já podemos supor o momento adequado.

Precisamos adicionar código no momento onde a _view_ já tem seu `frame` corretamente configurado. Recorrendo ao conhecimento compartilhado acima, podemos apostar em adicionar código ao final da fase de _layout_. Para isso é possível sobrescrever o método `layoutSubviews()`.

``` swift
//
//  DogViewCell.swift
//

    override func layoutSubviews() {
        imageView.layer.masksToBounds = true
        imageView.layer.cornerRadius = contentView.bounds.width / 2
    }
```

Com essa implementação, a partir da perspectiva de largura interna para a `contentView`, cuja a ImageView filha se estende por todo o eixo, podemos obter o valor adequado, equivalente ao raio da circuferência.

<p align="center">
<img alt="Imagem contendo um exemplo de layout com uma coleção de raças de cachorros cada um com sua imagem com shape arredondado de forma imperfeita" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/collection-views-teoria-o-que-pode-dar-errado-layout-com-circulos-imperfeitos.jpg?raw=true" width="100%" />
</p>

O resultado ainda não atinge o objetivo para a tela, mas traz à luz um problema recorrente que é importante exemplificar.

Nas seções acima, discorrendo sobre os detalhes e métodos mais importantes do ciclo _layout_ da _view_, tivemos notas importantes. A implementação atual não respeita uma delas: certificar-se de invocar `super.layoutSubviews()` ao sobrescrever o método. Dessa forma, impedimos a execução do fluxo natural de layout das _subviews_, importantes para a definição das dimensões da própria _view_ contêiner, substituindo seu código original integralmente.

É importante completar o ciclo de layout das _views_ e, somente após sua execução, adicionar qualquer implementação suplementar.

``` swift
//
//  DogViewCell.swift
//

    override func layoutSubviews() {
        super.layoutSubviews()
        
        imageView.layer.masksToBounds = true
        imageView.layer.cornerRadius = contentView.bounds.width / 2
    }
```

<p align="center">
<img alt="Imagem contendo o layout com uma coleção de raças de cachorros cada um com sua imagem com shape arredondado e com o nome da raça logo abaixo atingido" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/collection-views-teoria-o-que-pode-dar-errado-exemplo-de-layout.jpg?raw=true" width="100%" />
</p>

## Problemas com o layout: Quando o cálculo de constraints e o objeto de layout não se entendem

Embora alguns _layouts_ para as células de `UICollectionView` possam ser alcançados sem muito esforço através do uso de constraints de Auto Layout somadas às características base do objeto de layout da coleção, outros podem demonstrar algum comportamento estranho.

Por exemplo, algumas coleções podem precisar que suas células sejam dimensionadas dinamicamente através da resolução do conjunto de _constraints_ de Auto Layout que as definem.

| Exemplo de células para autor  | Exemplo de células para cores |
|:---|---:|
| <img height="540" alt="weird author cell" src="https://user-images.githubusercontent.com/13206745/126851230-0142b703-d84c-49f9-9247-b5e5303a8ae1.png" /> | <img height="540" alt="weird author cell" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/collection-views-teoria-o-que-pode-dar-errado-exemplo-de-layout-problematico.png?raw=true" />|
<figcaption align = "center"><b>Exemplos de layout problemático após a aplicação de constraints</b></figcaption>

Na imagem 1 do exemplo acima é necessário, para acomodar os componentes do _layout_, que as células tenham, por exemplo, sua definição de altura flexível. Na imagem 2, para o correto posicionamento e dimensionamento do label, o conjunto de constraints prende este elemento à região inferior da `contentView`, assim como à suas laterais. Em ambom os exemplos, há um conflito entre as dimensões de layout providas pelo objeto de layout da `UICollectionView` e as dimensões calculas pelo ciclo de layout baseado em Auto Layout que compromete a definição de todo o frame do componente.

Para solucionar o problema da UI é preciso intervir com código, ajustando as propriedades do layout que serão aplicados pela coleção.

``` swift
// Implementação da CollectionViewCell customizada

    override func preferredLayoutAttributesFitting(_ layoutAttributes: UICollectionViewLayoutAttributes) -> UICollectionViewLayoutAttributes {
        let size = contentView.systemLayoutSizeFitting(layoutAttributes.size)
        layoutAttributes.frame.size.height = size.height

        setNeedsLayout()
        return layoutAttributes
    }
```

A implementação acima sobrescreve o método [`preferredLayoutAttributesFitting(_:)`](https://developer.apple.com/documentation/uikit/uicollectionreusableview/1620132-preferredlayoutattributesfitting) da `UICollectionViewCell` que, como mencionada pela documentação, dá à célula a chance de modificar os atributos fornecidos pelo objeto de layout antes que eles sejam, de fato, aplicados. Assim, você pode adicionar implementação para calcular as dimensões do frame manualmente.

A implementação do método acima invoca o método [`systemLayoutSizeFitting(_:)`](https://developer.apple.com/documentation/uikit/uiview/1622624-systemlayoutsizefitting) da `contentView`. Esse método recebe como base um tamanho (preferido) a ser considerado, mas calcula e retorna o tamanho ideal da _view_ com base em suas _constraints_ atuais. Assim, obtemos o tamanho inicial (problemático) da célula pelos atributos de layout, calculamos o tamanho ideal para nossa composição de layout (definido pelas _constraints_ adicionadas via Interface Builder) e sobrescrevemos as informações do objeto de atributos de layout antes de devolvê-lo, corrigindo assim o problema.
