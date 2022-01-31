# Implementando a primeira tela

Primeiramente precisamos de uma nova tela disponível no canvas. Adiciona-se então um novo objeto de `View Controller` ao mesmo. 

* Clique no arquivo `Main.storyboard`.
* Clique no ícone _(+)_ superior direito do Interface Buider ou uso do atalho _⌘⇧ + L_.
* _`View Controller`_ no campo de busca.
* _Drag and drop_ do elemento para o canvas.

![imagem com view controller disposto no canvas do Interface Builder](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/auto-layout-lt2-view-controller-inicial.png?raw=true)
*OBS: Certificando de selecionar o View Controller e verificar a marcação da opção Is Initial View Controller*

Agora é preciso de um título para a tela. 

* Clique no ícone _(+)_ superior direito do Interface Buider ou uso do atalho _⌘⇧ + L_.
* _`label`_ no campo de busca.
* _Drag and drop_ do elemento para o canvas.
* Altera o identificador no Document Outline para `Label Título`.

Para configurar a visualização do elemento basta selecioná-lo, abrir o inspetor de atributos e na sequência alterar algumas propriedades.

* Valor do campo para `Meus Registros`.
* _Font style_ para _Medium_.
* Tamanho da fonte para `24`.

Com a tipografia já configurada, basta adicionar as constraints de posicionamento do componente:

* Clique no ícone inferior direito _Add New Constraints_.
* Clique no indicador visual da âncora superior _(top)_ e configuração de `24` pontos de distância em relação à `Safe Area Layout Guide`.
* Clique no indicador visual da âncora inicial/esquerda _(leading)_ e da âncora final/direita _(trailing)_ seguida de configuração de `20` pontos de distância em relação à `Safe Area Layout Guide` em ambas.
* Clique no botão _Add 3 Constraints_.

![imagem com label posicionado no canvas](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/auto-layout-lt2-label-posicionado.png?raw=true)
*Label posicionado*

Na sequência, uma _View_ é necessária para conter as divisões para cada item de seção.

* Clique no ícone _(+)_ superior direito do Interface Buider ou uso do atalho _⌘⇧ + L_.
* `uiv` no campo de busca.
* _Drag and drop_ de um elemento de `View` para o canvas, preferencialmente abaixo do label.
* Alterar o identificador no Document Outline para `View Container`.

Com a _View_ presente, configuramos suas constraints básicas:

* Clique no ícone inferior direito _Add New Constraints_.
* Clique no indicador visual da âncora superior _(top)_ e configuração de `24` pontos de distância em relação ao label`.
* Clique no indicador visual da âncora inicial/esquerda _(leading)_ e da âncora final/direita _(trailing)_ seguida de configuração de `0` ponto de distância em relação à `Safe Area Layout Guide` em ambas.
* Clique no _check box_ indicador de altura _(Height)_ e configuração da constante em `140` pontos.
* Clique no botão _Add 4 Constraints_.

Agora que a _View Container_ já se encontra posicionada é preciso adicionar as _views_ internas que dividirão o espaço de tela entre si.

* Clique no ícone _(+)_ superior direito do Interface Buider ou uso do atalho _⌘⇧ + L_.
* `uiv` no campo de busca.
* _Drag and drop_ de um elemento de `View` para o canvas, preferencialmente dentro dos limites da _View Container_ .
* Alterar o identificador no Document Outline para `View Minha Saude`.

Feito isso, configurar as contraints orientando posicionamento e dimensionamento:

* Clique no ícone inferior direito _Add New Constraints_.
* Clique no indicador visual da âncora superior _(top)_ e da âncora inferior _(bottom)_ seguida de configuração de `0` ponto de distância em relação à _view_ mãe em ambas`.
* Clique no indicador visual da âncora inicial/esquerda _(leading)_ e configuração de `0` ponto de distância em relação à _view_ mãe.
* Clique no botão _Add 3 Constraints_.
* Clique com a tecla _control_ pressionada seguido de drag para a _view_ mãe e seleção da opção _Equal Widths_.
* Clique no indicador de constraint de largura e edição do valor multiplicador para `1/3` no inspetor de atributos da constraint.

Adicionar a _Image View_ que vai apresentar o ícone dentro dos limites da `View Minha Saude`.

* Clique no ícone _(+)_ superior direito do Interface Buider ou uso do atalho _⌘⇧ + L_.
* `image` no campo de busca.
* _Drag and drop_ de um elemento de `Image View` para o canvas, preferencialmente ajustando seu tamanho para acomodar corretamente dentro da _view_ mãe.

Feito isso, configurar as contraints orientando posicionamento e dimensionamento.

* Clique no ícone inferior direito _Add New Constraints_.
* Clique no indicador visual da âncora inicial/esquerda _(leading)_ e configuração de `20` pontos de distância em relação à _view_ mãe.
* Clique no indicador visual da âncora superior _(top)_ e configuração de `0` ponto de distância em relação à _view_ mãe.
* Clique no indicador visual da âncora final/direita _(trailing)_ e configuração de `12` pontos de distância em relação à _view_ mãe.
* Clique no botão _Add 3 Constraints_.

