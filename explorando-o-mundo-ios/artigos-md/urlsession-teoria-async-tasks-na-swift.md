# Async Tasks e Closures na Swift

Aplicações no geral executam uma série de tarefas. Sejam elas mais próximas do contexto de negócio do seu produto ou mais ligadas à operações intrínsecas do sistema, elas são gerenciadas de acordo com uma estratégia de agendamento e execução otimizados. Entender mais detalhes sobre essa estratégia pode significar tirar proveito delas para obter uma experiência aprimorada para o seu aplicativo, ou ter graves problemas de usabilidade. Neste material teórico você será introduzido a estes conceitos através da utilização, como exemplo, de operações de IO potencialmente custosas e operação de UI.

> Esse material teórico foi baseado nas fontes originais disponíveis em: 
>   * https://developer.apple.com/documentation/DISPATCH
>   * https://developer.apple.com/documentation/dispatch/dispatchqueue

## Um pouco de contexto

Na Swift temos algumas formas diferentes de executar código assíncrono _(async)_ - código que não é executado imediatamente como parte do fluxo de controle principal do nosso programa, mas é **despachado** para uma fila de execução diferente - de forma sequencial ou concorrente. Mas antes de seguirmos adiante para conhecer uma das principais formas, é preciso de um pano de fundo para entender a necessidade.

No decorrer da execução de uma aplicação iOS, temos diversas tarefas sendo executadas. Desde a inicialização do processo da aplicação com o carregamento do contexto da sandbox da App, até o carregamento da hierarquia de objetos centrais do UIKit com as janelas e controladores principais, e por fim as operações sob o controle do nosso código customizado. Idealize uma aplicação que ao inicializar, carrega sua _home scene_ e dispara uma operação para carregar dados de um servidor que precisam ser exibidos.

<p align="center">
<img alt="Imagem com o exemplo de fluxo todo executado na main thread, inclusive o carregamento dos dados" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-async-tasks-exemplo-fluxo-main-thread.jpg?raw=true" width="70%" />
</p>

Perceba que uma série de tarefas está fora do nosso controle e são de responsabilidade do próprio sistema com base no funcionamento do UIKit framework. No entanto, a partir do código do `UIViewController` principal da aplicação temos tarefas designadas a partir da nossa necessidade de negócio. O carregamento dos dados junto ao servidor e a posterior atualização da _view_ com esses dados preenche uma parte importante do ciclo de vida da aplicação. Mas sua execução pode trazer certos riscos à experiência.

As operações descritas neste exemplo de fluxo ocorrem em um fila de execução principal do sistema, a popular _main queue_ (que por sua vez representa o fluxo de tarefas que são executadas na thread principal). Na plataforma iOS, essa _queue_ é responsável por executar código que controla a interação com o usuário e toda e qualquer operação de UI. Operações deste tipo só podem ser executadas na _main thread_. No entanto, parte deste fluxo de operações não tem na verdade relação com UI.

Considere o carregamento dos dados junto ao servidor. É esperado que esta operação resulte na maior parte das vezes com um bom tempo de resposta, no entanto uma série de fatores podem inteferir. Imagine um usuário com uma qualidade ruim ou com alguma instabilidade de conexão com a internet momentaneamente. Neste cenário que infelizmente é bastante comum, esta operação de IO pode se mostrar bastante custosa para o sistema. Em decorrência disso, o que pode acontecer com sua aplicação é o bloqueio da thread principal ao esperar pela resposta, impedindo por exemplo a execução de código relativo a interação com a UI, sua responsabilidade central. O app pode congelar a UI até que tenhamos os dados esperados.

Para evitar uma experiência de utilização como essa, e necessário controlar o contexto de execução de cada tarefa entre a _queue/thread_ principal e _queues/threads_ secundárias.

### O _Grand Central Dispatch_

O _Grand Central Dispatch_, ou GCD, é um importante componente da plataforma, fornecendo APIs através das quais é possível executar blocos de código em várias filas diferentes, que podem ser criadas ou simplesmente fornecidas pelo sistema. Ele é uma ótima opção para apoiar a escrita de código assíncrono ou concorrente. Além disso é uma tecnologia de base usada em várias plataformas Apple. Uma das filas fornecidas pelo sistema é a fila principal _(main queue)_, que é a fila na qual todo o nosso código de UI deve ser executado.

