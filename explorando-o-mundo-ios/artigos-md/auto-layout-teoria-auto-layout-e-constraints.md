# Entendendo o Auto Layout

>Esse material teórico foi traduzido e editado da fonte original disponível em https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/index.html

O Auto Layout calcula dinamicamente o tamanho e a posição de todas as *views* em sua hierarquia de *views*, com base nas *constraints* - restrições de layout - adicionadas a essas *views*. Por exemplo, você pode restringir um botão para que seja centralizado horizontalmente com uma *view* de imagem e para que a borda superior do botão sempre permaneça 8 pontos abaixo da parte inferior da imagem. Se o tamanho ou a posição da *view* da imagem mudar, a posição do botão se ajustará automaticamente para corresponder. Essa abordagem de design baseada em restrições permite que você crie interfaces de usuário que respondem dinamicamente às mudanças internas e externas.

## Mudanças Externas 

Mudanças externas ocorrem quando o tamanho ou a forma de sua *view* mãe mudam. A cada mudança, você deve atualizar o layout de sua hierarquia de *views* para melhor usar o espaço disponível. Aqui estão algumas fontes comuns de mudança externa: 

* O usuário redimensiona a janela (OS X). 
* O usuário entra ou sai da visualização dividida em um iPad (iOS). 
* O dispositivo gira (iOS). 
* A chamada ativa e as barras de gravação de áudio aparecem ou desaparecem (iOS). 
* Você deseja oferecer suporte a tamanhos de tela diferentes. 

A maioria dessas mudanças pode ocorrer em tempo de execução e exigem uma resposta dinâmica do seu aplicativo. Outros, como o suporte para diferentes tamanhos de tela, representam a adaptação do aplicativo a diferentes ambientes. Mesmo que o tamanho da tela normalmente não mude em tempo de execução, a criação de uma interface adaptável permite que seu aplicativo seja executado igualmente bem em um iPhone 4S, um iPhone 6 Plus ou até mesmo um iPad. Auto Layout também é um componente-chave para oferecer suporte a exibições de slides e divididas no iPad, por exemplo.

## Mudanças Internas

Mudanças internas ocorrem quando o tamanho das *views* ou controles em sua UI mudam.

Aqui estão algumas fontes comuns de mudança interna:

* O conteúdo exibido pelo aplicativo muda.
* O aplicativo oferece suporte à internacionalização.
* O aplicativo é compatível com Fonte Dinâmica (iOS).

Quando o conteúdo do seu aplicativo muda, o novo conteúdo pode exigir um layout diferente do antigo. Isso geralmente ocorre em aplicativos que exibem texto ou imagens. Por exemplo, um aplicativo de notícias precisa ajustar seu layout com base no tamanho dos artigos de notícias individuais. Da mesma forma, uma colagem de fotos deve lidar com uma ampla variedade de tamanhos e proporções de imagem.

Internacionalização é o processo de tornar seu aplicativo capaz de se adaptar a diferentes idiomas, regiões e culturas. O layout de um aplicativo internacionalizado deve levar em consideração essas diferenças e aparecer corretamente em todos os idiomas e regiões que o aplicativo suporta. Alterar o idioma pode afetar não apenas o tamanho do texto, mas também a organização do layout. Idiomas diferentes usam direções de layout diferentes. O inglês, por exemplo, usa uma direção de layout da esquerda para a direita, e o árabe e o hebraico usam uma direção de layout da direita para a esquerda. Em geral, a ordem dos elementos da interface do usuário deve corresponder à direção do layout. Se um botão estiver no canto inferior direito da *view* em inglês, ele deverá estar no canto inferior esquerdo em árabe.

Finalmente, se o seu aplicativo iOS suportar a fonte dinâmica, o usuário pode alterar o tamanho da fonte usada em seu aplicativo. Isso pode alterar a altura e a largura de quaisquer elementos textuais na interface do usuário. Se o usuário alterar o tamanho da fonte enquanto seu aplicativo está em execução, as fontes e o layout devem se adaptar.

## Auto Layout x layout baseado em *frames*

Existem três abordagens principais para definir uma interface de usuário. Você pode definir o layout da interface do usuário de maneira programática, pode usar máscaras de redimensionamento automático (autoresizing masks) para automatizar algumas das respostas a alterações externas ou pode usar o **Auto Layout**.

Tradicionalmente, os aplicativos estabelecem sua interface de usuário configurando programaticamente o *frame* de cada *view* em uma hierarquia de *views*. O *frame* define a origem, altura e largura da *view* no sistema de coordenadas da *view* mãe.

<p align="center">
<img alt="ilustração de frames no sistema de coordenadas" src="https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/Art/layout_views_2x.png" width="320"/>
</p>

Para criar o layout de sua interface de usuário, você tinha que calcular o tamanho e a posição de cada *view* em sua hierarquia de *views*. Então, se uma mudança ocorria, você tinha que recalcular o quadro para todas as *views* efetadas.

