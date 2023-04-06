# Por que não utilizar Interface Builder e storyboards?

Durante todo o treino Explorando o Mundo iOS utilizamos o XCode Interface Builder como base para o desenvolvimento de nossa camada de visualização. Através de arquivos _.storyboard_ adicionamos representações de todos os nossos controladores, bem como suas hierarquias de _views_, definimos todos os detalhes de estilização e layout para as mesmas e configuramos a sequência de navegações entre cenas como um fluxo contínuo constituindo nossa experiência de utilização.

Até aqui ele cumpriu bem seu papel. Com o uso do Interface Builder conseguimos entender de maneira prática e didática o funcionamento base dos objetos fundamentais de UIKit, assim como do Auto Layout. Com uma rápida resposta visual às iterações e pouca necessidade de entendimento de 
minucias do ciclo de vida dos componentes, pudemos avançar sobre temas diversos do desenvolvimento iOS. Entretanto, fizemos poucas análises reais dos possíveis contras a respeito dessa abordagem para o design.

Ao longo do desenvolvimento dos projetos deste treino - ou mesmo os do seu dia-a-dia - você deve ter se deparado com diversas situações onde o objetivo para sua _view_ não pode ser atingido apenas com o suporte da ferramenta. Nesses momentos é provável que você tenha tido que adicionar comportamento às suas _views_ através da definição de _Runtime Attributes_ ou mesmo programaticamente, obtendo uma solução "híbrida" de Interface Builder mais código. Em alguns casos é possível que tenha se deparado com toda a sua definição de _view_ sendo escrita através de código (numa prática conhecida como View Code), simplesmente por razões de clareza e praticidade.

Este material teórico tem por objetivo endereçar esta matéria. Pela primeira vez traremos à luz as possíveis razões pelas quais você poderia considerar que o uso de arquivos de Interface Builder pode ser inadequado para determinados contextos.

## Layout

A introdução acima já nos adianta um destes critérios de análise. Nem todas as customizações necessárias para que se atinja as diferentes especificações de design de projetos são suportadas pelo Interface Builder. É comum que técnicas avançadas de layout (e nem sempre tão avançadas assim) requeiram a adição de código às classes de visualização para que sejam possíveis.

Dessa forma, em longo prazo, aplicações acabam por ter sua camada de visualização fragmentada em diferentes pontos do projeto. O que também nos leva à próxima seção.

## Legibilidade e manutenção

<p align="center">
<img alt="Imagem com um arquivo storyboard com uma grande quantidade de representações de view controllers e com fluxos de navegação entre elas" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/viewcode-teoria-nao-usar-storyboards-imagem-storyboards-massivos.jpg?raw=true" width="70%" />
</p>

A imagem acima introduz bem sobre o tópico desta seção. É comum, conforme um projeto cresce, que ele adicione uma série de fluxos de navegação e até mesmo jornadas diversas independentes entre si. Ainda que existam meios para que essa carga seja distribuída em diversos arquivos, é natural que, nessas circunstancias, os arquivos _.storyboards_ escalem para contem a representação de uma infinidade de cenas, cada uma com seus detalhes e diferentes hierarquias de visualizações. Como a introdução deste material teórico relembra, com a distribuição de código de visualização ainda por arquivos de outra natureza, como por exemplo nas próprias classes derivadas de `UIView` e `UIViewController` pela aplicação, esse ponto fica ainda mais evidente.

Em muitos casos desenvolvendo projetos dessas proporções que utilizam o Interface Builder, ao precisar fazer alguma alteração ou _bug fix_ em uma parte da interface com o usuário, é comum se perceber com dificuldades de localizar o ponto específico onde a alteração se deve aplicar ou mesmo ao alternar entre Interface Builder e código (Swift ou Objective-C) que definem sua _view_. O entendimento, manutenção e desenvolvimento da UI rapidamente se torna confuso e propenso a erros.

## Reutilização

O Interface Builder pode tornar mais difícil a criação de _views_ reutilizáveis. Usar _storyboards_ em vez de arquivos _nib_ independentes torna isso ainda mais difícil. Algo tão simples como reutilizar um protótipo de célula `UITableViewCell` em dois _view controllers_, por exemplo, não é possível.

