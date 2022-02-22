# UIStackView

Uma interface simplificada para exibir um conjunto de _views_ em linhas ou colunas.

>Esse material teórico foi adaptado da fonte original disponível em https://developer.apple.com/documentation/uikit/uistackview

## Visão geral

_Stack views_ permitem que você aproveite o poder do Auto Layout, criando interfaces com o usuário que podem se adaptar dinamicamente à orientação do dispositivo, tamanho da tela e quaisquer alterações no espaço disponível. A _stack view_ gerencia o layout de todas as _views_ na sua propriedade [`arrangedSubviews`](https://developer.apple.com/documentation/uikit/uistackview/1616232-arrangedsubviews). Essas _views_ são organizadas ao longo do eixo da _stack view_, com base em sua ordem no array [`arrangedSubviews`](https://developer.apple.com/documentation/uikit/uistackview/1616232-arrangedsubviews). O layout varia dependendo das propriedades [axis](https://developer.apple.com/documentation/uikit/uistackview/1616223-axis), [distribution](https://developer.apple.com/documentation/uikit/uistackview/1616233-distribution), [alignment](https://developer.apple.com/documentation/uikit/uistackview/1616243-alignment), [spacing](https://developer.apple.com/documentation/uikit/uistackview/1616225-spacing) e outras propriedades da _stack view_.

<p align="center">
<img alt="Imagem de demonstração geral de uma stack view horizontal" src="https://docs-assets.developer.apple.com/published/82128953f6/uistack_hero_2x_04e50947-5aa0-4403-825b-26ba4c1662bd.png" width="80%"/>
</p>

Para usar uma _stack view_, abra o Storyboard que você deseja editar. Arraste uma Stack View Horizontal ou uma Stack View Vertical da biblioteca de objetos e posicione-a onde desejar. Em seguida, arraste o conteúdo desejado, soltando a _view_ dentro dos limites da _stack view_. Você pode continuar a adicionar _views_ e controles à sua _stack view_, conforme necessário. O Interface Builder redimensiona a _stack view_ com base em seu conteúdo. Você também pode ajustar a aparência do conteúdo da _stack_ modificando as propriedades da Stack View no inspetor de atributos.

>Nota: Você é responsável por definir a posição e (opcionalmente) o tamanho da _stack view_. A _stack view_ gerencia o layout e o tamanho de seu conteúdo apenas.

### Stack View e Auto Layout

A _stack view_ usa o Auto Layout para posicionar e dimensionar suas _views_. A _stack view_ alinha a primeira e a última _view_ com suas bordas ao longo do eixo da _stack view_. Em uma _stack view_ horizontal, isso significa que a borda inicial (_leading edge_) da primeira _view_ é fixada à borda inicial da _stack view_ e a borda final (trailing edge) da última _view_ é fixada à borda final da _stack view_. Em _stack views_ verticais, as bordas superior e inferior são fixadas nas bordas superior e inferior da _stack view_, respectivamente. Se você definir a propriedade (`isLayoutMarginsRelativeArrangement`)[https://developer.apple.com/documentation/uikit/uistackview/1616220-islayoutmarginsrelativearrangeme] da _stack view_ como `true` (no Interface Builder, através do atributo _Layout Margins_ do _Size Inspector_), a _stack view_ fixará seu conteúdo na margem configurada em vez de na borda.

Para todas os tipos de **distribuição**, exceto a distribuição (`UIStackView.Distribution.fillEqually`)[https://developer.apple.com/documentation/uikit/uistackview/distribution/fillequally], a _stack view_ usa a propriedade (`intrinsicContentSize`)[https://developer.apple.com/documentation/uikit/uiview/1622600-intrinsiccontentsize] de cada _view_ ao calcular seu tamanho ao longo do eixo da _stack_. (`UIStackView.Distribution.fillEqually`)[https://developer.apple.com/documentation/uikit/uistackview/distribution/fillequally] redimensiona todas as _views_ para que tenham o mesmo tamanho, preenchendo a _stack view_ ao longo de seu eixo. Se possível, a _stack view_ estica todas as _views_ para corresponder à _view_ com o maior tamanho intrínseco ao longo do eixo da _stack_.

Para todos os **alinhamentos**, exceto o alinhamento (`UIStackView.Alignment.fill`)[https://developer.apple.com/documentation/uikit/uistackview/alignment/fill], a _stack view_ usa a propriedade `intrinsicContentSize` de cada _view_ ao calcular seu tamanho perpendicular ao eixo da _stack_. (`UIStackView.Alignment.fill`)[https://developer.apple.com/documentation/uikit/uistackview/alignment/fill] redimensiona todas as _views_ para que elas preencham a _stack view_ perpendicularmente ao seu eixo. Se possível, a _stack view_ estica todas as _views_ para corresponder à _view_ com o maior tamanho intrínseco perpendicular ao eixo da _stack_.

<p align="center">
<img alt="Imagem de exemplo das propriedades de distribuição e alinhamento aplicadas" src="https://docs-assets.developer.apple.com/published/60e0116332/cd8978aa-c754-456a-b2b3-d5485952fc9d.png" width="60%"/>
</p>

#### Posicionando e dimensionando a _stack view_

Embora uma _stack view_ permita que você organize seu conteúdo sem usar o Auto Layout diretamente, você ainda precisa usar o Auto Layout para posicionar a _stack view_ em si. Em geral, isso significa fixar pelo menos duas direções de layout da _stack view_ para definir sua posição. Sem nenhuma _constraint_ adicionada, o sistema calcula o tamanho da _stack view_ com base em seu conteúdo.

* Ao longo do eixo da _stack view_, seu tamanho é igual à soma dos tamanhos de todas as _views_ gerenciadas mais o espaço entre as _views_.

* Perpendicular ao eixo da _stack view_, seu tamanho é igual ao tamanho da maior _view gerenciada_.

* Se a propriedade (`isLayoutMarginsRelativeArrangement`)[https://developer.apple.com/documentation/uikit/uistackview/1616220-islayoutmarginsrelativearrangeme] da _stack view_ estiver definida como `true`, o tamanho da _stack view_ será aumentado para incluir espaço para as margens.

Você pode fornecer _constraints_ adicionais para especificar a altura, a largura ou ambos da _stack view_. Nesses casos, a _stack view_ ajusta o layout e o tamanho de suas _views gerenciadas_ para preencher a área especificada. O layout varia com base nas propriedades da _stack view_. Consulte as enums (`UIStackView.Distribution`)[https://developer.apple.com/documentation/uikit/uistackview/distribution] e (`UIStackView.Alignment`)[https://developer.apple.com/documentation/uikit/uistackview/alignment] para obter uma descrição completa de como a _stack view_ lida com espaço extra ou espaço insuficiente para seu conteúdo.

Você também pode posicionar uma _stack view_ com base em sua primeira ou última linha de base, em vez de usar a posição do eixo Y superior, inferior ou central. Assim como o tamanho da _stack view_, essas linhas de base são calculadas com base no conteúdo da _stack view_.

> Nota: O alinhamento via _baseline_ funciona apenas em _views_ cuja altura corresponde à altura do tamanho do conteúdo intrínseco. Se a vista for esticada ou comprimida, a linha de base aparecerá no local errado.

#### Layouts comuns com Stack Views

Abordagens comuns para layouts com conteúdo apresentado através de _stack views_:

**Defina apenas a posição**. Você pode definir a posição da _stack view_ fixando dois de seus eixos em sua _superview_. Nesse caso, o tamanho da _stack view_ cresce livremente em ambas as dimensões, com base em suas _views_ gerenciadas. Essa abordagem é particularmente útil quando você deseja que o conteúdo da _stack view_ apareça em seu tamanho de conteúdo intrínseco e deseja organizar outros elementos da interface do usuário em relação à _stack view_.

A figura abaixo mostra uma _stack view_ com suas bordas inicial e superior fixadas em sua _superview_. Os _labels_ são ajustados em relação ao _first baseline_, com um espaço de 8 pontos entre eles, alinhando à esquerda o conteúdo da _stack view_ em sua _superview_.

<p align="center">
<img alt="Imagem de exemplo de um arranjo comum de layout com stack views horizontais" src="https://docs-assets.developer.apple.com/published/f8e8e88004/786347a6-3c16-480d-949e-8319ebcc68f7.png" width="60%"/>
</p>

**Defina o tamanho da _stack_ de acordo com seu eixo**. Fixe ambas as bordas da _stack_ ao longo de seu eixo em sua _superview_, definindo o tamanho da _stack view_ nessa dimensão. Você também precisa fixar uma das outras bordas para definir a posição da _stack view_. A _stack view_ dimensiona e posiciona seu conteúdo ao longo de seu eixo para preencher o espaço definido; no entanto, a borda não fixada se move livremente, com base no tamanho da maior _view_ interna.

A figura  abaixo mostra uma _stack view_ com as bordas inicial, superior e final fixadas em sua _superview_. O uso da distribuição `fill` faz com que o conteúdo seja redimensionado para preencher a largura da _view_ e, como o campo de texto tem uma propriedade _content hugging priority_* menor do que o _label_, ele é esticado conforme necessário.

<p align="center">
<img alt="Imagem de exemplo de um arranjo comum de layout com label e campos horizontais" src="https://docs-assets.developer.apple.com/published/19ba11ed1c/0b922694-dbfc-4123-b3d6-02f70ac60a6c.png" width="60%"/>
</p>

*_Content hugging priority_ é uma propriedade que dita o quão aderente ao seu tamanho intrínseco (tamanho suficiente para a apresentação de seu conteúdo) uma _view_ deve ser. Em situações possivelmente ambíguas de layout, como a da última imagem, essa propriedade pode ser utilizada para diferenciar views por essa característica. Quanto maior o valor para _content hugging priority_, maior a probabilidade dessa _view_ permanecer com seu tamanho intrínseco, quanto menor o valor para _content hugging priority_ maior a probabilidade da _view_ ser esticada além de seu tamanho intrínseco para resolver situações como a mencionada acima.

**Defina o tamanho da _stack_ de forma perpendicular ao seu eixo**. Essa abordagem é semelhante ao exemplo anterior, mas você fixa as duas bordas perpendiculares ao eixo da _stack view_ e apenas uma borda ao longo do eixo. Isso permite que a _stack view_ cresça e diminua ao longo de seu eixo à medida que você adiciona e remove _views_ gerenciadas. A menos que você use uma distribuição `fillEqually`, as _views_ gerenciadas pela _stack view_ são dimensionadas de acordo com o tamanho intrínseco. Perpendiculares ao eixo, as _views_ são dispostas no espaço definido com base no alinhamento da _stack view_.

A figura abaixo mostra uma _stack view_ vertical contendo quatro _labels_ e um botão. A _stack_ usa espaçamento de 8,0 pontos e alinhamento central. A altura da _stack view_ aumentará e diminuirá à medida que os itens forem adicionados ou removidos da pilha.

<p align="center">
<img alt="Imagem de exemplo de um arranjo comum de layout vertical" src="https://docs-assets.developer.apple.com/published/cc6e21faab/651cc05e-4df4-493a-8113-41fed7da7d3f.png" width="60%"/>
</p>

**Defina o tamanho e a posição da _stack view_**. Nesse caso, você fixa todas as quatro bordas da _stack view_, fazendo com que a _stack view_ disponha seu conteúdo dentro do espaço fornecido.

A abaixo mostra uma _stack view_ vertical com todas as quatro bordas fixadas em sua _superview_. Ao usar o alinhamento `center` e a distribuição `fill`, a _stack view_ garante que seu conteúdo seja centralizado horizontalmente e preencha verticalmente a tela. No entanto, obter o layout desejado com essa abordagem requer algumas etapas adicionais. Por padrão, a _stack view_ estica verticalmente o rótulo e não a _view_ da imagem. Para redimensionar a _view_ da imagem, diminua sua _content hugging priority_ abaixo do valor de _content hugging priority_ do rótulo. Além disso, para manter a proporção da _view_ da imagem à medida que ela é redimensionada, defina seu _Content Mode_ como _Aspect Fit_. Adicionar uma _constraint_ para _equal width_ entre a _view_ da imagem e a _stack view_ ajuda a garantir que a imagem seja dimensionada para preencher o espaço disponível.

<p align="center">
<img alt="Imagem de exemplo de um arranjo comum de layout vertical com tamanho e posição fixos para o stack view" src="https://docs-assets.developer.apple.com/published/e77213019f/1ccce13c-b8b5-4653-8866-f5cc53331641.png" width="60%"/>
</p>

### Gerenciando a aparência da Stack View

Uma _stack view_ gerencia a posição e o tamanho de suas _views internas_. Há várias propriedades que definem como a _stack view_ apresenta seu conteúdo.

* A propriedade (`axis`)[https://developer.apple.com/documentation/uikit/uistackview/1616223-axis] determina a orientação da _stack_, seja vertical ou horizontalmente.

* A propriedade (`distribution`)[https://developer.apple.com/documentation/uikit/uistackview/1616233-distribution] determina o layout das _views_ internas ao longo do eixo da _stack_.

* A propriedade (`alignment`)[https://developer.apple.com/documentation/uikit/uistackview/1616243-alignment] determina o layout das _views_ dispostas perpendicularmente ao eixo da _stack_.

* A propriedade (`spacing`)[https://developer.apple.com/documentation/uikit/uistackview/1616225-spacing] determina o espaçamento mínimo entre as _views_ internas.

* A propriedade (`isBaselineRelativeArrangement`)[https://developer.apple.com/documentation/uikit/uistackview/1616224-isbaselinerelativearrangement] determina se o espaçamento vertical entre as _views_ é medido a partir dos _baselines_.

* A propriedade (`isLayoutMarginsRelativeArrangement`)[https://developer.apple.com/documentation/uikit/uistackview/1616220-islayoutmarginsrelativearrangeme] determina se a _stack view_ apresenta suas _views_ internas em relação às suas margens de layout.

Normalmente, você usa uma _stack view_ única para dispor um pequeno número de itens. Você pode criar hierarquias de _views_ mais complexas aninhando _stack views_ dentro de outras _stack views_. Por exemplo, a figura abaixo mostra uma _stack view_ vertical contendo duas _stack views_ horizontais. Cada uma das _stack views_ horizontais contém um _label_ e um campo de texto.

<p align="center">
<img alt="Imagem com exemplo de aninhamento de stack views" src="https://docs-assets.developer.apple.com/published/82128953f6/NestedStacks_2x_551c6bce-06bc-4bcb-98ea-ab8776378d11.png" width="60%"/>
</p>

Você também pode ajustar a aparência de uma _view_ interna adicionando _constraints_ adicionais à _view_ interna. Por exemplo, você pode usar _constraints_ para definir uma altura ou largura mínima ou máxima para a _view_. Ou você pode definir uma proporção para a _view_. A _stack view_ usa essas _constraints_ ao dispor seu conteúdo. Por exemplo, na _view_ da imagem, há uma _constraint_ de proporção que impõe uma proporção constante à medida que a imagem é redimensionada.

> Nota: Tenha cuidado para não introduzir conflitos ao adicionar _constraints_ a _views_ dentro de uma _stack view_. Como regra geral, se o tamanho de uma _view_ voltar ao tamanho do conteúdo intrínseco para uma determinada dimensão, você poderá adicionar com segurança uma _constraint_ para essa dimensão.