De muitas maneiras, definir programaticamente o *frame* de uma *view* fornece mais flexibilidade e poder. Quando ocorre uma mudança, você pode literalmente fazer qualquer mudança que desejar. No entanto, como você também deve gerenciar todas as alterações sozinho, o layout de uma interface de usuário simples requer uma quantidade considerável de esforço para projetar, depurar e manter. A criação de uma interface de usuário verdadeiramente adaptável aumenta a dificuldade em uma ordem de magnitude.

Você pode usar máscaras de redimensionamento automático para ajudar a aliviar parte desse esforço. Uma máscara de redimensionamento automático define como o *frame* de uma *view* muda quando o *frame* de sua *view* mãe muda. Isso simplifica a criação de layouts que se adaptam às mudanças externas.

No entanto, as máscaras de redimensionamento automático oferecem suporte a um subconjunto relativamente pequeno de layouts possíveis. Para interfaces de usuário complexas, você normalmente precisa acrescentar as máscaras de redimensionamento automático com suas próprias alterações programáticas. Além disso, as máscaras de redimensionamento automático se adaptam apenas à mudanças externas. Eles não suportam mudanças internas.

Embora as máscaras de redimensionamento automático representem uma melhoria iterativa nos layouts programáticos, o Auto Layout representa um paradigma inteiramente novo. Em vez de pensar sobre o *frame* de uma *view*, você pensa sobre seus **relacionamentos**.

O Auto Layout define sua interface de usuário usando uma série de *constraints* - restrições de layout. **As restrições geralmente representam um relacionamento entre duas *views***. O Auto Layout então calcula o tamanho e a localização de cada *view* com base nessas restrições. Isso produz layouts que respondem dinamicamente às mudanças internas e externas.

<p align="center">
<img alt="ilustração de constraints definindo as relações entre as views" src="https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/Art/layout_constraints_2x.png" width="320" />
</p>

A lógica usada para projetar um conjunto de *constraints* criando comportamentos específicos é diferente da lógica usada para escrever código procedural ou orientado a objetos. Felizmente, dominar o Auto Layout não é diferente de dominar qualquer outra tarefa de programação. Existem duas etapas básicas: primeiro, você precisa entender a lógica por trás dos layouts baseados em restrições e, em seguida, precisa aprender a API. Você executou com sucesso essas etapas ao aprender outras tarefas de programação. O Auto Layout não é exceção.

# Anatomia de uma *constraint*

O layout de sua hierarquia de *views* é definido como uma série de equações lineares. Cada *constraint* representa uma única equação. Seu objetivo é declarar uma série de equações que possuem uma e apenas uma solução possível.

Um exemplo de equação é mostrado abaixo.

<p align="center">
<img alt="ilustração que exemplifica a equação equivalente de uma constraint" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/auto-layout-teoria-auto-layout-constraints.png?raw=true" width="70%" />
</p>

Essa *constraint* afirma que a borda inicial¹ - *leading edge* - da *view* em vermelho deve ter `8,0` pontos após a borda final² - *trailing edge* - da *view* em azul. Sua equação tem várias partes:

* Item 1: O primeiro item na equação - neste caso, a *view* em vermelho. O item deve ser uma *view* ou um guia de layout (veremos no futuro).
* Atributo 1: O atributo a ser restringido no primeiro item - neste caso, a borda inicial da *view* em vermelho.
* Relacionamento: A relação entre os lados esquerdo e direito. O relacionamento pode ter um de três valores: igual, maior ou igual, ou menor ou igual. Nesse exemplo, define que os lados esquerdo e direito são iguais.
* Multiplicador: O valor do atributo 2 é multiplicado por este número de ponto flutuante. Nesse caso, o multiplicador é `1,0`.
* Item 2: O segundo item da equação - neste caso, a *view* azul. Ao contrário do primeiro item, pode ser deixado em branco.
* Atributo 2: O atributo a ser restringido no segundo item - neste caso, a borda final da *view* azul. Se o segundo item não existir, deve ser *`Not An Attribute`* - algo diferente de um atributo.
* Constante: Um deslocamento de ponto flutuante constante - neste caso, `8,0`. Este valor é adicionado ao valor do atributo 2.

A maioria das *constraints* define um relacionamento entre dois itens em nossa interface de usuário. Esses itens podem representar *views* ou guias de layout. As restrições também podem definir a relação entre dois atributos diferentes de um único item, por exemplo, definindo uma proporção entre a altura e a largura de um item. Você também pode atribuir valores constantes à altura ou largura de um item. Ao trabalhar com valores constantes, o segundo item é deixado em branco, o segundo atributo é definido como *`Not An Attribute`* e o multiplicador é definido como `0,0`.

> ¹ equivalente ao lado de origem da direção de layout, por exemplo, esquerdo na direção de layout do português e inglês ou direito na direção de layout do árabe ou hebraico.
>
> ² equivalente ao lado do destino da direção de layout, por exemplo, direito na direção de layout do português e inglês ou esquerdo na direção de layout do árabe ou hebraico.