Por vezes, desenvolvedores já são levados a abstrair a definição visual e comportamento de certas _views_ para o código, e reutilizá-los através de arquivos de Interface Builder através da definição de _custom classes_ nas configurações de identidade de objetos de _view_ específicos.

## Trabalhando com _source control_ em grandes equipes: _code review_

Arquivos de Interface Builder não são naturalmente legíveis por seres humanos. Estes arquivos se baseam em uma especificação proprietária de XML que é gerada e interpretada automaticamente pela ferramenta. Este fato impossibilita por exemplo que você revise as alterações que entram no repositório de código do seu projeto com confiança, o que significa que alterações indesejadas ou não intencionais na interface do usuário têm maior risco de acontecer e passarem despercebidas no seu processo.

<p align="center">
<img alt="Imagem com a representação do diff de um Pull Request contendo códigos não naturalmente legíveis a serem humanos dificultando a revisão de código" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/viewcode-teoria-nao-usar-storyboards-imagem-code-review-1.png?raw=true" width="70%" />
</p>

<p align="center">
<img alt="Continuação da imagem com a representação do diff de um Pull Request contendo códigos não naturalmente legíveis a serem humanos dificultando a revisão de código" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/viewcode-teoria-nao-usar-storyboards-imagem-code-review-2.png?raw=true" width="70%" />
</p>

O _diff_ acima de um _Pull Request_ hipotético vai direto ao ponto para ilustrar o problema.

## Trabalhando com _source control_ em grandes equipes: _rebasing_ e _merging_

Apenas para o caso de não termos ainda mencionado: arquivos de Interface Builder não são naturalmente legíveis por seres humanos. Um outro ponto de dor no trabalho com estes arquivos em grandes times com código com controle de versão é o trabalho com a resolução de conflitos.

<p align="center">
<img alt="Imagem com o código xml de base para um storyboard contendo as inserções de indicadores de pontos de conflito de merge" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/viewcode-teoria-nao-usar-storyboards-imagem-conflitos-de-merge-no-xml.png?raw=true" width="70%" />
</p>

É natural que durante o desenvolvimento das funcionalidades das aplicações tenhamos que fazer alterações em pontos de código que outros colegas de equipe também tenham alterado. O Git, por exemplo, já sabe exercer tal controle e nos alertar que uma resolução de conflito precisa ser feita manualmente. No entanto, quando conflitos desta natureza ocorrem em arquivos não naturalmente legíveis, é bastante difícil decidir por qual caminho seguir.

<Imagem com o xcode ignorando a abertura do arquivo>

<p align="center">
<img alt="Imagem com o exemplo de suporte (nenhum) à resolução de conflitos do Interface Builder" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/viewcode-teoria-nao-usar-storyboards-imagem-suporte-xcode-conflitos.png?raw=true" width="70%" />
</p>

A imagem acima mostra o suporte que o XCode Interface Builder nos dá para a resolução de conflitos de merge em um arquivo _storyboard_. 😬

## Ciclo de vida de controladores

O simples fato de a representação - pelo menos visual - dos seus controladores viverem dentro de arquivos _storyboard_ já evidencia que parte do ciclo de vida destes objetos pode estar fora da sua mão. Em alguns casos isso pode não ser de fato um problema, mas em situações específicas é possível que se torne confusa a forma como você injeta as dependências aos controladores que são exibidos durante a execução de fluxos. A decisão de utilizar `UIStoryboardSegue`s ou controlar a apresentação programaticamente pode levar a diferentes métodos de implementação, por exemplo.

## Conclusão

O debate sobre a recomendação (ou não-recomendação) do uso de Interface Builder já é de longa data na comunidade iOS e você agora já tem mais alguns fatos que podem apoiar argumentos. É importante ressaltar que este material teórico não visa indicar a utilização ou não-utilização da prática em projetos, mas sim orientar sobre o que é possível analisar para uma decisão mais adequada. Diferentes contextos de projetos podem indicar diferentes decisões. Ao longo da seção deste material você seguirá com a abordagem de não-utilização para que seja possível treinar a construção de views de maneira programática, algo que tem sido a recomendação geral no desenvolvimento de aplicativos atualmente.
