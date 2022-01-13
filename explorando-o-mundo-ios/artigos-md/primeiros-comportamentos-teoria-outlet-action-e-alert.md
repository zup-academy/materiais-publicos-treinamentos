# Entendendo IBOutlets

>Esse material teórico foi atualizado tendo como base a fonte original sobre IBOutlets em https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/Outlet.html.

Um _outlet_ é uma propriedade armazenads que é anotada com `@IBOutlet` e cujo valor você pode definir graficamente em um arquivo Storyboard (ou arquivo interface builder equivalente). Você declara um _outlet_ em uma classe e faz uma conexão entre o _outlet_ e outro objeto no storyboard. Quando a _view_ referente a um _View Controller_ no arquivo Storyboard é carregada, a conexão é estabelecida.

<p align="center">
<img alt="uma imagem com um diagrama ilustrando a relação entre o outlet como propriedade armazenada de um View Controller e o objeto View declarado no Interface Builder conectado ao mesmo" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/primeiros-comportamentos-teoria-outlets-imagem-vc-outlet.jpg?raw=true" width="60%" />
</p>

Você define um _outlet_ como uma propriedade armazenada com a anotação `@IBOutlet`:

``` swift
@IBOutlet var loginTextField: UITextField!
```

A anotação `IBOutlet` é usado apenas pelo Xcode, para determinar quando uma propriedade é um _outlet_, não tendo valor real para o código da classe onde se encontra.

Por meio de um _outlet_, um objeto em seu código pode obter uma referência a um objeto definido em um arquivo Storyboard e, em seguida, pode ser carregado tendo como base as definições desse arquivo. O objeto que contém um _outlet_ geralmente é um objeto _View Controller_ customizado. Você frequentemente define _outlets_ para poder enviar mensagens para objetos de _View_ do UIKit framework. 

Um _outlet_ pode ser especialmente útil para evitar o trabalho de manipulação das referências das _subviews_ da _root view_ (em _View Controllers_) em uma hierarquia complexa. Provendo uma referência direta a um objeto _View_ muitos níveis abaixo na árvore de componentes de uma tela, um _outlet_ facilita o código necessário para que um _View Controller_ configure dinamicamente o estado inicial correto para um objeto _TextField_. Como no exemplo ilustrado acima, é possível de forma direta solicitar que um campo de texto seja focado a partir do carregamento da _view_ de uma tela.

## Conectando IBOutlets

# Entendendo IBActions

Explicação IBAction

## Conectando IBActions

Exemplo de como conectar IBOutlets

# Dialogos com o usuário: UIAlert

Exemplo de como exibir em alert ao invés de `print()`