## Atributos de Auto Layout

No Auto Layout, os atributos definem um recurso que pode ser restringido. Em geral, isso inclui as quatro bordas (inicial, final, superior e inferior), bem como a altura, a largura e os centros vertical e horizontal. Os itens de texto também têm um ou mais atributos de *baseline*.

<p align="center">
<img alt="ilustração dos atributos de auto layout sobre o frame de uma view de exemplo" src="https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/Art/attributes_2x.png" width="460" />
</p>

[Veja aqui](https://developer.apple.com/documentation/uikit/nslayoutattribute) a lista completa de atributos.

## Exemplos de equações

A ampla gama de parâmetros e atributos disponíveis para essas equações permite criar muitos tipos diferentes de *constraints*. Você pode definir o espaço entre as *views*, alinhar a borda das *views*, definir o tamanho relativo de duas *views* ou mesmo definir a proporção de uma *view*. No entanto, nem todos os atributos são compatíveis.

Existem dois tipos básicos de atributos. Atributos de tamanho (por exemplo, *`Height`* e *`Width`*) e atributos de localização (por exemplo, *`Leading`*, *`Left`* e *`Top`*). Os atributos de tamanho são usados para especificar o tamanho de um item, sem qualquer indicação de sua localização. Os atributos de localização são usados para especificar a localização de um item em relação a outra coisa. No entanto, eles não trazem nenhuma indicação do tamanho do item.

Com essas diferenças em mente, as seguintes regras se aplicam:

* Você não pode restringir um atributo de tamanho a um atributo de local.
* Você não pode atribuir valores constantes a atributos de localização.
* Você não pode usar um multiplicador de não identidade (um valor diferente de 1,0) com atributos de localização.
* Para atributos de localização, você não pode restringir atributos verticais a atributos horizontais.
* Para os atributos de localização, você não pode restringir os atributos de *`Leading`* ou *`Trailing`* aos atributos de *`Left`* ou *`Right`*.

Por exemplo, definir o *`Top`* de um item com o valor constante `20,0` não tem significado sem contexto adicional. Você deve sempre definir os atributos de localização de um item em relação a outros itens, por exemplo, `20,0` pontos abaixo do topo da *view*. No entanto, definir a altura de um item para `20,0` é perfeitamente válido. Para obter mais informações, consulte [Interpretando valores](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/AnatomyofaConstraint.html#//apple_ref/doc/uid/TP40010853-CH9-SW22).

A Listagem 1 mostra exemplos de equações para uma variedade de *constraints* comuns.

>Nota: Todas as equações de exemplo neste capítulo são apresentadas usando pseudocódigo.

*Listagem 1* Exemplos de equações para *constraints* comuns

``` pseudo
// Definindo uma altura constante de 40 pontos
View.height = 0.0 * NotAnAttribute + 40.0
 
// Definindo uma distância fixa de 8 pontos entre dois botões
Button2.leading = 1.0 * Button1.trailing + 8.0
 
// Alinhando a borda inicial de dois botões
Button1.leading = 1.0 * Button2.leading + 0.0
 
// Definindo a mesma largura à dois botões
Button1.width = 1.0 * Button2.width + 0.0
 
// Centralizando uma view à sua view mãe
View.centerX = 1.0 * Superview.centerX + 0.0
View.centerY = 1.0 * Superview.centerY + 0.0
 
// Definindo uma proporção (aspect ratio) constante a uma view
View.height = 2.0 * View.width + 0.0

```

## Igualdade, não atribuição

É importante notar que as equações mostradas acima representam igualdade, não atribuição.

Quando o Auto Layout resolve essas equações, ele não atribui apenas o valor do lado direito ao esquerdo. Em vez disso, ele calcula o valor do atributo 1 e do atributo 2 que torna o relacionamento verdadeiro. Isso significa que muitas vezes podemos reordenar livremente os itens na equação. Por exemplo, as equações na Listagem 2 são idênticas às suas contrapartes vistas acima.

*Listagem 2* Equações invertidas

``` pseudo 
// Definindo uma distância fixa de 8 pontos entre dois botões
Button1.trailing = 1.0 * Button2.leading - 8.0
 
// Alinhando a borda inicial de dois botões
Button2.leading = 1.0 * Button1.leading + 0.0
 
// Definindo a mesma largura à dois botões
Button2.width = 1.0 * Button1.width + 0.0
 
// Centralizando uma view à sua view mãe
Superview.centerX = 1.0 * View.centerX + 0.0
Superview.centerY = 1.0 * View.centerY + 0.0
 
// Definindo uma proporção (aspect ratio) constante a uma view
View.width = 0.5 * View.height + 0.0

```

>NOTA: Ao reordenar os itens, certifique-se de inverter o multiplicador e a constante. Por exemplo, uma constante de `8,0` torna-se `-8,0`. Um multiplicador de `2,0` torna-se `0,5`. Constantes de `0,0` e multiplicadores de `1,0` permanecem inalterados.
