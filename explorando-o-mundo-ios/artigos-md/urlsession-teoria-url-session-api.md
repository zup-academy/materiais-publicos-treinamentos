# A API de URLSession

Aplicações no geral executam uma série de chamadas para obter dados de servidores através da rede. Este material introduz a API de URLSession, através da qual é possível executar requisições HTTP para consumir serviços web através da rede.

> Esse material teórico foi baseado na fonte original disponível em https://developer.apple.com/documentation/foundation/urlsession

## URLSession

Um objeto que coordena um grupo de tarefas de transferência de dados pela rede relacionadas.

#### Declaração

``` swift
class URLSession : NSObject
```

#### Visão geral

A classe `URLSession` e as classes relacionadas fornecem uma API para download e upload de dados para _endpoints_ indicados por `URL`s. Seu aplicativo também pode usar essa API para realizar downloads em _background_ quando seu aplicativo não estiver em execução ou enquanto seu aplicativo estiver suspenso. Você pode usar `URLSessionDelegate` e `URLSessionTaskDelegate` relacionados para dar suporte à autenticação e receber eventos como redirecionamento e conclusão de tarefas.

> Nota: A API URLSession envolve muitas classes diferentes que funcionam juntas de uma maneira bastante complexa, o que pode não ser óbvio ao ler apenas esta documentação de referência por si só. Mais a frente temos adições a este material que visam facilitar a associação deste conceito

Seu aplicativo cria uma ou mais instâncias de `URLSession`, cada uma coordenando um grupo de tarefas de transferência de dados relacionadas. Por exemplo, se você estiver criando um navegador da Web, seu aplicativo poderá criar uma sessão por guia ou janela ou uma sessão para uso interativo e outra para downloads em _background_. Dentro de cada sessão, seu aplicativo adiciona uma série de tarefas, cada uma representando uma requisição para uma URL específica (seguindo redirecionamentos HTTP, se necessário).

### Tipos de URLSession

As tarefas em uma determinada `URLSession` compartilham um objeto de configuração de sessão comum, que define o comportamento da conexão, como o número máximo de conexões simultâneas a serem feitas para um único host, se as conexões podem usar a rede celular e assim por diante.

URLSession tem um singleton, uma sessão compartilhada, que não tem um objeto de configuração para requisições básicas. Não é tão personalizável quanto as sessões que você cria e configura, mas serve como um bom ponto de partida se você tiver requisitos muito limitados. Você acessa essa sessão chamando o método de classe compartilhada. Para outros tipos de sessões, você cria uma URLSession com um dos três tipos de configurações:

* Uma sessão padrão se comporta de maneira muito semelhante à sessão compartilhada, mas permite que você a configure. Você também pode atribuir um _delegate_ à sessão padrão para um ajuste mais fino;

* As sessões efêmeras são semelhantes às sessões compartilhadas, mas não gravam caches, cookies ou credenciais no disco;

* As sessões em _background_ permitem que você faça uploads e downloads de conteúdo em segundo plano enquanto seu aplicativo não está em execução.

### Tipos de tarefas de URLSession _(URLSessionTask)_

Em uma sessão, você cria tarefas que opcionalmente carregam dados em um servidor e, em seguida, recuperam dados do servidor como um arquivo no disco ou como um ou mais objetos `Data` na memória. A API URLSession fornece quatro tipos de tarefas:

* Tarefas de dados _(data tasks)_ enviam e recebem dados usando objetos `Data`. Tarefas de dados destinam-se a requisições simples e geralmente interativas para um servidor. Como um _get_ para obter uma lista de valores, ou _post_ para criar um recurso no servidor. A maior parte das tarefas que você irá criar serão deste tipo;

* As tarefas de upload _(upload tasks)_ são semelhantes às _data tasks_, mas também enviam dados (geralmente na forma de um arquivo) e suportam uploads em _background_ enquanto o aplicativo não está em execução;

* As tarefas de download _(download tasks)_ recuperam dados na forma de um arquivo e suportam downloads e uploads em _background_ enquanto o aplicativo não está em execução;