Abaixo você pode observar código que agenda a execução de uma tarefa assíncrona ainda no contexto da _main thread_, por exemplo:

``` swift
var posts: [Post]?

DispatchQueue.main.async {
    posts = carregaPosts()
}
```

### As _DispatchQueues_

A classe `DispatchQueue` é uma abstração baseada em fila que pode ser usada para agendar (despachar) blocos de código, ou tarefas, em um contexto de execução isolado e independente. É utilizada para obter referências às filas de execução existentes do sistema ou mesmo para que se construa _queues_ específicas. Uma `DispatchQueue` pode ser utilizada para a execução de tarefas em série ou concorrentes executadas de forma síncrona ou assíncrona.

Você pode obter as referências de _queues_ do sistema através da propriedade `main` (referencia a _thread_ de UI) ou da função estática `global(qos:)`, assim como instanciar uma _queue_ personalizada para gerenciar um contexto de execução específico:

``` swift
var posts: [Post]?

/*
Agenda a execução async do código na main thread
*/
DispatchQueue.main.async {
    posts = carregaPosts()
}
```

``` swift
/*
Agenda a execução async de uma operação custosa para o sistema 
em uma fila secundária do sistema
*/
DispatchQueue.global().async {
    let arquivos = carregaArquivos()
    processa(arquivos)
}
```

``` swift
/*
Define uma fila secundária personalizada com menor prioridade, 
apontada pelo quality of service (qos) .background
*/
let backgroundQueue = DispatchQueue(
    label: "br.com.zup.queue.background", 
    qos: .background
)

// agenda a execução async de bloco de código
backgroundQueue.async { // ou .sync
    // code
}

/*
Define uma fila secundária personalizada capaz de executar 
múltiplas operações paralelas
*/
let queueConcorrente = DispatchQueue(
    label: "br.com.zup.queue.concurrent",
    attributes: .concurrent
)                               
```

Existem diferentes maneiras de se usar _DispatchQueues_, mas talvez o caso de uso mais mais comum entre todos seja o de levar algum processamento pesado ou oneroso para _queues_ seperadas paralelas à _main queue_ para não bloquear a UI e a interação com o usuário.

## Aplicação prática em um cenário comum

Considere um escopo de aplicação onde temos uma tela contendo uma `ImageView` principal posicionada ao centro. É requerido que esta tela apresente o conteúdo de imagem carregado a partir de um servidor, como por exemplo alguma CDN que fornece fotos para uma aplicação qualquer. Para efeitos práticos da explicação, adicione a esta ideia de tela um botão, a partir do qual vamos acionar o dowload.

> Nota: Poderíamos acionar o download a partir do carregamento da _view_ do _View Controller_, mas para ficar mais clara a resposta à execução, o botão será necessario.

Um controlador equivalente poderia ser atingido a partir do código abaixo para esta ideia de tela:

``` swift
class MyViewCodeController: UIViewController {
    
    /**
     Uma URL de uma foto em HD de Júpiter.
     Esta propriedade simula o recebimento de uma URL de uma tela anterior
     */
    let photoURL = URL(string: "https://apod.nasa.gov/apod/image/1906/LDN1773-Jupiter.jpg")!
    // MARK: - Views e Subviews
    
    private lazy var button: UIButton = {
        let button = UIButton(configuration: .filled())
        button.translatesAutoresizingMaskIntoConstraints = false
        button.setTitle("Click to download", for: .normal)
        return button
    }()
    
    private lazy var imageView: UIImageView = {
        let imageView = UIImageView()
        imageView.translatesAutoresizingMaskIntoConstraints = false
        imageView.backgroundColor = .secondarySystemBackground
        imageView.contentMode = .scaleAspectFit
        return imageView
    }()
    
    private lazy var mainView: UIView = {
        let view = UIView()
        view.translatesAutoresizingMaskIntoConstraints = false
        view.backgroundColor = .systemBackground
        return view
    }()
    // MARK: - Métodos do ciclo de vida
    
    override func loadView() {
        super.loadView()
        setup()
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        button.addTarget(self,
                         action: #selector(dowloadImage(_:)),
                         for: .touchUpInside)
    }
    
    /**
     Uma função que adiciona as views de acordo com a hierarquia desejada
     e aplica as constraints de autolayout adequadas
     */
    func setup() {
        self.view = mainView
        
        view.addSubview(imageView)
        
        NSLayoutConstraint.activate([
            imageView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            imageView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            imageView.widthAnchor.constraint(equalToConstant: 350),
            imageView.heightAnchor.constraint(equalToConstant: 350),
        ])
        
        view.addSubview(button)
        
        NSLayoutConstraint.activate([
            button.centerXAnchor.constraint(equalTo: imageView.centerXAnchor),
            button.topAnchor.constraint(equalTo: imageView.bottomAnchor, constant: 24),
        ])
    }
    // MARK: - Action
    
    /**
    Uma função de resposta ao evento de pressionar do botão
     que aciona o dowload da imagem
     */
    @objc func dowloadImage(_ sender: UIButton) {
        imageView.image = try? ImageDowload().execute(for: photoURL)
        print("clicked")
    }
}

// exibe a view no playground 😉 caso queira (imports de UIKit e PlaygroundSupport são requeridos)
PlaygroundPage.current.liveView = MyViewCodeController.init()
```