* No inspetor de tamanho, definir uma altura _(Height)_ igual à largura _(Width)_ calculada a partir da aplicação das constraints acima. Caso o canvas do Interface Builder esteja utilizando a referência de tamanho do iPhone 13 Pro como nos vídeos do especialista, o valor deve ser `98`.
* Com a imagem selecionada, clique ícone inferior direito _Add New Constraints_.
* Selecione o _check box_ referente a proporção _Aspect Ratio_.
* Clique no botão _Add 1 Constraint_.

Com a _Image View_ disposta de forma adequada podemos finalizar seus ajustes configurando as últimas propriedades de visualização. Primeiro adicionando alguns recursos necessários.

* No explorador de projeto, seleção da pasta `Assets`.
* No quadro central, clique com o botão direito do mouse e seleção da opção _New Folder_.
* Renomeação da nova pasta para _Icons_.
* No _Finder_ do Mac OS, seleção dos arquivos `.svg` com os ícones de avatar, doações de sangue e contados de emergência.
* _Drag and drop_ dos ícones para dentro da pasta _Icons_ na janela do XCode - caso o XCode abra uma janela assistência, certificação de marcação da opção de criar cópias dos arquivos no novo diretório.

Ajustando atributos básicos.

* De volta ao Interface Builder, através de clique no arquivo `Main.storyboard`.
* Seleção da _Image View_ e do inspetor de atributos.
* Seleção da imagem adequada para o ícone na opção _Image_.
* Seleção da opção _Center_ no seletor de _Content Mode_.
* Seleção da opção _Secondary System Background Color_ no seletor de _Background_.

Adicionando suporte para os cantos arredondados através de atributos que serão adicionado em tempo de execução pelo Storyboard.

* Seleção do inspetor de identidade.
* Clique no ícone de _+_ na seção _User Defined Runtime Attributes_.
* Definição de `layer.cornerRadius`, _Number_ e `8` para _Key Path_, _Type_ e _Value_ respectivamente.
* Novo clique no ícone de _+_ na seção _User Defined Runtime Attributes_.
* Definição de `layer.maskToBounds`, Boolean e `true` (check box marcado) para _Key Path_, _Type_ e _Value_ respectivamente.

![imagem com objeto Image View configurado no canvas](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/auto-layout-lt2-imagem-configurada.png?raw=true)
*Image View configurada*

Com mais esse passo, podemos ir em frente e adicionar o label com o título da seção.

* Clique no ícone _(+)_ superior direito do Interface Buider ou uso do atalho _⌘⇧ + L_.
* `label` no campo de busca.
* _Drag and drop_ de um elemento de `Label` para o canvas, preferencialmente abaixo da `Image View` e dentro dos limites da _view_ mãe.

Com o label selecionado, no inspetor de atributos, configurar visualmente o componente.

* `Minha Saúde` como valor para o label.
* `14` para _Font Size_.
* `Medium` para _Font Style_.
* _Alignment_ configurado para _center_.
* _Lines_ configurado para `2` para orientar a quantidade máxima de linhas.

Por fim, configurar as constraints para o elemento.

* _Control + drag_ do componente do label para o componente da imagem no canvas.
* Pressionando _Shift_, seleção de _Vertical Spacing_, _Center Horizontally_ e _Equal Widths_, seguido de _Enter_ para aplicar - o Interface Builder adiciona automaticamente as constraints com os valor correntes do canvas.
* Seleção da constraint de _Vertical Spacing_ e ajuste do valor da constante, caso necessário, para `12`.
* Seleção da constraint de _Equal Widths_ e ajuste do valor do multiplicador para `1`.

![imagem com objeto Label configurado abaixo da Image View](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/auto-layout-lt2-label-de-secao-posicionado.png?raw=true)
*Label configurado*

Com a primeira das três _views_ finalizada, podemos reaproveitar o trabalho criando uma cópia para a seção Doação de Sangue e adicionar novas constraints.

* Clique com a tecla _option_ pressionada e _drag_ para o lado no canvas - ou para baixo pelo Document Outline - mantendo a nova cópia no mesmo nível hierárquico.
* Alteração do identificador para `View Doacoes` no Document Outline.
* _Control + drag_ da View Doacoes para o componente da View Minha Saúde.
* Pressionando _Shift_, seleção de _Horizontal Spacing_, _Top_, _Bottom_ e _Equal Widths_, seguido de _Enter_ para aplicar.

Com a nova seção já configurada, selecionando sua _Image View_ interna, podemos corrigir alguns valores de constraints e conteúdos da imagem e do label para atingir a especificação de layout.

* Seleção das constraints de leading e trailing da _Image View_ (pressionando _command_) e ajuste do valor das constantes para `16` no inspetor de atributos.
* Seleção da imagem com o ícone apropriado para doação de sangue no inspetor de atributos da _Image View_.
* Alteração para `Doação de Sangue` no valor do _Label_.

![imagem com a View Doações configurada ao lado da View Minha Saúde](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/auto-layout-lt2-view-doacoes.png?raw=true)
*View Doações configurada*

Com a possibilidade de reaproveitar o trabalho copiando as _views_ não fica muito difícil presumir os passos necessários para a seção Contatos. Apenas necessário se atentar aos valores de constraints e conteúdos e repetir o trabalho do passo acima criando uma cópia a partir da _View Doacoes_.

![imagem com a View Contatos configurada ao lado da View Doações](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/auto-layout-lt2-view-contatos.png?raw=true)
*View Contatos configurada*

Temos assim a primeira tela construída.
