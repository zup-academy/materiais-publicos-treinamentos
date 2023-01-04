# Mais de URLSession e Design de Código

Através do consumo do material teórico da última seção você foi capaz de entender um pouco mais sobre como funciona a API de URLSession, além de detalhes da plataforma iOS em si. No entanto, você foi capaz de observar isso pela perspectiva de consumo de endpoints que simplesmente consultam informações de um servidor. Por mais que isso nos tenha trazido conhecimento precisamos complementá-lo. Nesta seção e material teórico você vai ser introduzido a mais detalhes de utilização de URLSession, assim como a uma ideia de design de código para a abstração da camada de _networking_ trazendo-nos um bom nível de reutilização de código.

## URLSession e URLRequest

Ainda deve estar fresco em sua mente o código construído na seção anterior para a execução de alguns dos requests mais comuns. Caso não esteja, no _snippet_ abaixo trazemos ele de volta.

``` swift
class ITunesSearchAPI {
    
    let baseUrl: String = "https://itunes.apple.com/search?"
    
    var session: URLSession
    var dataTask: URLSessionDataTask?
    
    init(with session: URLSession = .shared) {
        self.session = session
    }
    
    func getSongs(for term: String,
                  completionHandler: @escaping ([Song]) -> Void,
                  failureHandler: @escaping (Error) -> Void) {
        dataTask?.cancel()
        
        guard let url = createURL(for: baseUrl, and: term) else {
            preconditionFailure("Não foi possível construir a URL")
        }
            
        dataTask = session.dataTask(with: url) { [weak self] data, response, error in
            defer {
                self?.dataTask = nil
                PlaygroundPage.current.finishExecution()
            }
            
            if let error = error {
                debugPrint(error.localizedDescription)
                
                failureHandler(.unableToRequest(error))
                return
            }
            
            guard let data = data,
                  let response = response as? HTTPURLResponse,
                  response.statusCode == 200 else {
                
                failureHandler(.requestFailed)
                return
            }
            
            do {
                let songsResponse = try JSONDecoder().decode(Song.Response.self, from: data)
                completionHandler(songsResponse.results)
                
            } catch let error {
                debugPrint(error.localizedDescription)
                failureHandler(.invalidData(error))
            }
        }
        
        dataTask?.resume()
    }

    private func createURL(for baseURL: String, and term: String) -> URL? {
        guard var urlComponents = URLComponents(string: baseURL) else { return nil }

        urlComponents.queryItems?.append(.init(name: "media", value: "music"))
        urlComponents.queryItems?.append(.init(name: "entity", value: "song"))
        urlComponents.queryItems?.append(.init(name: "term", value: term))
        
        return urlComponents.url
    }
}
```