> Nota: Você pode rodar o código acima em um Swift Playground para ter uma resposta visual do que é construído por este código

O código acima define um controlador para este escopo de tela e já é possível perceber a função de resposta ao botão inferindo para a chamada do carregamento da imagem, seguido de um _log_ informando que o clique ocorreu. No entanto, ainda não existe um módulo definido para o serviço de `ImageDowload`, o que pode estar causando problemas neste momento. 

### Trabalhando com tarefas assíncronas

O código abaixo introduz a ideia de uma classe que oferece como serviço um `ImageDownload`. É possível invocar sua função executa que espera receber uma `URL` e devolve uma `UIImage` ou lança um possível erro. Ele será nosso objeto de estudo desta seção.

``` swift
class ImageDowload {
    func execute(for url: URL) throws -> UIImage {
        do {
            let data = try Data(contentsOf: url)
            
            guard let image = UIImage(data: data) else {
                throw Error.invalidData
            }
            
            return image
            
        } catch let error {
            throw Error.underlyingError(error)
        }
    }
}

extension ImageDowload {
    enum Error: Swift.Error, LocalizedError {
        case underlyingError(Swift.Error)
        case invalidData
        
        var errorDescription: String? {
            switch self {
            case .underlyingError(let error):
                return "Could not possible to load image. \(error.localizedDescription)"
            case .invalidData:
                return "Could not possible to build image. Possible data corruption. Try again!"
            }
        }
    }
}
```

Perceba que este modulo já define seus possíveis erros com causas e mensagens predefinidas. A função `execute(for:)` tenta obter os dados para a URL junto ao servidor através de uma requisição HTTP. A partir da chamada, com os dados em mãos construímos a UIImage a partir dos dados recebidos do servidor, ou lançamos os erros.

No código do _View Controller_, por questões de brevidade, apenas invocamos a função utilizando `try?`, já que a recuperação de uma possível falha de download não é de interesse deste material teórico.

Ao executar o código e interagir com o exemplo, o clique do botão já indica o que pode estar acontecendo.

<p align="center">
<img alt="Gif animado demonstrando a execução do exemplo acima, que até então bloqueia a main thread e a experiência do usuário" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-async-tasks-block-da-main-thread.gif?raw=true" width="100%" />
</p>

Veja que ao pressionar o botão, o mesmo sequer termina de executar sua resposta visual, permanecendo no estado pressionado (com seu background em cor mais clara que o habitual). Um outro indicativo pode ser a execução do _log_ no console, a ideia era avisar o usuário que o evento ocorreu. Tudo isso, no entanto, aguarda a execução do request e só após seu término, continua seu trabalho.

Todo o trabalho atual esta sendo realizado na _main queue_ (lê-se também, _main thread_) que está sendo bloquada e impedida de realizar sua responsabilidade, processar as tarefas de interação com o usuário e manter a UI atualizada e respondendo. Essa deve ser sua prioridade máxima.

Processamentos como o que o `ImageDownload` executa devem ser levado para longe da _thread_ de UI, e, preferencialmente, sendo executados de forma _async_ em prioridade mais baixa. Tudo para favorecer a responsabilidade principal da aplicação, o fluxo de atualização e resposta da UI. 