* Tarefas WebSocket _(websocket tasks)_ trocam mensagens por TCP e TLS, usando o protocolo WebSocket definido na [RFC 6455](https://tools.ietf.org/html/rfc6455).

### Usando um _session delegate_

As tarefas em uma sessão também compartilham um objeto _delegate_ comum. Você implementa este _delegate_ para fornecer e obter informações quando vários eventos ocorrem, incluindo quando:

* Uma autenticação falha;

* Os dados chegam do servidor;

* Os dados ficam disponíveis para armazenamento em cache;

Se você não precisar dos recursos fornecidos por um _delegate_, poderá usar essa API sem fornecer um passando `nil` ao criar uma sessão.

> Nota importante: 
>
>O objeto de sessão mantém uma referência forte ao _delegate_ até que seu aplicativo saia ou invalide explicitamente a sessão. Se você não invalidar a sessão, seu aplicativo terá um vazamento de memória até que o aplicativo seja encerrado.

Cada tarefa que você cria com a sessão chama de volta os métodos do _session delagate_, usando os métodos definidos em `URLSessionTaskDelegate`. Você também pode interceptar esses retornos de chamada antes que eles cheguem ao _delegate_ de sessão preenchendo um _delegate_ separado específico para a tarefa.

### Assincronicidade e URLSession

Como a maioria das APIs de _networking_, a API URLSession é altamente assíncrona. Ele retorna dados para seu aplicativo de três maneiras, dependendo dos métodos que você chama:

* Em Swift ou Objective-C, você pode fornecer um bloco _completion handler_, que é executado quando a transferência é concluída.

* Em Swift ou Objective-C, você pode receber retornos de chamada para um método de _delegate_ à medida que a transferência avança e imediatamente após sua conclusão.

Além de fornecer essas informações aos _delegates_, a URLSession fornece propriedades de status e progresso. Consulte essas propriedades se precisar tomar decisões programáticas com base no estado atual da tarefa (com a ressalva de que seu estado pode mudar a qualquer momento).

### Thread Safety

A API de URLSession é _thread-safe_. Isso quer dizer que você pode criar sessões e tarefas livremente em qualquer contexto de _thread_. Quando seus métodos de _delegate_ chamam os _completion handlers_ fornecidos, o trabalho é agendado automaticamente na _queue_ correta.

## Entendendo a API

A introdução oferecida pela documentação nos ajuda a ter uma visão geral da organização da API de URLSession, mas como ela mesmo cita, não aprofunda para que seja possível vê-la em ação. Nesta seção é o que faremos. 

### Uma revisão

_URLSession_ é ao mesmo tempo uma classe e um grande grupo de objetos relacionados dentro da organização de sua API, dos quais podemos tirar proveito para construir código que faz o carregamento de dados. A classe nos dá a representação de uma sessão, através da qual podemos processar diversas operações (_tasks_) em seu contexto.

Para obter uma URLSession devidamente configurada para uso apropriado, é possível se utilizar da classe URLSessionConfiguration. Com uma URLSession em mãos, é possível criar tarefas através de seus métodos utilitários. Por sua vez, para que seja possível responder ao processamento das operações (assíncronas por definição) é possível se utilizar de simples _completionHandlers_, ou mesmo implementar um _delegate_ para um maior nível de controle sobre o processamento. 

<p align="center">
<img alt="Imagem com relação entre os objetos centrais da API de URLSession" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-urlsession-api-imagem-urlsession.jpeg?raw=true" width="70%" />
</p>

A imagem acima ilustra a organização dos objetos centrais dessa API.

A implementação de base em URLSessionConfiguration permite obter o objeto de sessão com todas as predefinições mais adequadas para o contexto de uso necessário. No entanto para processar tarefas simples, a API de URLSession já define uma instância compartilhada (_singleton_) de sessão com configuração padrão disponível na propriedade `shared` estática. Para grande parte das requisições simples de uma aplicação esta configuração pode ser suficiente.

<p align="center">
<img alt="Imagem com a relação entre os objetos centrais da API com foco na utilização do singleton shared session" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-urlsession-api-imagem-urlsession-shared.jpeg?raw=true" width="70%" />
</p>

Ao utilizarmos a URLSession _shared_, não podemos nos apoiar nos objetos de configuração, tampouco na implementação de _delegates_ para controlar as operações dessa sessão. A imagem acima ilustra essa relação.

O objeto de URLSession nos permite criar e executar diversos tipos de operações. Desde simples carregamentos de dados em memória (abordagem da maior parte dos requests para APIs REST), até upload e download de arquivos em disco (estes últimos com a possibilidade de execução em _background_).

<p align="center">
<img alt="Imagem com a relação entre o objeto url session e os diversos tipos de tarefa que ele nos permite criar, com foco nas data tasks" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-urlsession-api-imagem-urlsessiontask.jpeg?raw=true" width="70%" />
</p>

A imagem acima ilustra as diferenças entre possíveis URLSessionTasks.

> Nota: Neste material teórico utilizaremos a _shared url session_ executando _data tasks_ para um primeiro contato mais direcionado ao consumo de APIs REST simples.

### Aplicando em um exemplo

Nesta seção trataremos de colocar em prática o conteúdo da anterior. Através de alguns trechos de código, trabalharemos com a URLSession a partir da instância compartilhada para construir _data tasks_ que carregam representações de dados em JSON junto a servidores HTTP.

Para isso, utilizaremos o contexto da ITunes Search API, da própria Apple. Através de uma simples requisição HTTP de tipo GET, vamos pedir pelos dados de músicas dado um determinado termo de busca. A resposta da API será como o exemplo de JSON abaixo:

``` json
{
    "results": [
        {
            "trackName": "Nome da música 1",
            "artistName": "Nome do artista",
            "previewUrl": "https://link-do-preview.m4a"
        },
        {
            "trackName": "Nome da música 2",
            "artistName": "Nome do artista",
            "previewUrl": "https://link-do-preview.m4a"
        }
    ]
}
```

Para termos uma representação equivalente aos dados retornados, definiremos uma _struct_ `Song`:

``` swift
struct Song: Decodable {
    let name: String
    let artist: String
    let previewURL: URL
    
    enum CodingKeys: String, CodingKey {
        case name = "trackName"
        case artist = "artistName"
        case previewURL = "previewUrl"
    }
}
```

Para que seja possível já ter uma representação do objetos que agrupa os resultados, além de uma forma de exibir os dados de um objeto de `Song` como uma simples `String`, podemos lançar mão das `extensions` abaixo:

``` swift
extension Song {
    struct Response: Decodable {
        let results: [Song]
    }
}

extension Song: CustomStringConvertible {
    var description: String {
        return "{\nname: \"\(name)\",\nartist: \"\(artist)\",\npreview: \"\(previewURL.absoluteString)\"\n}"
    }
}
```

> Nota: Você pode se utilizar do Swift Playground para executar o código desta seção.

Com a representação para o objeto de `Song` pronta, podemos introduzir um recorte de uma classe para representar o trabalho com a _search API_ do ITunes.

``` swift
class ITunesSearchAPI {
    
    let baseUrl: String = "https://itunes.apple.com/search"
    
    var session: URLSession
    var dataTask: URLSessionDataTask?
    
    init(with session: URLSession = .shared) {
        self.session = session
    }
    
    // precisamos implementar a função que efetua o request aqui

    private func createURL(for baseURL: String, and term: String) -> URL {
        var urlComponents = URLComponents(string: "\(baseURL)?")!

        urlComponents.queryItems?.append(.init(name: "media", value: "music"))
        urlComponents.queryItems?.append(.init(name: "entity", value: "song"))
        urlComponents.queryItems?.append(.init(name: "term", value: term))
        
        return urlComponents.url!
    }
}
```

A classe acima já nos traz alguma funcionalidade. Primeiro, ela define propriedades armazenadas para lidar com a _session_ e com a _dataTask_ que criaremos, assim como um inicializador para construir o objeto em um estado adequado. O inicializador oferece uma valor padrão para configurar a _session_ a partir da instância compartilhada caso nenhuma outra seja passada como argumento. Por hora também, a propriedade para a _data task_ prevê a possibilidade de nulidade. Voltaremos a trabalhar com ela em breve.

Para além das propriedades e inicializador, a classe também introduz uma constante com a URL base para o request, além de uma função utilitária que nos ajuda a obter os parâmetros de requisição já com o encoding adequado para cada uma. (Onde `"zeca pagodinho"`, por exemplo, se torna `"zeca%20pagodinho"`). Ao construir uma URL com a função para o exemplo, devemos obter `"https://itunes.apple.com/search?media=music&entity=song&term=zeca%20pagodinho"`.

#### Executando o request

Para que seja possível que a classe execute o request, primeiro definiremos uma função para oferecer o recurso em sua interface.

``` swift
class ITunesSearchAPI {
    
    // código anterior omitido
    
    func getSongs(for term: String,
                  completionHandler: @escaping ([Song]) -> Void) {        
        
        let url = createURL(for: baseUrl, and: term)
        
        let dataTask = session.dataTask(with: url) { data, response, error in
            // data? representa os possíveis dados recebidos do servidor
            // response? representa os metadados da resposta para a requisição
            // error? representa o possível erro que ocorreu ao tentar efetuar a requisição

            // código deve ir aqui
        }
        
        dataTask.resume()
    }
    
    // código posterior omitido
}
```

Perceba que já adicionamos a função `getSongs` que recebe, além do termo de busca, uma closure para que seja possível executar algum código ao final do processamento do request, por exemplo, para obter a informação da lista de músicas (lembrando da assincronicidade da API). Além disso, a função já tira proveito da API de URLSession para construir uma simples _data task_ através da função `dataTask(with:completionHandler:)`. A presença da chamada `dataTask.resume()` se dá pela necessidade de acionar a execução da tarefa em uma _thread_ secundária, já que a invocação da função `dataTask(with:completionHandler:)` apenas constroi uma nova tarefa (como se fosse um _factory method_).

Precisamos agora lidar com o código que opera sobre o resultado da requisição. Mas antes disso, já podemos adicionar mais controle sobre a execução das tarefas. O que aconteceria se chamassemos a função `getSongs(for:completionHandler:)` várias vezes em sequência, antes de recebermos uma resposta, por exemplo? A resposta é, criariamos uma série de requisições em segundo plano, provavelmente desnecessárias. (Esse caso de uso poderia ocorrer por exemplo a partir de funcionalidade de refresh em alguma tela por exemplo, somada a alguma conexão de rede mais lenta). Deveríamos evitar isso.

``` swift
class ITunesSearchAPI {
    
    // código anterior omitido
    var dataTask: URLSessionDataTask?

    // código omitido
    
    func getSongs(for term: String,
                  completionHandler: @escaping ([Song]) -> Void) {
        dataTask?.cancel()
        
        let url = createURL(for: baseUrl, and: term)
        
        dataTask = session.dataTask(with: url) { [weak self] data, response, error in
            defer {
                self?.dataTask = nil
            }
            
            // código vai aqui
        }
        
        dataTask?.resume()
    }
    
    // mais código omitido
}
```

Veja que nos apoiamos na propriedade armazenada para guardar a referência para a _data task_. Dessa forma é possível usar o suporte ao cancelamente da tarefa, caso alguma já exista e esteja em execução. Note também a necessidade de sobrescrever a referência com `nil` ao final do processamento, para uma boa gestão do estado neste modulo. O uso de `defer` posterga a execução do código para o final da escopo, então temos por garantido que no fim do processamento a referência para a _data task_ será destruída.

Outro ponto importante a se notar neste código é o uso da lista de captura da closure _(capture list)_. É muito importante que a closure passada adiante não capture o contexto de `self`, necessário explicitamente para referenciar a _data task_, com uma referência forte. Isso poderia aumentar o risco de obtermos um _retain cycle_ e possíveis vazamentos de memória.

> Nota: Na seção onde conhecemos o padrão Delegate vimos a explicação teórica de um _retain cycle_ e pode valer uma rápida revisão, caso ele não esteja vindo à mente no momento.

Podemos seguir adiante para o código que opera sobre o resultado.

``` swift
class ITunesSearchAPI {
    
    // código omitido
    
    func getSongs(for term: String,
                  completionHandler: @escaping ([Song]) -> Void) {
        dataTask?.cancel()
        
        let url = createURL(for: baseUrl, and: term)
        
        dataTask = session.dataTask(with: url) { [weak self] data, response, error in
            defer {
                self?.dataTask = nil
            }
            
            // 1
            if let error = error {
                print(error.localizedDescription)
                return
            }
            
            // 2
            guard let data = data,
                  let response = response as? HTTPURLResponse,
                  response.statusCode == 200 else {
                print("Fail to receive data")
                return
            }
            
            // 3
            do {
                let songsResponse = try JSONDecoder().decode(Song.Response.self, from: data)
                completionHandler(songsResponse.results)
                
            } catch let error {
                print(error.localizedDescription)
            }
        }
        
        dataTask?.resume()
    }
    
    // mais código omitido
}
```

Bastante coisa foi adiciona neste código, mas no geral todas elas são simples de entender. Os marcadores via comentário nos ajudam a dividir as instruções em três grandes blocos:

1. A primeira coisa a se fazer é verificar se tivemos um problema ao processar a requisição. Idealmente deveríamos sempre ter a resposta com os dados, mas na prática existem diferentes razões pelas quais podemos ter problemas. Um ponto importante a se notar sobre o trabalho com o `error` é que, no contexto de uma resposta HTTP, não é aqui que temos notificado, por exemplo, um erro do servidor (HTTP Response Status 500) ou um erro do cliente por não existir o recurso no servidor (HTTP Response Status 404), isso fica por conta do passo a seguir;

1. Depois de assegurar que não sofremos com algum erro que tenha nos impedido de processar a requisição, é hora de analisar os metadados da response HTTP para descobrirmos como o servidor processou nossa _request_. É aqui que podemos descobrir erros comuns definidos pela especificação através dos _status codes_. Caso tudo tenha corrido bem (HTTP Response Status 200) temos então os dados em mãos e podemos prosseguir com o trabalho;

1. Ao chegar a este ponto, temos um resposta de sucesso e os dados que vieram na resposta. É hora então de deserializar os dados JSON e obter a representação através de instâncias de `Song`. Em qualquer caso não esperado, seja por erro ao realizar o request, ao receber a resposta, ou fazer o _parse_ dos dados, a implementação apenas imprime a informação do erro. Em um cenário real, você provavelmente gostaria de ter uma _handler_ também para lidar com uma resposta adequada ao usuário nestes casos.

Com essa implementação já podemos processar nossa _request_.

> Nota: Caso você esteja testando esse código no Swift Playground será preciso adicionar a instrução `PlaygroundPage.current.finishExecution()` na cláusula `defer { ... }`. Isso por que, como o código roda num contexto assíncrono, a execução do playground se encerraria antes de recebermos uma resposta e percebemos nossa implementação. A instrução faz com que o final da execução seja também postergado para o final da execução do código assíncrono.
>
>    ``` swift
>        func getSongs(for term: String,
>                      completionHandler: @escaping ([Song]) -> Void) {
>            // código omitido
>
>            dataTask = session.dataTask(with: url) { [weak self] data, response, error in
>                defer {
>                    PlaygroundPage.current.finishExecution()
>                    self?.dataTask = nil
>                }
>
>                // código omitid
>    ```
>
> O import de `PlaygroundSupport` é requerido.

``` swift
class ITunesSearchAPI {
    
    // código omitido
    
    func getSongs(for term: String,
                  completionHandler: @escaping ([Song]) -> Void) {
        dataTask?.cancel()
        
        let url = createURL(for: baseUrl, and: term)
        
        dataTask = session.dataTask(with: url) { [weak self] data, response, error in
            defer {
                PlaygroundPage.current.finishExecution()
                self?.dataTask = nil
            }
            
            if let error = error {
                print(error.localizedDescription)
                return
            }
            
            guard let data = data,
                  let response = response as? HTTPURLResponse,
                  response.statusCode == 200 else {
                print("Fail to receive data")
                return
            }
            
            do {
                let songsResponse = try JSONDecoder().decode(Song.Response.self, from: data)
                completionHandler(songsResponse.results)
                
            } catch let error {
                print(error.localizedDescription)
            }
        }
        
        dataTask?.resume()
    }
    
    // mais código omitido
}

// executa a função
let itunesAPI = ITunesSearchAPI()

itunesAPI.getSongs(for: "zeca pagodinho") { songs in
    print(songs)
}
```

<p align="center">
<img alt="Gif animado demonstrando a execução do exemplo acima, onde a request para a itunes search api é processada com sucesso, retornando a lista de músicas para o artista buscado pelo termo" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-teoria-urlsession-api-exemplo-request-processado.gif?raw=true" width="100%" />
</p>

Perceba que temos no console a representação _string_ do objeto, conforme descrita na _extension_ adicionada, para facilitar a identificação da mensagem.

Pronto! Já temos um modelo funcional de código async consumindo _endpoints_ REST. 🚀
