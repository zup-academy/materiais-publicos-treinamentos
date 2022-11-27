# JSONDecoder

Um objeto que decodifica instâncias de um tipo de dados de objetos JSON.

> Esse material teórico foi traduzido e adaptado da fonte original disponível na documentação em https://developer.apple.com/documentation/foundation/jsondecoder

#### Declaração

``` swift
class JSONDecoder
```

#### Visão Geral

O exemplo abaixo mostra como decodificar uma instância de um tipo simples de `Produto` de um objeto JSON. O tipo adota `Decodable` para que seja decodificável usando uma instância `JSONDecoder`.

``` swift
struct Produto: Decodable {
    var nome: String
    var pontos: Int
    var descricao: String?
}

let json = """
{
    "nome": "Banana",
    "pontos": 600,
    "descricao": "Uma fruta alongada e comestível produzida por vários tipos de grandes plantas herbáceas com flores do gênero Musa"
}
""".data(using: .utf8)!

let decoder = JSONDecoder()
let produto = try decoder.decode(Produto.self, from: json)

print(produto.nome) // Prints "Banana"
```

#### Para saber mais

A classe JSONDecoder, com sua API demonstrada acima, foi introduzida na versão 4 da linguagem Swift (ver detalhes da proposta de inclusão [aqui](https://github.com/apple/swift-evolution/blob/main/proposals/0166-swift-archival-serialization.md)) para substituir APIs mais antigas e alinhadas com a abordagem Objective-C como a [JSONSerialization](https://developer.apple.com/documentation/foundation/jsonserialization). Com a abordagem anterior o mapeamento de valores de um objeto seguindo a especificação JSON era feito para tipos muito mais genéricos, como NSDictionary. Isso levava à necessidade de executar uma conversão bastante trabalhosa para os tipos de dados alto nível descritos no modelo.

## Customizando como objetos JSON são mapeados para _structs_

Nem todos os casos de mapeamentos de dados JSON para objetos Swift ocorrem de maneira simples e natural. Para exemplificar isso, imagine uma API descrita há tempos, com padrões de valores e chaves que sofreram diversas manutenções ao longo do tempo. O que acontece se essa API é defina com chaves seguindo o padrão _snake\_case_ ao invés de _camelCase_? Nessa seção você verá como pode proceder para customizar o mapeamento dos dados JSON para seus objetos.

### Mapeando chaves em padrão snake_case

Imagine que a API que você esteja consumindo exponha dados seguindo esse esquema:

``` json
{
    "id": 10,
    "full_name": "Thomas Shelby",
    "active": false,
    "email_address": "tommy@shelby.co"
}
```

Segundo a especificação JSON estes dados são perfeitamente válidos. Ele representa um único objeto JSON com quatro chaves e valores. Com o que sabemos até o momento, se propuséssemos uma _struct_ Swift para mapear os dados, teríamos:

``` swift
struct User: Decodable {
  let id: Int
  let full_name: String
  let active: Bool
  let email_address: String
}
```

Esta _struct_ também é valida e poderíamos aplicar o exemplo demonstrado acima com JSONDecoder para obtê-la normalmente. No entanto, as propriedade armazenadas que definimos não estão de acordo com as convensões e guide line da linguagem. Se estivermos checando o estilo de código com algum linter, por exemplo, provavelmente haverá uma regra nos impedindo de entregar este código. Precisamos de valores seguindo camelCase, como abaixo:

``` swift
struct User: Decodable {
    let id: Int
    let fullName: String
    let active: Bool
    let emailAddress: String
}
```

Com o código atual, se tentarmos a conversão teríamos:

``` swift
let json = """
{
    "id": 10,
    "full_name": "Thomas Shelby",
    "active": false,
    "email_address": "tommy@shelby.co"
}
""".data(using: .utf8)!

do {
    let decoder = JSONDecoder()
    let user = try decoder.decode(User.self, from: json)
    
    print(user)
} catch {
    print(error)
}

// Imprime "keyNotFound(CodingKeys(stringValue: "fullName", intValue: nil), Swift.DecodingError.Context(codingPath: [], debugDescription: "No value associated with key CodingKeys(stringValue: \"fullName\", intValue: nil) (\"fullName\").", underlyingError: nil))"
```

O erro impresso no console nos indica que o JSONDecoder não conseguiu encontrar uma chave correspondente nos dados JSON para uma chave descrita com _fullName_.

Para que tenhamos a saída esperada precisamos então configurar a estratégia de decodificação de chaves do objeto `JSONDecoder`:

``` swift
do {
    let decoder = JSONDecoder()
    decoder.keyDecodingStrategy = .convertFromSnakeCase
    
    let user = try decoder.decode(User.self, from: json)
    print(user)
} catch {
    print(error)
}
```

Com `decoder.keyDecodingStrategy = .convertFromSnakeCase`, antes de realizar a decodificação, informamos ao objeto que as chaves seguem o padrão, e agora elas podem ser automaticamente mapeadas para camelCase.

#### Para saber mais: estratégia de codificação de chaves customizada

Dado que a especificação JSON não restringe de maneira específica a declaração de chaves para a representação de um objeto é possível que encontremos diversos padrões diferentes que podem oferecer ainda mais ruídos no código de mapeamento. Para casos menos comuns onde isso ocorra, você pode definir sua própria estratégia de decodificação de chaves. Pesquise sobre a `JSONDecoder.KeyDecodingStrategy` _.custom(\_:)_ em https://developer.apple.com/documentation/foundation/jsondecoder/keydecodingstrategy/custom

### Mapeando valores de data em diferentes formatos

Assim como é possível que a nossa aplicação interaja com dados chaveados de maneiras diversas, alguns valores para um dado também podem variar entre diferentes formatos e representações. Este é um caso aplicável para datas, por exemplo. Algumas podem ser definidas em formatos especificados, como por exemplo pela ISO 8601 ['2011-12-03', '2011-12-03T10:15:30Z'] outras podem ser definidas em termos de miliseconds transcorridos a partir de um momento base, ou mesmo seguir uma padronização customizada.

#### Um exemplo

Imagine que a API que você esteja consumindo exponha dados seguindo esse esquema:

``` json
{
    "id": 1234,
    "name": "my-awesome-repo",
    "isFork": false,
    "createdAt": "2022-01-01T01:01:01Z",
}
```

Podemos definir uma _struct_ Swift como abaixo para a representação, e prosseguir com o código para a deserialização com JSONDecoder:

``` swift
struct Repository: Decodable {
    let id: Int
    let name: String
    let isFork: Bool
    let createdAt: Date
}
```

No entanto, ao rodar teríamos:

```
typeMismatch(Swift.Double, Swift.DecodingError.Context(codingPath: [_JSONKey(stringValue: "Index 0", intValue: 0), CodingKeys(stringValue: "created_at", intValue: nil)], debugDescription: "Expected to decode Double but found a string/data instead.", underlyingError: nil))
```

O erro impresso no console nos indica que o JSONDecoder não conseguiu encontrar uma maneira de adequar o valor para a chave _createdAt_ da _struct_. Por padrão o decoder tenta construir uma data a partir da inicialização por um Double, mas o que temos como valor sabidamente é um _string_ com uma data no padrão ISO 8601.

Para termos sucesso na deserialização, precisamos configurar o JSONDecoder para usar uma forma de inicialização adequada com `dateDecodingStrategy`:

``` swift
do {
    let decoder = JSONDecoder()
    decoder.dateDecodingStrategy = .iso8601
    
    let repo = try decoder.decode(Repository.self, from: json)
    print(repo)
} catch {
    print(error)
}
```

#### Para saber mais: valores em formatos customizados

Para os casos onde seja necessário interagir com valores seguindo diferentes padronizações, é possível se utilizar de `dateFormatters` para os mesmos ao configurar o `dateDecodingStrategy` de um JSONDecoder. Pesquise sobre os `JSONDecoder.DateDecodingStrategy` _.formatted(\_:)_ e _.custom(\_:)_ em https://developer.apple.com/documentation/foundation/jsondecoder/datedecodingstrategy/formatted e https://developer.apple.com/documentation/foundation/jsondecoder/datedecodingstrategy/custom, respectivamente.

### Mapeando chaves customizadas com CodingKeys enum

Para os casos mais específicos onde se deseja que os objetos Swift mantenham chaves com padrões, ou mesmo identificadores, diferentes dos dados recebidos em JSON uma solução bastante útil é definir quais são as _coding keys_ associadas para cada propriedade armazenada da _struct_. _Coding keys_ customizadas são definidas nos seus objetos Decodable através de uma _enum_ interna ao tipo.

Considere o seguinte JSON:

``` json
{
    "id": 1234,
    "name": "my-awesome-repo",
    "full_name": "gh-user/my-awesome-repo",
    "fork": false,
    "created_at": "2022-01-01T01:01:01Z",
    "updated_at": "2022-01-01T01:01:01Z",
    "language": "Swift",
    "stargazers_count": 1
}
```

Este JSON pode ser bastante diferente do código que seja atingir para a `struct`:

``` swift
struct Repository: Decodable {
    let id: Int
    let name: String
    let fullName: String
    let isFork: Bool
    let createdAt: Date
    let updatedAt: Date
    let language: String
    let stars: Int 
}
```

Neste caso é possível utilizar a enum `CodingKeys` aninhada ao tipo para inferir para qual propriedade cada chave deve ser mapeada:

``` swift
struct Repository: Decodable {
    let id: Int
    let name: String
    let fullName: String
    let isFork: Bool
    let createdAt: Date
    let updatedAt: Date
    let language: String
    let stars: Int 

    enum CodingKeys: String, CodingKey {
        case id, name, language
        case fullName = "full_name"
        case isFork = "fork"
        case createdAt = "created_at"
        case updatedAt = "updated_at"
        case stars = "stargazers_count" 
    }
}
```

> Nota: A enum `CodingKeys` deve implementar `CodingKey` e deve ser definida em termos de uma `String`. Ela também precisa conter todas as propriedades da _struct_ como seus _cases_. Um caso em si precisa sempre corresponder a uma propriedade da _struct_ ou _class_ e o valor deve ser a chave JSON. Perceba que para as chaves onde o mapeamento é natural, é possível apenas definir um caso sem valor associado.

Dessa forma o código de deserialização pode permanecer simples e a Swift vai mapear automaticamente os valores para as instâncias.

``` swift
do {
    let decoder = JSONDecoder()
    decoder.dateDecodingStrategy = .iso8601
    
    let repo = try decoder.decode(Repository.self, from: json)
    print(repo)
} catch {
    print(error)
}
```