Podemos colocar nosso conhecimento adquirido sobre o GCD para jogo neste exemplo de download de imagem. `DispatchQueue.global().async { ... }` habilita exatamente o que precisamos:

``` swift
class ImageDowload {
    func execute(for url: URL) throws -> UIImage {
        DispatchQueue.global().async {
            do {
                
                let data = try Data(contentsOf: url)
                
                guard let image = UIImage(data: data) else {
                    throw Error.invalidData
                }
                
                return image
            } catch let error {
                throw Error.underlyingError(error)
            }
        }
    }
}
```

O problema é que este código, da forma como está, apresenta problemas em tempo de compilação.

```
Cannot convert return expression of type '()' to return type 'UIImage'

Invalid conversion from throwing function of type '() throws -> Void' to non-throwing function type '@convention(block) () -> Void'
```

Não se assuste com as indicações de erro do compilador. Elas apenas estão nos dizendo que o Swift inferiu `() throws -> Void` como tipo de retorno para o metodo `execute(for:)`, entretanto, nossa assinatura indica o retorno de uma simples `UIImage`.

Da forma como pretendemos executar a função não podemos manter a ideia de uma saída/retorno simples dessa execução. E faz todo o sentido mudarmos a abordagem. Imagine a execução de acordo com o que temos acima. O código do controlador invocaria a função de download, e, como a mesma roda em um _thread_ separada e com contexto assíncrono, seguiria a vida normalmente (respondendo a UI e logando no console) sem aguardar sua execução (que como vimos, pode levar um tempo). Dessa forma, como teríamos a atribuição do retorno do método, já que o contexto de execução da função `dowloadImage(_:)` (action do botão) se encerrou?

Isso explica por que a maior parte das APIs assíncronas que utilizamos não suportam esse padrão de comunicação em suas chamadas. Precisamos de outra forma de estabeler essa comunicação.

### O padrão Closure Callback

Sabemos que a execução do download pode demorar mais que o esperado, ainda mais considerando possíveis padrões ruins de conexão de internet pelos usuários. Esse foi o principal motivador para levar o código da _main queue_, para um _global queue_ de menor prioridade executando código _async_.

Não sabemos ao certo quando a execução da tarefa assíncrona se encerra, nem como. Sucesso? Ou erro? Não temos portanto como esperar um retorno ou _throw_ direto de função. Precisamos de uma outra abordagem, que suporte que, ao final da execução async, quando quer que seja, execute o final do trabalho por nós. Algo como se o _View Controller_ chamasse a execução da _async task_, mas também já combinasse com o executor desta tarefa o que deve ser feito ao final dela. Essa abordagem é bastante conhecida: os populares _callbacks_.

<p align="center">
<img alt="Imagem com exemplo de modelo de comunicação async com o padrão callback" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-async-tasks-exemplo-modelo-callbacks.jpg?raw=true" width="70%" />
</p>

Podemos passar adiante uma função que será executada ao final do processamento, retornando ao contexto inicial de trabalho (chamada de volta, _callback_), atualizando a `ImageView`. Utilizando os recursos aprendidos da linguagem Swift, podemos fazê-lo de maneira bastante simples com as closures.

``` swift
class ImageDowload {
    func execute(for url: URL,
                 completionHandler: @escaping (UIImage) -> Void,
                 failureHandler: @escaping (ImageDowload.Error) -> Void) {
        DispatchQueue.global().async {
            do {
                
                let data = try Data(contentsOf: url)
                
                guard let image = UIImage(data: data) else {
                    failureHandler(.invalidData)
                    return
                }
                
                completionHandler(image)
            } catch let error {
                failureHandler(.underlyingError(error))
            }
        }
    }
}
```

Perceba que agora a função não mais se preocupa em retornar a `UIImage` ou mesmo lançar um erro. Todo o bloco de código está sendo executado em um contexto assíncrono. Ao final da execução, teremos a chamada da função adequada para lidar com a completude do trabalho.

``` swift
class MyViewCodeController: UIViewController {
    
    // código anterior omitido
    
    /**
    Uma função de resposta ao evento de pressionar do botão
     que aciona o dowload da imagem
     */
    @objc func dowloadImage(_ sender: UIButton) {
        ImageDowload().execute(for: photoURL) { [weak self] loadedImage in
            self?.imageView.image = loadedImage
            
        } failureHandler: { error in
            print(error.localizedDescription)
        }

        print("clicked")
    }
}
```

