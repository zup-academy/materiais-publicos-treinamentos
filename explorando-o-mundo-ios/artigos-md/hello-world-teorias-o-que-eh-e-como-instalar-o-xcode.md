# O que é e como instalar o Xcode?

O Xcode é a ferramenta padrão para desenvolvimento de aplicações para as plataformas _macOS_ e _iOS_. Se você deseja criar aplicativos para Mac, iPhone ou iPad, o Xcode é o lugar para começar. Mas o que exatamente ele faz?

O Xcode não é exatamente **uma** ferramenta, mas uma coleção delas - o que é conhecido como ambiente de desenvolvimento integrado (_Integrated Development Environment_ - IDE). A palavra "integrado" é a chave aqui: o Xcode integra tudo o que você precisa para desenvolver um aplicativo em um único ambiente organizado.

O editor de código-fonte do Xcode oferece preenchimento automático inteligente conforme você digita, enquanto o _color coding_ automático torna seu código o mais fácil de ler possível, independentemente da linguagem de programação.

<p align="center">
<img alt="Imagem com exemplo da aplicação Xcode, demonstrando o organizador de arquivos, o editor de código, o preview do resultado da tela do app sendo desenvolvido e o helper da ferramenta auxiliando na consulta de documentação" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/hello-world-teoria-xcode-exemplo-editor-preview-e-helper.png?raw=true" width="70%"/>
</p>

O modelo de grupos, pastas e subpastas do Xcode permite que você aninhe os arquivos com a profundidade que for necessária. O Xcode também lida com qualquer tipo de arquivo que você abra através do seu editor, de imagens a JSON e plists.

Além de prover uma forma bem estruturada de organização de arquivos e o editor de código fonte, o Xcode também oferece um Interface Builder, que ajuda a definir a UI dos seus aplicativos usando uma variedade de ferramentas. Ele também oferece mecanismos para auxiliar a conexão de todos os elementos de interface ao seu código-fonte. Esse recurso permite que você crie um protótipo e, de forma prática, preencha todo o código para dar vida a esses elementos de interface.

Na base do Xcode também está um conjunto de compiladores e ferramentas de build, designados para identificar bugs e sugerir alterações para ajudá-lo a colocar seus projetos em funcionamento adequadamente. O _debugger_ gráfico serve como a ferramenta para identificar problemas com seu código e ajudá-lo a corrigi-los. É bastante simples utilizá-lo para alterar o valor das variáveis ​​em tempo de execução, avaliar expressões e definir pontos de interrupção que param seu programa em um ponto específico permitindo uma execução controlada e inspeção do funcionamento.

<p align="center">
<img alt="Imagem com exemplo do memory graph debugger do Xcode que auxilia a pessoa que desenvolve a identificar problemas de gestão de referências à objetos e por sua vez, de alocação de memória" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/hello-world-teoria-xcode-memory-graph-debugger.png?raw=true" width="70%"/>
</p>

<p align="center">
<img alt="Imagem com exemplo do view hierarchy debugger do Xcode que auxilia a pessoa que desenvolve a identificar as camadas dos componentes de visualização desenhados na tela de forma interativa" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/hello-world-teoria-xcode-view-hierarchy-debugger.png?raw=true" width="70%"/>
</p>

O Xcode oferece ainda muitas outras ferramenta e tem um sistema de ajuda que torna fácil fazer consultas à documentação e executar _snippets_ de código de exemplo.

## Como obter e instalar o Xcode?

Como o Xcode é a ferramenta oficial de desenvolvimento para as plataformas Apple, instalá-lo não poderia ser mais fácil. O download e instalação estão disponíveis na aplicação App Store do macOS.

### Passos para a instalação

1. Certifique-se de ter pelo menos 13 GB livres em disco antes de tentar a instalação.

    * Clique no menu com ícone da Apple para selecionar _"Sobre este Mac”_ (_About This Mac_).
    * Clique na aba "Armazenamento" (_Storage_).

1. Em um navegador, acesse o Mac App Store Preview para a aplicação do Xcode. Você pode fazer isso através do link https://apps.apple.com/us/app/xcode/id497799835?mt=12.

    * Clique em "Ver na App Store" (_View in Mac App Store_), e selecione "Abrir App Store.app” (_Open App Store.app_) no _pop-up_.

        Você também pode optar por abrir a aplicação App Store manualmente e buscar por "xcode" através da ferramenta de pesquisa no canto superior esquerdo.

    <p align="center">
    <img alt="Imagem com exemplo da pesquisa por 'xcode' na app store app" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/hello-world-teoria-xcode-instalacao-do-xcode-atraves-da-mac-app-store.jpg?raw=true" width="70%"/>
    </p>

1. No canto superior direito da página de detalhes da aplicação Xcode - ou da view com o resultado da pesquisa - encontra-se o botão para obter a aplicação. Clique em "Obter" (_Get_).

    i. (Opcional) Caso solicitado, forneça seu Apple ID e sua senha e/ou passe pelo processo de autenticação para a obtenção e download da aplicação - se você ainda não tiver um, terá que passar pelas etapas de criação de um Apple ID pela aplicação da App Store.

    >Nota: Evite a instalação de versões beta do Xcode por hora. Você provavelmente ainda não precisa se preocupar com as features mais recentes que estão sendo lançadas para as ferramentas de desenvolvimento em fase de testes.

    >Nota: Este treinamento foi construído usando como base o Xcode na **versão `13.2`**. Embora ainda exista a possibilidade de realizar o treinamento utilizando versões anteriores, é importante estar atento à questões de compatibilidade. Existem dois pontos principais que podem afetar a experiência de consumo do treinamento caso sua versão seja anterior à mencionada: (a) Nem todas as features de Swift utilizadas no código do treino podem funcionar e (b) A UI da IDE pode não ser exatamente a mesma que você verá nos vídeos e imagens disponibilizados nos materiais de suporte e atividades.

    ii. Clique no ícone de download da aplicação e aguarde - faça um café, isso pode levar algum tempo.


1. Quando tudo terminar, clique no botão "Abrir" (_Open_) para abrir a aplicação. Alternativamente, você pode procurar por Xcode.app na pasta `/Applications` usando o Finder.