> Nota: Este é apenas um recorte do código utilizado na teoria da seção anterior. Para acessar o código completo acesse o link abaixo:
>
>   [Gist com o código do Playground](https://gist.github.com/rafaelrollozup/ba9d5aad29b6469ad9b56f119e3cc4f6)

O código apresentado tem apenas algumas poucas adições em relação ao original da seção anterior, como um tratamento para o caso de alguma falha, por exemplo. Ele é perfeitamente funcional e você pode executá-lo, caso queira.

<p align="center">
<img alt="Imagem com resultado da execução do request proposto acima" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-post-teoria-mais-urlsession-imagem-resultado-request.png?raw=true" width="75%" />
</p>

No entanto, assim como a introdução deste material cita, o código é apenas mais um retrato de um consumo de API simples. Executamos um request através da configuração padrão de URLSession, sem qualquer parametrização adicionada. Sendo assim, executamos um simples _GET Request_. Buscamos as informações em JSON no servidor e a repassamos para o contexto da aplicação através da representação do modelo adequado.

Para fins didáticos, idealize então outro _endpoint_ para a API, que desta vez nos permita enviar um novo _song_ para o servidor. Este _endpoint_ seria acessado através da URI `https://itunes.apple.com/api/song`.

Poderíamos supor a construção de um código que habilitasse a integração.

``` swift
class ITunesAPI {
    
    let baseUrl: String = "https://itunes.apple.com/api%@"
    
    var session: URLSession
    var dataTask: URLSessionDataTask?
    
    init(with session: URLSession = .shared) {
        self.session = session
    }
    
    func createNew(_ song: Song,
                   completionHandler: @escaping (Song) -> Void,
                   failureHandler: @escaping (Error) -> Void) {
        dataTask?.cancel()
        
        let urlString = String(format: baseUrl, "/song")
        
        guard let uri = URL(string: urlString) else {
            preconditionFailure("Não foi possível construir a URI")
        }
            
        dataTask = session.dataTask(with: uri) { [weak self] data, response, error in
            defer {
                self?.dataTask = nil
                PlaygroundPage.current.finishExecution()
            }
            
            if let error = error {
                debugPrint(error.localizedDescription)
                
                failureHandler(.unableToRequest(error))
                return
            }
            
            guard let data = data,
                  let response = response as? HTTPURLResponse,
                  response.statusCode == 200 else {
                
                failureHandler(.requestFailed)
                return
            }
            
            do {
                let songsResponse = try JSONDecoder().decode(Song.Response.self, from: data)
                completionHandler(songsResponse.results)
                
            } catch let error {
                debugPrint(error.localizedDescription)
                failureHandler(.invalidData(error))
            }
        }
        
        dataTask?.resume()
    }
    
}
```

Além do fato de o endpoint não existir propriamente, teríamos outro problema caso testássemos a execução. Com REST como o padrão de design para APIs, _endpoints_ dessa natureza - através dos quais é desejável a criação de recursos no servidor - são expostos através do método de requisição `'POST'`.

Como visto acima, este _snippet_ de código seria então apenas aplicável a requisições simples, que se destinam a consultar informações. Como seria então possível configurar nossa _data task_ para contar com as capacidades extras necessárias ao exemplo?

### Parametrizando detalhes de um HTTP Request

A resposta para a pergunta ainda aberta é: utilizando a função `dataTask(with:)` do objeto de sessão que recebe uma instância de `URLRequest`, ao invés de apenas a `URL` do recurso.

A partir do objeto de _request_, é possível parametrizar nossa requisição HTTP para adicionar a ela as características necessárias.

Por exemplo, para que seja possível a criação de um recurso no servidor, além de configurar o método de requisição adequado, é necessário informar quais dados vão trafegar nos pacotes através da rede, qual a especificação utilizada para representar os dados, quais credenciais o cliente da chamada possui para que seja autorizada ou não a execução, entre outros detalhes.

Para que seja possível seguir adiante com essa abordagem precisamos primeiramente contar com uma instância de `URLRequest`. Vejamos como isso é possível no código abaixo:

``` swift
    // código omitido

    let urlString = String(format: baseUrl, "/song")
        
    guard let uri = URL(string: urlString) else {
        preconditionFailure("Não foi possível construir a URI")
    }
    
    var request = URLRequest(url: uri)
        
    dataTask = session.dataTask(with: request) {
        // código da closure
    } 
```

O código acima introduz a utilização do inicializador de `URLRequest` que pode ser utilizado de forma tão simples quanto ao passar uma referência de `URL` valida. Além desses detalhes o inicializador ainda oferece opções para configurar políticas de cache e tempo máximo de execução (_timeout_).

Perceba que no código a variável _request_ foi definida através de uma `var`, já que, em se tratando de uma _struct_, se a definíssemos através de `let` não seria possível a alteração do estado deste objeto.

Com tudo posto, o código abaixo exemplifica a ideia de parametrização para a execução da _POST Request_ conforme o exemplo:

``` swift
    // código omitido

    var request = URLRequest(url: uri)
    request.httpMethod = "POST"
    
    do {
        let data = try JSONEncoder().encode(song)
        
        request.httpBody = data
        request.setValue("application/json", forHTTPHeaderField: "Content-type")
        
    } catch let error {
        debugPrint(error)
        let contextError = Error.invalidData(error)
        
        failureHandler(.unableToRequest(contextError))
        return
    }
    
    request.setValue("application/json", forHTTPHeaderField: "Accept")
    request.setValue("Bearer eyJhbGc.eyJzdWIiO.SflKxwJV_ad...", forHTTPHeaderField: "Authorization")
    
    dataTask = session.dataTask(with: request) { 
        // código da closure
    }
```

O código acima ilustra todas as capacidades antes mencionadas. Através do objeto em `request`, conseguimos apontar o método, os dados, a representação sob a qual os dados são enviados e recebidos como resposta, além da informação de um _authentication token_ identificando o cliente da execução.

> Nota: Para que seja possível codificar a informação do objeto como um JSON - através do `JSONEncoder` - é necessário que o objeto serializável conforme com o protocolo `Encodable`. Até então, a _struct_ `Song` apenas conformava com o protocolo `Decodable`. É possível adicionar a referência ao novo protocolo necessário, ou apenas conformar com `Codable`, outro protocolo que funciona como um composição dos dois adjacentes. 

O código completo pode ser visto no link abaixo:

>   [Gist com o código do Playground](https://gist.github.com/rafaelrollozup/97f6d48a77eb174ec6c2bba9016ae182)

## Design de Código

A seção anterior introduziu ao menos dois grandes pedaços de código. Primeiro, o código que consome o _endpoint_ de _search_ da iTunes API, segundo, o código que consome um _pseuso-endpoint_ que habilitaria a criação de novos recursos de _song_ no servidor. 

Ao longo das classes e métodos que realizam estes trabalhos foi possível perceber uma grande similaridade. Elas dependem no geral dos mesmos módulos externos, conhecem em detalhes a API de `URLSession` para configurar e executar as operações necessárias, os grupos de instruções ordenadas para os fluxos de execução são os mesmos, entre outras características.

Conforme as apps crescem é muito comum que o número de pontos de contato com o servidor acompanhem o crescimento. No cenário atual de aplicações, é possível supor um aumento linear onde para cada feature, tenhamos no mínimo um consumo de dados remotos. Dessa forma, se cada _endpoint_ adicionado ao contexto de funcionamento de uma app levar ao acréscimo ou alteração de uma classe ou método de acordo com os exemplos acima, é possível supor os seguintes malefícios para o código que guarda os detalhes de consumo de APIs:

1. Repetição exagerada de código: pode indicar problemas de reuso de código e ser efeito do item a seguir;
1. Má separação de responsabilidades entre módulos: Com o mesmo módulo cuidando de detalhes de consumo de informações específicas, além de cuidar de detalhes de funcionamento da execução de requests HTTP com `URLSession`;
1. Maior carga cognitiva exercida pelo código: pode levar a dificuldades de entendimento e manutenção.

Ainda sendo possível citar maior dificuldade para a escrita de testes destes módulos. 

Precisamos buscar uma implementação mais saudável ao levar códigos desta natureza para o contexto das aplicações.

### Oferecendo um HTTPRequest como serviço

Após a identificação destes problemas é possível pensar em formas de obtermos um código mais saudável para as aplicações. O segundo item da lista ordenada anterior nos dá um bom ponto de partida: podemos atacar o acúmulo de responsabilidades nas classes propostas.

Podemos supor um isolamento de todo o código repetitivo que lida com os detalhes da API de `URLSession` ao oferecer essa infraestrutura como um serviço para os demais módulos da aplicação. Podemos pensar nisso como uma adição de mais um nível de abstração sobre o código da plataforma. Tornaremos o trabalho dos clientes interessados nessa capacidade significativamente mais simples e objetivo. Assim, as classes de `API` focam apenas nos detalhes do consumo pela visão da regras de negócio (por exemplo, ao indicarem qual recurso é oferecido às camadas superiores mais próximas da UI, se para este recurso é necessário uma autenticação ou mesmo quais parâmetros é necessário enviar para obter o que é desejado requisição), e a classe do serviço foca apenas em como gerenciar a utilização de `URLSession` para realizá-lo (contento detalhes mais internos de infraestrutura).

Vamos começar propondo o _request_ como um serviço:

``` swift
enum NetworkError: Swift.Error, LocalizedError {
    case unableToRequest(Swift.Error)
    case requestFailed
    case invalidData(Swift.Error)
    
    var errorDescription: String? {
        switch self {
        case .unableToRequest(let error):
            return "Unable to perform the request. \(error.localizedDescription)"
        case .requestFailed:
            return "Request failed"
        case .invalidData(let error):
            return "Unable to parse the request data. \(error.localizedDescription)"
        }
    }
}

class HTTPRequest {
    
    private var session: URLSession
    private var dataTask: URLSessionDataTask?
    
    init(session: URLSession = .shared) {
        self.session = session
    }
    
    func execute<T: Decodable>(for url: URL,
                 completionHandler: @escaping (T) -> Void,
                 failureHandler: @escaping (NetworkError) -> Void) {
        dataTask?.cancel()
            
        dataTask = session.dataTask(with: url) { [weak self] data, response, error in
            defer {
                self?.dataTask = nil
                playgroundPage.finishExecution() // necessário apenas para a execução via playground
            }
            
            if let error = error {
                debugPrint(error.localizedDescription)
                
                failureHandler(.unableToRequest(error))
                return
            }
            
            guard let data = data,
                  let response = response as? HTTPURLResponse,
                  response.statusCode == 200 else {
                
                failureHandler(.requestFailed)
                return
            }
            
            do {
                let decodable = try JSONDecoder().decode(T.self, from: data)
                completionHandler(decodable)
                
            } catch let error {
        
                debugPrint(error.localizedDescription)
                failureHandler(.invalidData(error))
            }
        }
        
        dataTask?.resume()
    }
}
```

Perceba como o código acima agrupa realmente todo o trabalho com a API de `URLSession` e mantém seus detalhes encapsulados. Dessa maneira um código cliente, como o que consome a _search_ API conseguiria ser escrito de forma tão simples e clara quanto abaixo:

``` swift
extension ITunesSearchAPI {
    enum Error: Swift.Error, LocalizedError {
        case executionFailed(NetworkError)

        var errorDescription: String? {
            switch self {
            case .executionFailed(let error):
                return error.localizedDescription
            }
        }
    }
}

class ITunesSearchAPI {

    let baseUrl: String = "https://itunes.apple.com/search?"

    private var httpRequest: HTTPRequest

    init(httpRequest: HTTPRequest = .init()) {
        self.httpRequest = httpRequest
    }

    func getSongs(for term: String,
                  completionHandler: @escaping ([Song]) -> Void,
                  failureHandler: @escaping (Error) -> Void) {

        guard let url = createURL(for: baseUrl, and: term) else {
            preconditionFailure("Não foi possível construir a URL")
        }

        httpRequest.execute(for: url) { (songs: Song.Response) in
            completionHandler(songs.results)

        } failureHandler: { error in
            failureHandler(.executionFailed(error))
        }
    }

    private func createURL(for baseURL: String, and term: String) -> URL? {
        guard var urlComponents = URLComponents(string: baseURL) else { return nil }

        urlComponents.queryItems?.append(.init(name: "media", value: "music"))
        urlComponents.queryItems?.append(.init(name: "entity", value: "song"))
        urlComponents.queryItems?.append(.init(name: "term", value: term))

        return urlComponents.url
    }
}
```

#### HTTPRequest: Atendendo à necessidades de diversos clientes

Da forma como proposta, a API do serviço já torna a vida dos clientes mais simples, no entanto, nem todos já estão contemplados. Idealize o código de API que consome o _pseudo-endpoint_ dos exemplos anteriores que se proprõe a criar um novo _song_ no servidor.

Vejamos como poderíamos consiumir nosso serviço através das necessidades deste outro cliente:

``` swift
class ITunesAPI {

    let baseUrl: String = "https://itunes.apple.com/api%@"

    private var httpRequest: HTTPRequest

    init(httpRequest: HTTPRequest = .init()) {
        self.httpRequest = httpRequest
    }

    func createNew(_ song: Song,
                   completionHandler: @escaping (Song) -> Void,
                   failureHandler: @escaping (Error) -> Void) {

        let urlString = String(format: baseUrl, "/song")

        guard let uri = URL(string: urlString) else {
            preconditionFailure("Não foi possível construir a URI")
        }

        var request = URLRequest(url: uri)
        request.httpMethod = "POST"

        do {
            let data = try JSONEncoder().encode(song)

            request.httpBody = data
            request.setValue("application/json", forHTTPHeaderField: "Content-type")

        } catch let error {
            debugPrint(error)
            let contextError = NetworkError.invalidData(error)

            failureHandler(.executionFailed(contextError))
            return
        }

        request.setValue("application/json", forHTTPHeaderField: "Accept")
        request.setValue("Bearer eyJhbGc.eyJzdWIiO.SflKxwJV_ad...", forHTTPHeaderField: "Authorization")

        httpRequest.execute(for: request <-) { (createdSong: Song) in // erro de compilação aqui: request <-
            completionHandler(createdSong)
            
        } failureHandler: { error in
            debugPrint(error)
            
            failureHandler(.executionFailed(error))
        }
    }

}
```

Perceba que este código insere um erro em tempo de compilação. A API atual do serviço espera receber uma `URL` válida apenas, e não um `URLRequest`.

Já sabemos da importância de utilizarmos o `URLRequest` para termos a possibilidade de parametrizarmos nossa execução, então poderíamos já supor uma alteração da API do serviço para receber um objeto de request, devidamente configurado. Mas não avance tão rápido: o que aconteceria com o código do cliente anterior?

Receber um _request_ poderia fazer com que os clientes com o consumo mais simples e recorrentes tivessem seu código afetado de forma não muito interessante. Agora eles precisariam criar uma `URL` para com ela construir um `URLRequest` sem nenhuma configuração, apenas para chamaram o método `execute(for:completionHandler:failureHandler:)`. Poderíamos procurar uma forma de deixar a API do serviço sempre o mais simples possível para os clientes, idealmente trocando mensagens a partir de objetos de uso geral. 

Por exemplo, podemos analisar a parametrização necessária para os _requests_ e transportar essa responsabilidade também para o serviço. Dessa maneira, caso o cliente tenha uma necessidade mais específica, ele pode através da execução explicitá-la e assim o serviço a executa.

Abaixo podemos ver essa abordagem. Levamos todo o código de parametrização necessário para _Post_ (ou outros) _requests_ para dentro do serviço: 

``` swift
enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
    case put = "PUT"
    case delete = "DELETE"
}

// enum NetworkError: Swift.Error, LocalizedError { ... }

class HTTPRequest {
    
    let baseUrl: String = "https://itunes.apple.com/api%@"
    
    private var session: URLSession
    private var dataTask: URLSessionDataTask?
    
    init(session: URLSession = .shared) {
        self.session = session
    }
    
    func execute<T: Decodable>(for resource: String,
                               method httpMethod: HTTPMethod = .get,
                               body encodable: Encodable? = nil,
                               decoder: JSONDecoder = .init(),
                               encoder: JSONEncoder = .init(),
                               completionHandler: @escaping (T) -> Void,
                               failureHandler: @escaping (NetworkError) -> Void) {
        dataTask?.cancel()
        
        let urlString = String(format: baseUrl, resource)

        guard let uri = URL(string: urlString) else {
            preconditionFailure("Não foi possível construir a URI")
        }
        
        var request = URLRequest(url: uri)
        request.httpMethod = httpMethod.rawValue

        if let encodable = encodable {
            do {
                let data = try encoder.encode(encodable)

                request.httpBody = data
                request.setValue("application/json", forHTTPHeaderField: "Content-type")

            } catch let error {
                debugPrint(error)
                let contextError = NetworkError.invalidData(error)

                failureHandler(.unableToRequest(contextError))
                return
            }
        }
    
        request.setValue("application/json", forHTTPHeaderField: "Accept")
            
        dataTask = session.dataTask(with: request) { [weak self] data, response, error in
            defer {
                self?.dataTask = nil
                playgroundPage.finishExecution()
            }
            
            if let error = error {
                debugPrint(error.localizedDescription)
                
                failureHandler(.unableToRequest(error))
                return
            }
            
            guard let data = data,
                  let response = response as? HTTPURLResponse,
                  response.statusCode == 200 else {
                
                failureHandler(.requestFailed)
                return
            }
            
            do {
                let decodable = try decoder.decode(T.self, from: data)
                completionHandler(decodable)
                
            } catch let error {
        
                debugPrint(error.localizedDescription)
                failureHandler(.invalidData(error))
            }
        }
        
        dataTask?.resume()
    }
}
```

Assim, deixamos que o cliente nos informe apenas a _string_ com o recurso a ser consumido, assim como outras informações opcionais sobre a requisição:

``` swift
class ITunesAPI {

    private var httpRequest: HTTPRequest

    init(httpRequest: HTTPRequest = .init()) {
        self.httpRequest = httpRequest
    }

    func createNew(_ song: Song,
                   completionHandler: @escaping (Song) -> Void,
                   failureHandler: @escaping (Error) -> Void) {

        httpRequest.execute(for: "/song",
                            method: .post,
                            body: song) { (createdSong: Song) in
            completionHandler(createdSong)
            
        } failureHandler: { error in
            debugPrint(error)
            
            failureHandler(.executionFailed(error))
        }
    }

}

extension ITunesAPI {
    enum Error: Swift.Error, LocalizedError {
        case executionFailed(NetworkError)

        var errorDescription: String? {
            switch self {
            case .executionFailed(let error):
                return error.localizedDescription
            }
        }
    }
}
```

Duas coisas ainda podemos colocar em risco a capacidade de uso do serviço `HTTPRequest`: (a) algumas request podem ser autenticadas, outras não; e (b) os _endpoints_ podem ter padrões de _status codes_ diferentes, mesmo que para indicar sucesso.

A adição abaixo poderia resolver essas questões:

``` swift
// enum HTTPMethod: String { .. }

typealias HTTPHeaders = [String : String]

extension HTTPURLResponse {
    
    var inSuccessRange: Bool {
        return (200..<300).contains(statusCode)
    }

}

// enum NetworkError: Swift.Error, LocalizedError { .. }

class HTTPRequest {
    
    let baseUrl: String = "https://itunes.apple.com/api%@"
    
    private var session: URLSession
    private var dataTask: URLSessionDataTask?
    
    init(session: URLSession = .shared) {
        self.session = session
    }
    
    func execute<T: Decodable>(for resource: String,
                               method httpMethod: HTTPMethod = .get,
                               body encodable: Encodable? = nil,
                               headers httpHeaders: HTTPHeaders? = nil,
                               decoder: JSONDecoder = .init(),
                               encoder: JSONEncoder = .init(),
                               completionHandler: @escaping (T) -> Void,
                               failureHandler: @escaping (NetworkError) -> Void) {
        dataTask?.cancel()
        
        // código omitido
    
        request.setValue("application/json", forHTTPHeaderField: "Accept")
        
        if let httpHeaders = httpHeaders { // aplica possíveis headers
            httpHeaders.forEach { (header, value) in request.setValue(value, forHTTPHeaderField: header) }
        }
            
        dataTask = session.dataTask(with: request) { [weak self] data, response, error in
            defer {
                self?.dataTask = nil
                playgroundPage.finishExecution()
            }
            
            if let error = error {
                debugPrint(error.localizedDescription)
                
                failureHandler(.unableToRequest(error))
                return
            }
            
            guard let data = data,
                  let response = response as? HTTPURLResponse,
                  response.inSuccessRange else { // valida se resposta se encontra em algum status que possa indicar sucesso da operação
                
                failureHandler(.requestFailed)
                return
            }
            
            do {
                let decodable = try decoder.decode(T.self, from: data)
                completionHandler(decodable)
                
            } catch let error {
        
                debugPrint(error.localizedDescription)
                failureHandler(.invalidData(error))
            }
        }
        
        dataTask?.resume()
    }
}
```

Com este código, os clientes podem indicar quais headers customizados são necessários para o contexto de cada _endpoint_, além do que, não trabalhamos mais com um status fixo de sucesso.

``` swift
class ITunesAPI {

    private var httpRequest: HTTPRequest

    init(httpRequest: HTTPRequest = .init()) {
        self.httpRequest = httpRequest
    }

    func createNew(_ song: Song,
                   completionHandler: @escaping (Song) -> Void,
                   failureHandler: @escaping (Error) -> Void) {
        
        // obtem junto a alguma infraestrutura de persistência
        let authToken = "Bearer eyJhbGc.eyJzdWIiO.SflKxwJV_ad..."
        
        let headers = ["Authorization": authToken]

        httpRequest.execute(for: "/song",
                            method: .post,
                            body: song,
                            headers: headers) { (createdSong: Song) in
            completionHandler(createdSong)
            
        } failureHandler: { error in
            debugPrint(error)
            
            failureHandler(.executionFailed(error))
        }
    }

}
```

Essa separação de responsabilidades nos permite manter o código de consumo de API bastante direto ao ponto, e ganhar bastante reúso sobre os detalhes de utilização da API de `URLSession`.

<p align="center">
<img alt="Imagem com o diagrama dos componentes de software no design de código proposto" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-post-teoria-mais-urlsession-imagem-design.jpeg?raw=true" width="75%" />
</p>

A imagem acima ilustra o design dos módulos envolvidos neste tipo de funcionalidade.

#### Para ir além: Definindo uma API de alto nível para lidar com _endpoints_ parametrizáveis

Ao executar os exemplos em um playground, o leitor atento deve ter notado que nossas últimas alterações não permitem mais que a `ITunesSearchAPI` execute com sucesso. Neste momento você deve estar se deparando com um erro em tempo de compilação: o código que guarda os detalhes da API de _search_ tenta passar adiante uma `URL` na chamada do método de serviço `execute(for:completionHandler:failureHandler:)` ao invés de _string_ esperada.

O problema em si foi gerado a partir da nossa tentativa de simplificar a API do serviço. Nossa busca teve uma justa causa, mas de nono, ainda não temos uma maneira de atender a todos os clientes.

O caso particular da _search_ API é que não apenas o nome do recurso é necessário para a execução do _request_, mas também o conjunto de parâmetros de requisição ( ou _query parameters_) com as informações do que efetivamente estamos procurando. Por este motivo o código da API continha uma função utilitária que nos ajudava a criar a `URL` com o formato e codificação adequados.

``` swift
    private func createURL(for baseURL: String, and term: String) -> URL? {
        guard var urlComponents = URLComponents(string: baseURL) else { return nil }

        urlComponents.queryItems?.append(.init(name: "media", value: "music"))
        urlComponents.queryItems?.append(.init(name: "entity", value: "song"))
        urlComponents.queryItems?.append(.init(name: "term", value: term))
        
        return urlComponents.url
    }
```

Perceba que foi utilizado a API de `URLComponents` para habilitarmos as funcionalidades mencionadas. O problema aqui é que para tanto, precisamos conhecer os detalhes da classe, além de guardar detalhes da URL base da API, possivelmente espalhando código dessa natureza por diversos outros códigos consumidores com necessidades parecidas. 

Deveríamos buscar uma forma mais alto nível de apontar para o serviço qual o recurso desejado e quais informações adjacentes devem ser consideradas para a construção da `URL`. A _struct_ abaixo pode implementar ideia semelhante:

``` swift
struct Endpoint {
    typealias QueryParams = [String : String]
    
    let path: String
    private var queryParams: [URLQueryItem]? = nil
    
    init(path: String, queryParams: QueryParams? = nil) {
        self.path = path
        
        if let queryParams = queryParams {
            self.queryParams = queryParams.map { URLQueryItem(name: $0, value: $1) }
        }
    }
    
    var uri: URL? {
        var components = URLComponents()
        components.scheme = "https"
        components.host = "itunes.apple.com"
        components.path = self.path
        components.queryItems = self.queryParams

        return components.url
    }
}
```

Perceba que definimos um objeto _Endpoint_ que encapsula detalhes de implementação como a utilização de `URLComponents`, `URLQueryItem` e `URL`, além de informações de base para a construção de qualquer URI para a API que consumimos. Um `Endpoint` pode ser definido em termos de um `path` (nome do recurso, por exemplo, `"/search"` ou `"/song"`) e de seus possíveis parâmetros de requisição (tratados como um simples dicionário), além de expor a URI esperada como uma simples `URL`.

Para que a API do serviço seja capaz de lidar com diferentes _enpoints_ de diversos clientes, podemos ainda definir um protocolo a partir do qual podemos orientar a disponibilização de objetos que representem recursos.

``` swift
protocol APIResource {
    var endpoint: Endpoint { get }
}

// e utilizá-lo na API do método execute(for:method:body:headers:decoder:encoder:completionHandler:failureHandler)

func execute<T: Decodable>(for resource: APIResource,
                            method httpMethod: HTTPMethod = .get,
                            body encodable: Encodable? = nil,
                            headers httpHeaders: HTTPHeaders? = nil,
                            decoder: JSONDecoder = .init(),
                            encoder: JSONEncoder = .init(),
                            completionHandler: @escaping (T) -> Void,
                            failureHandler: @escaping (NetworkError) -> Void) {
    dataTask?.cancel()

    guard let uri = resource.endpoint.uri else {
        preconditionFailure("Não foi possível construir a URI")
    }

    // resto do código omitido
```

Com essa alteração se torna fácil para o código de `ITunesSearchAPI`, por exemplo, inferir para qual recurso é necessário executar a requisição. Podemos dispor de uma enum com seus _associated values_ para tornar o código bastante expressivo e ainda ganhar robustez na implementação contando com as checagens em tempo de compilação.

``` swift
class ITunesSearchAPI {

    private var httpRequest: HTTPRequest

    init(httpRequest: HTTPRequest = .init()) {
        self.httpRequest = httpRequest
    }

    func getSongs(for term: String,
                  completionHandler: @escaping ([Song]) -> Void,
                  failureHandler: @escaping (Error) -> Void) {
        
        let resource = Resource.songsList(by: term)

        httpRequest.execute(for: resource) { (songs: Song.Response) in
            completionHandler(songs.results)

        } failureHandler: { error in
            failureHandler(.executionFailed(error))
        }
    }
    
}

extension ITunesSearchAPI {
    enum Resource: APIResource {
        case songsList(by: String)
        
        var endpoint: Endpoint {
            switch self {
            case .songsList(let searchTerm):
                let params = ["media": "music", "entity": "song", "term": searchTerm]
                return Endpoint(path: "/searchy", queryParams: params)
            }
        }
    }
}
```

Agora nosso código se encontra ainda mais flexível. Caso se façam necessários novos recursos, que requeiram ou não configurações diferentes de parâmetros de requisição, basta adicionar o caso particular na enum e completar a implementação.

### Utilizando a abstração Result para tornar a API mais concisa

Caso durante os testes de código você tenha incorrido em algum error do servidor, ou mesmo do cliente, deve ter percebido como o código atual nos revela pouco sobre as possíveis causas para os problemas - você pode testar isso simplesmente alterando algum caractere da URI consultada, por exemplo.

De acordo como nosso mecanismo de recuperação de falhas, caso nosso código em `ITunesSearchAPI` receba um `NetworkError` nessas condições, apenas teremos revelado através do console uma mensagem semelhante a `executionFailed(NetworkError.requestFailed)`. Podemos melhorar nossa implementação para garantir que o código cliente tenham acesso a informações mais precisas sobre o que ocorreu na execução da request pelo servidor.

Podemos adicionar valor ao tratamento de erro na camada de rede que estamos criando adicionando um valor associado ao caso `requestFailed` informando o _status code_ retornado pelo servidor. Mas não antes sem questionar mais sobre o design do código em `HTTPRequest`.

#### Dois idiomas para tratamento de erros

O já conhecido mecanismo de tratamento de erro padrão da Swift, funciona bem na maior parte dos cenários. No entanto, ele talvez não se comporte bem em implementações de natureza assíncrona. Pela característica de processamento de código assíncrono, onde no geral agendamos a execução de códigos para um ponto futuro, é incomum fazer uso de `throw`, por exemplo. De outra forma, em qual ponto do código teríamos controle e a recuperação da falha reportada?

Por este motivo programadores no geral vem escrevendo código com uma abordagem diferente, apostando em gerenciar os possíveis erros a partir da execução de _callbacks_ seja por meio de _closures_ ou _delegates_ em casos mais complexos. É comum por exemplo, ao necessitar do carregamento de dados remotos em JSON, a API de alguns módulos retornar representações de valores e erros, ambos passíveis de nulidade, sendo necessário então a checagem particular de cada um deles.

A explicação acima pode facilmente fazer você se lembrar da API de `URLSession`. Essa é exatamente uma das abordagens utilizada por ela e é possível relembrar do código que a utiliza abaixo.

``` swift
    session.dataTask(with: request) { [weak self] data, response, error in
        // check do erro
        // check da response
        // check de data
        // parse
    }
```

O design do código utilizá-lo pode não ser particularmente ruim, mas pode ter seus problemas, principalmente ao propagar este design para módulos adjacentes. Você precisa deliberadamente checar cada um dos possíveis valores retornados por sua própria conta. O que você executa caso ambos os valores sejam nulos? 

Bem, a possibilidade acima, por mais que seja afastada pela própria documentação de `URLSession`, não é aplicável em termos de código. Em teoria você poderia receber (e/ou propagar) ambos dados de resposta e erro, ou nenhum dos dois. Para assegurar robustez, sua implementação precisa prever o que pode acontecer dada qualquer possível combinação de estados entre valores, sem qualquer auxílio em tempo de compilação para isso. Como resultado, podemos obter um código bastante defensivo e portanto mais denso, podendo exercer maior carga cognitiva e dificuldades de entendimento.

``` swift
    dataTask = session.dataTask(with: request) { [weak self] data, response, error in
        // código omitido
        
        // check do erro
        if let error = error {
            debugPrint(error.localizedDescription)
            
            failureHandler(.unableToRequest(error))
            return
        }
        
        // check da resposta
        if let response = response as? HTTPURLResponse, !response.inSuccessRange {
            failureHandler(.requestFailed(statusCode: response.statusCode))
            
        } else if let data = data {
            // check da data

            // parse
            do {
                let decodable = try decoder.decode(T.self, from: data)
                completionHandler(decodable)
                
            } catch let error {
                debugPrint(error.localizedDescription)
                
                failureHandler(.invalidData(error))
            }
            
        }
        
    }
```

Perceba a extração da checagem da resposta HTTP para compreendermos em detalhes o que houve na execução do _request_ pelo servidor. Além do que já tinhamos, foi necessário adicionar mais uma checagem (com seu devido _binding_ de opcional) e mais um nível de aninhamento. Essa carga extra pode levar a um aumento de complexidade dos nossos códigos.

#### Utilizando uma abstração de mais alto nível para expressar a ideia de Resultado

Para endereçar a questão anterior a comunidade Swift passou a utilizar uma abstração inspirada em tipos existentes em outras linguagens: a `Result`. Com a escalada da adoção da abordagem, mais e mais desenvolvedores pediam sua inclusão na biblioteca padrão da linguagem, o que aconteceu, depois de bastante discussão, na versão 5 da Swift.

##### Conhecendo o Result

Antes de propor uma solução com este novo recurso é importe conhecê-lo um pouco mais.

``` swift
public enum Result<Success, Failure: Error> {

    /// Indicates success with value in the associated object.
    case success(Success)

    /// Indicates failure with error inside the associated object.
    case failure(Failure)

    // código restante omitido por questões de brevidade
}
```

Como é possível perceber a partir da implementação base (assim como sua documentação), `Result` é definido em termos de sucesso (`case success`) ou falha (`case failure`) através de uma _enum_. O tipo `Result` é também bastante parecido com outro já bem conhecido, o `Optional`. Ambas as representações são feitas através de simples _enums_ (com `Optional` tendo casos de `none` ou `some`), mas contém uma série de funcionalidades interessantes das quais podemos tirar proveito.

A diferença mais notável entre eles é que, ao invés de termos um valor (`some`) ou `nil` (`none`) no caso dos opcionais, com um `Result` temos valores para ambos os casos possíveis. Para o caso mais crítico no geral, `Result` indica uma falha e não `nil`, o que significa dizer que utilizando um `Result` você pode ter contexto do por quê de uma operação ter falhado.

Você deve ter notado a utilização de _generics_ para inferir ao tipo de dado utilizado como base para a representação do resultado. Em especial, o _type parameter_ para a falha possui uma _constraint_ que o define como qualquer tipo que conforme com o protocolo `Error`. Essa _constraint_ habilita um recurso interessante que veremos adiante.

##### `Result` na prática

Para conhecer melhor como a abstração `Result` pode nos ajudar em termos de código, nada melhor do que reunir tudo o que foi apresentado até aqui e colocar em prática em um cenário. Nesta subseção vamos alterar a API do nosso serviço `HTTPRequest` passo a passo em busca de um código mais robusto e direto ao ponto.

Começaremos por tentar evitar a propagação do design herdado da `URLSession` _callback_ para as classes consumidoras. Como visto, é comum que nestes tipos de código propaguemos a ideia de utilização de _callbacks_ com valores possivelmente nulos associados ao sucesso e falha da operação, ou mesmo múltiplos _callbacks_ para as situações específicas. Utilizamos este último nos códigos até aqui, e, por mais que eles suavizem o esforço de trabalho evitando os opcionais, ainda exerce uma carga não muito agradável ao código dos clientes com ainda mais um _callback_ sendo necessário.

Começaremos por utilizar um `Result<T, NetworkError>` no contexto de _completion handler_ como único _callback_.

``` swift
func execute<T: Decodable>(for resource: APIResource,
                            method httpMethod: HTTPMethod = .get,
                            body encodable: Encodable? = nil,
                            headers httpHeaders: HTTPHeaders? = nil,
                            decoder: JSONDecoder = .init(),
                            encoder: JSONEncoder = .init(),
                            completionHandler: @escaping (Result<T, NetworkError>) -> Void) {

    // código omitido
}
```

A alteração acima já nos força a realizarmos algumas alterações na implementação. As invocações de `failureHandler` não mais são válidas e é necessário construir um objeto de `Result` com o estado adequado.

``` swift
    func execute<T: Decodable>(for resource: APIResource,
                               method httpMethod: HTTPMethod = .get,
                               body encodable: Encodable? = nil,
                               headers httpHeaders: HTTPHeaders? = nil,
                               decoder: JSONDecoder = .init(),
                               encoder: JSONEncoder = .init(),
                               completionHandler: @escaping (Result<T, NetworkError>) -> Void) {
        dataTask?.cancel()

        guard let uri = resource.endpoint.uri else {
            preconditionFailure("Não foi possível construir a URI")
        }
        
        var request = URLRequest(url: uri)
        request.httpMethod = httpMethod.rawValue

        if let encodable = encodable {
            do {
                let data = try encoder.encode(encodable)

                request.httpBody = data
                request.setValue("application/json", forHTTPHeaderField: "Content-type")

            } catch let error {
                debugPrint(error)
                let contextError = NetworkError.invalidData(error)

                completionHandler(Result.failure(.unableToRequest(contextError))) // ALTERADO
                return
            }
        }
    
        request.setValue("application/json", forHTTPHeaderField: "Accept")
        
        if let httpHeaders = httpHeaders {
            httpHeaders.forEach { (header, value) in request.setValue(value, forHTTPHeaderField: header) }
        }
            
        dataTask = session.dataTask(with: request) { [weak self] data, response, error in
            defer {
                self?.dataTask = nil
                playgroundPage.finishExecution()
            }
            
            if let error = error {
                debugPrint(error.localizedDescription)
                
                completionHandler(Result.failure(.unableToRequest(error))) // ALTERADO
                return
            }
            
            if let response = response as? HTTPURLResponse, !response.inSuccessRange {
                completionHandler(Result.failure(.requestFailed(statusCode: response.statusCode))) // ALTERADO
                
            } else if let data = data {
                
                do {
                    let decodable = try decoder.decode(T.self, from: data)
                    completionHandler(Result.success(decodable)) // ALTERADO
                    
                } catch let error {
            
                    debugPrint(error.localizedDescription)
                    completionHandler(Result.failure(.invalidData(error))) // ALTERADO
                }
                
            }
            
        }
        
        dataTask?.resume()
    }
```

Utilizamos os casos `.success(T)` e `.failure(NetworkError)` para construir a instância da _enum_ com o estado apropriado, e invocamos o _completionHandler_ com sua passagem para o contexto. Aqui ainda não temos um código funcional. A alteração quebrou o contrato da API conhecido pelos consumidores. O código abaixo efetua os ajustes na implementação de `ITunesSearchAPI` e consequentemente em sua API e clientes para usar a nova abordagem.

``` swift
class ITunesSearchAPI {

    private var httpRequest: HTTPRequest

    init(httpRequest: HTTPRequest = .init()) {
        self.httpRequest = httpRequest
    }

    func getSongs(for term: String,
                  completionHandler: @escaping (Result<[Song], Error>) -> Void) {
        
        let resource = Resource.songsList(by: term)

        httpRequest.execute(for: resource) { (result: Result<Song.Response, NetworkError>) in
            switch result {
            case .success(let songs):
                completionHandler(Result.success(songs.results))
                
            case .failure(let error):
                debugPrint(error)
                
                completionHandler(Result.failure(.executionFailed(error)))
            }
        }
        
    }
    
}

let api = ITunesSearchAPI()

api.getSongs(for: "martinho da vila") { result in
    
    switch result {
    case .success(let songs):
        print(songs)
        
    case .failure(let error):
        let message = "Não foi possível carregar a lista de músicas. \(error.localizedDescription)"
        
        // exibição do erro via alerta ou outra API customizada
        print(message)
    }
    
}
```

Perceba que com as alterações acima já temos alguma redução da carga dos códigos de múltiplos _callbacks_, além de contar com o suporte da checagem dos estados em tempo de compilação ao usar o _match_ do `switch` com as _enums_.

O código acima é funcional e já mostra um pouco da nova abordagem, mas de certo ainda não a utilizamos em todo o seu potencial. O código do serviço `HTTPRequest` não teve nenhuma melhoria significativa. 

Você deve se lembrar do último trecho de código relacionado a ele. Alteramos sua API para oferecer um `Result`. No entanto, fizemos apenas isso. Ao permitir a codificação sobre o design induzido pelo _callback_ da função `dataTask(with:completionHandler:)` perdemos ainda muito recurso.

##### `Result` e suas possíveis transformações

Para que possamos obter um código substancialmente mais simples e entender mais vantagens do uso da nova abordagem, devemos o quanto antes trocar o contexto entre o design proposto pela API de `URLSession` e o uso da abstração `Result`. Isso significa o quanto antes na implementação do _callback_ do _completion handler_, traduzir o trabalho com `data` e `error` para a língua do `Result`. Para facilitar o processo, o código abaixo introduz uma _extension_ que habilita a criação de um `Result` no estado mais adequado a partir dos possíveis valores e erros.

``` swift
extension Result {
    
    init(value: Success?, error: Failure?) {
        if let error = error {
            self = .failure(error)
            return
        }
        
        if let value = value {
            self = .success(value)
            return
        }

        fatalError("Could not possible to create a valid result")
    }
    
}
```

Com a extensão acima, é possível alterar a implementação do _handler_ para construir o quanto antes o result com `Result(value: _?, error: _?)`.

``` swift
    dataTask = session.dataTask(with: request) { [weak self] data, response, error in
        defer {
            self?.dataTask = nil
            playgroundPage.finishExecution()
        }

        let result = Result(value: data, error: error)

        // código omitido
    }
```

Com o código acima temos a criação de um `Result<Data, Error>`, mas a julgar pelo que prometemos entregar aos nossos clientes (`Result<T, NetworkError>`), ainda estamos longe. Entretanto, a partir do ponto onde estamos podemos aproveitar mais os recursos da abstração de `Result`.

O `Result` da Swift foi inspirado em tipos existentes em linguagens também reconhecidas por um bom suporte ao paradigma de programação funcional, e assim sendo, traz consigo uma série de recursos naturais desse estilo. Em situações como a atual, onde desejamos transpor uma ideia de resultado mais genérico para algo em termos mais específicos, podemos trabalhar sobre os dados que um `Result` mantém através de operações que o manipulam como uma espécie de fluxo de dados. Podemos prever operações de mapeamento de valores de origem para resultantes esperados.

Uma das operações mais conhecidas para este fim é o `map`. Um `map` utiliza uma função que considera um valor em seu formato de entrada e através dele, provê um valor de saída em um possível novo formato. Em nosso contexto, podemos utilizá-lo, por exemplo, para prever a transformação de um `Error`. Um `Error`, definido através da constante `error` do _completionHandler_, simboliza um problema que nos impediu de executar requisição. Dessa forma, sabemos que para um `error` não nulo, nosso serviço deve produzir um `NetworkError.unableToRequest(error)`.

``` swift
    dataTask = session.dataTask(with: request) { [weak self] data, response, error in
        defer {
            self?.dataTask = nil
            playgroundPage.finishExecution()
        }

        let result = Result(value: data, error: error)
            .mapError { error in
                return NetworkError.unableToRequest(error)
            }

        // código omitido
    }
```

As funções `map` e `mapError`, respectivamente, nos permitem prever operações de transformação aos valores para `Success` e `Failure` do Result. Suas assinaturas podem indicar isso: `map(_ transform: (Data) -> NewSuccess) -> Result<NewSuccess, Error>` e `mapError(_ transform: (Error) -> Error) -> Result<Data, Error>`.

No contexto do nosso `Result` a função `map` aplicaria a transformação sobre o valor do caso de sucesso (`Data`) o transformaria em um novo valor de sucesso, inferido genericamente por `NewSuccess`. O valor resultante seria embrulhado em outro `Result` mas agora com a inferência para o tipo mais específico: `Result<NewSuccess, Error>`. 

De mesma forma a função `mapError` aplica uma transformação sobre o error do caso de falha (`Error`) o transformando em um novo valor de falha, agora do tipo `NetworkError` (caso `unableToRequest`). O valor resultante é embrulhado em um `Result` agora com a inferência para o tipo de falha mais adequado: `Result<Data, NetworkError>`. Com isso damos um passo na direção do `Result` desejado.

<p align="center">
<img alt="Imagem com o diagrama com o modelo de transformação do map explicado" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-post-teoria-mais-urlsession-imagem-mapping.jpeg?raw=true" width="75%" />
</p>

A imagem acima ilustra a transformação de um _map_.

##### Mapeando com aninhamento de resultados

Indo adiante no nosso pipeline de transformações, é hora de executar as operações de parse para efetivamente obter o objeto desejado. Com base no que vimos acima, podemos supor a utilização de outro `map`, dessa vez sobre o valor de `Success`.

``` swift
    let result = Result(value: data, error: error)
        .mapError { error in
            return NetworkError.unableToRequest(error)
        }
        .map { data in
            // parse
        }
```

O código que realiza o parse, já com seus detalhes de recuperação de falha, é conhecido. O conjunto de instruções abaixo o ilustra.

``` swift
        do {
            let decodable = try decoder.decode(T.self, from: data)
            // já se pode retornar T
            
        } catch let error {
            debugPrint(error.localizedDescription)
            
            // o que seria retornado aqui?
        }
```

Podemos então supor a junção desses dois pedaços de código:

``` swift
    let result = Result(value: data, error: error)
        .mapError { error in
            return NetworkError.unableToRequest(error)
        }
        .map { data in
            do {
                let decodable = try decoder.decode(T.self, from: data)
                return decodable
                
            } catch let error {
                debugPrint(error.localizedDescription)
                return NetworkError.invalidData(error) // ???
            }
        }
```

O código acima nos apresenta dois problemas. Um fundamentalmente prático e outro mais conceitual.

1. Pela definição da função `map` para o `Result` é impossível retornar o erro de parse onde nesse caso seria esperado `T` - lembrando que apenas em `mapError` conseguimos produzir um erro. Talvez seria possível pensar em algo como lançar um erro, mas..

1. A abordagem com o pipeline de transformações com `Result` deve privilegiar o caminho de sucesso do código em detrimento de possíveis verificações ou tratamentos de falha.

Além dos problemas citados, cabe ainda analisar o contexto de execução de ambas as _branches_ desse código contindo na função de transformação do `map`. Em ambos os casos, temos uma situação final para a transformação. Tanto ao ter o objeto serializado, quanto ao ter um erro de parse, temos o que é preciso para inferir o `Result<T, NetworkError>` que devemos passar aos clientes.

Podemos então seguir por uma abordagem próxima à sugestão do item um, no entanto, levando em consideração o ponto do item dois. Para isso, vamos contar com um recurso interessante de `Result`. Podemos construir um `Result` a partir da execução de um bloco de código prevendo a possibilidade desse código lançar um erro.

``` swift
    let result = Result(value: data, error: error)
        .mapError { error in
            return NetworkError.unableToRequest(error)
        }
        .map { data in
            return Result(catching: { try decoder.decode(T.self, from: data) })
        }
```

O inicializador `init(catching: () throws -> Success)` nos permite ir direto a ponto, pensando apenas no caminho de sucesso do código do serviço. Essa é uma das principais vantagens da abordagem com `Result` e a utilidade da _constraint_ em `<_, Failure: Error>` agora também pode ficar um pouco mais clara.

Embora o código acima seja perfeitamente compilável, temos um problema a resolver. O tipo inferido para a variável `result` após as transformações é `Result<Result<T, Error>, NetworkError>`.

Em um primeiro momento isso pode parecer um comportamento um tanto estranho, mas há pouco, ao falarmos sobre `map` e `mapError`, vimos como os resultados da funções de transformações são aplicados. Essas transformações são aplicadas a nível dos valores associados aos casos de sucesso e falha. Ao final, eles são embrulhados em um `Result` que é então devolvido como resposta à invocação.

Dessa forma, ao final da função de transformação com o `Result(catching: { ... })` temos um `Result<T, Error>`. Ao embrulhar isso em um `Result`, temos então, `Result<Result<T, Error>, NetworkError>`

> Nota: Perceba que a função de mapeamento sempre retorna o valor adjacente de acordo com o tipo original, neste caso, ao mapear o valor de sucesso, o valor de falhar do produto se torna um `Error` ao invés de `NetworkError`. E faz todo sentido afinal, o `Resultado` da transformação pode agora ser um erro de parsing, por exemplo, ainda não mapeado para um `NetworkError`.

##### Conhecendo o flatMap

Ao trabalhar com fluxos de dados seguindo o estilo funcional de código é perfeitamente normal nos depararmos com situações como essas. De tal forma, existe um mecanismo também comum para que consigamos lidar com esse aninhamento indesejado de resultados.

Quando os diversos mapeamentos geram resultados tal como o nosso, com diversas camadas do tipo que encapsula os dados (o _wrapper_) podemos "achatar" (_flat_) esse resultado de mapeamento através do uso da função `flatMap`.

``` swift
    let result = Result(value: data, error: error)
        .mapError { error in
            return NetworkError.unableToRequest(error)
        }
        .flatMap { data in
            return Result(catching: { try decoder.decode(T.self, from: data) })
        }
```

A substituição do `map` para `flatMap` evita que tenhamos o resultado aninhado. A imagem abaixo ilustra o comportamento.

<p align="center">
<img alt="Imagem com o diagrama com o modelo de transformação do flatmap" src="https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/explorando-o-mundo-ios/imagens/urlsession-post-teoria-mais-urlsession-imagem-flatmapping.jpeg?raw=true" width="75%" />
</p>

Com o resultado da transformação a partir de um `flatMap`, agora temos um `Result<T, Error>`. Falta apenas mais um passo para que tenhamos o resultado exatamente nos termos em que precisamos.

De maneira geral, o código que temos agora já seria capaz de retornar um caso de sucesso perfeitamente válido. Precisamos apenas adicionar o mapeamento dos possíveis erros que podem ter ocorrido na tentativa de serialização do objeto.

Em tempo de execução, as transformações são acionadas apenas em contexto onde façam sentido. Em outras palavras, um `Result` com estado de erro não executará transformações para o caso de sucesso e vice-versa. Esse comportamento nos estimula a compor nosso pipeline de forma bastante independente.

Caso tenhamos um erro que nos impeça de executar a requisição, o mesmo será mapeado para `NetworkError.unableToRequest` e o pipeline encerra sua execução, no entanto com este representado a nível de `Error` apenas. Caso executemos o _request_ e tenhamos a resposta normalmente, podemos receber um _status code_ que indique que o processamento não foi possível, ou mesmo em uma resposta de sucesso, podemos ter falha ao serializar o objeto. De uma forma ou de outra temos um estado final para casos de falha.

O código abaixo adiciona mais uma transformação para que seja possível obter nosso `Result<T, NetworkError>`.

``` swift
    let result = Result(value: data, error: error)
        .mapError { error in
            return NetworkError.unableToRequest(error)
        }
        .flatMap { data in
            return Result(catching: { try decoder.decode(T.self, from: data) })
        }
        .flatMapError { error in
            if let error = error as? NetworkError {
                return Result.failure(error)
            }
            
            if let response = response as? HTTPURLResponse, !response.inSuccessRange {
                return Result.failure(.requestFailed(statusCode: response.statusCode))
            }
            
            return Result.failure(.invalidData(error))
        }
```

Com o código acima conseguimos chegar ao resultado com uma implementação bastante mais concisa que a abordagem anterior.

Ao longo das últimas seções conseguimos navegar pelas diferentes abordagens de trabalho com API assíncronas, além de estabelecer um bom design para a comunicação entre os módulos.

Caso queira, você pode acessar o código final para esta teoria pelo link abaixo:

>   [Gist com o código final do Playground](https://gist.github.com/rafaelrollozup/a36729c3fd98a891531d52bb613fe7ac)