> Nota: Consulte o material teórico sobre Closures caso não esteja familiarizado com a utilização das mesmas.
>
> * [Closures na Swift](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/artigos-md/urlsession-teoria-closures-na-swift.md)

### A _global_ o que é de _global_, e a _main_ o que é de _main_

Caso você esteja testando a execução ao ler este texto, pode estar se questionando neste momento por qual razão as alterações aplicadas acima não resultaram como esperado. A aplicação se encerra antes de completar o trabalho em estado de erro.

<p align="center">
<img alt="Gif animado demonstrando a execução do exemplo acima, já trabalhando em uma thread secundária, porém ainda com problemas" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-async-tasks-atualizacao-ui-fora-da-main.gif?raw=true" width="100%" />
</p>

A causa do problema atual é descrita com exatidão pela própria imagem de exemplo do modelo de comunicação com _callbacks_ apresentada acima. Vejamos novamente.

<p align="center">
<img alt="Imagem com exemplo de modelo de comunicação async com o padrão callback" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-async-tasks-exemplo-modelo-callbacks.jpg?raw=true" width="70%" />
</p>

Perceba que a ideia de levar também o código que deve ser executado ao final (com as closures) pode indicar o problema. Por mais que, como vimos, as closures capturam o contexto adequado do _View Controller_ para que sua execução em outro ponto seja possível, elas não levam consigo a informação da _queue_ onde estão rodando. Logo, quando da sua chamada ao final da execução, código de UI (responsabilidade definida no _controller_) será executado em um _thread_ secundária. Isso quebra o design proposto pela própria arquitetura da plataforma e causa o erro.

O que precisamos é garantir que o código das funções de _callback_ sejam executados de volta ao contexto da _main queue_. Algo como na imagem abaixo:

<p align="center">
<img alt="Imagem com exemplo de modelo de comunicação async com o padrão callback" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-async-tasks-exemplo-modelo-callbacks-executados-na-main.jpg?raw=true" width="70%" />
</p>

Novamente, utilizando o conhecimento sobre o GCD, podemos supor como resolver. Podemos utilizar mais uma vez a API de `DispatchQueue` para inferir a execução da chamada dos _callbacks_ na _main queue_ com `DispatchQueue.main.async { ... }`.

``` swift
class ImageDowload {
    func execute(for url: URL,
                 completionHandler: @escaping (UIImage) -> Void,
                 failureHandler: @escaping (ImageDowload.Error) -> Void) {
        DispatchQueue.global().async {
            do {
                
                let data = try Data(contentsOf: url)
                
                guard let image = UIImage(data: data) else {
                    DispatchQueue.main.async { failureHandler(.invalidData) }
                    return
                }
                
                DispatchQueue.main.async { completionHandler(image) }
            } catch let error {
                DispatchQueue.main.async { failureHandler(.underlyingError(error)) }
            }
        }
    }
}
```

Com a alteração, podemos ver que chegamos ao resultado esperado.

<p align="center">
<img alt="Gif animado demonstrando a execução do exemplo acima trabalhando em uma thread secundária e com o correta troca de contexto, voltando pra main thread pra realizar a atualização da tela" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-async-tasks-atualizacao-ui-na-main.gif?raw=true" width="100%" />
</p>

Para evitar ainda alguma dificuldade no reaproveitamento do código de `ImageDownload`, caso necessário, podemos tornar flexível a _queue_ onde as tarefas de completude serão executadas, expondo um parêmetro na definição da função `execute(for:completionHandler:failureHandler)`.

``` swift
class ImageDowload {
    func execute(for url: URL,
                 completeOn completionQueue: DispatchQueue = .main,
                 completionHandler: @escaping (UIImage) -> Void,
                 failureHandler: @escaping (ImageDowload.Error) -> Void) {
        DispatchQueue.global().async {
            do {
                
                let data = try Data(contentsOf: url)
                
                guard let image = UIImage(data: data) else {
                    completionQueue.async { failureHandler(.invalidData) }
                    return
                }
                
                completionQueue.async { completionHandler(image) }
            } catch let error {
                completionQueue.async { failureHandler(.underlyingError(error)) }
            }
        }
    }
}
```
