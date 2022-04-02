# Implementando API REST com chave de unicidade para dados sensíveis

Nesse conteúdo veremos como podemos implementar uma API REST responsável por cadastrar um registro único no sistema, onde sua chave de unicidade será uma informação sensível que não pode ser armazenada abertamente no banco de dados. Vamos partir da construção da API desde a escrita do controller via módulo Spring Web até a persistência com Spring Data, JPA e Hibernate.

## Cadastrando destinatários para entregas de pedidos

Imagine que tenhamos um sistema que faz entregas de refeições de restaurantes na cidade, e para efetuar uma entrega ela precisa das informações do destinátario, como nome, telefone, endereço, além de manter histórico de entregas etc. Desse modo, o desenvolvedor(a) do time criou uma classe `Destinatario` para representar um destinatário dentro do sistema, onde essa classe é mapeada via anotações da JPA, como abaixo:

```java
@Entity
class Destinatario {

    @Id
    @GeneratedValue
    private Long id;

    private String nome;
    private String telefone;
    
    @Column(nullable = false, unique = true)
    private String cpf

    private String endereco;

    // desmais atributos, construtores e métodos

}
```

Um ponto importante é que o destinatário deve ser único sistema, por esse motivo o desenvolvedor(a) definiu uma constraint de unicidade no atributo `cpf`. Junto a isso, ele implementou o controller de cadastro de destinatários como a seguir:

```java
@RestController
class NovoDestinatarioController {

    @Autowired
    private DestinatarioRepository repository;

    @Transactional
    @PostMapping("/api/destinatarios")
    public ResponseEntity<?> cadastra(@RequestBody @Valid NovoDestinatarioRequest request, UriComponentsBuilder  uriBuilder) {

        if (repository.existsByCpf(request.getCpf())) {
            throw new ResponseStatusException(HttpStatus.UNPROCESSABLE_ENTITY, "destinatário existente no sistema");
        }

        Destinatario destinatario = request.toModel();
        repository.save(destinatario);

        URI location = uriBuilder.path("/api/destinatarios/{id}")
                    .buildAndExpand(destinatario.getId())
                    .toUri();

        return ResponseEntity
                .created(location).build();
    }

    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<?> handleDatabaseConstraintErrors(ConstraintViolationException ex, WebRequest request) {
        // lógica de tratamento de erros de constraints UNIQUE do banco
    }
}
```

A implementação da API REST funciona bem e atende aos requisitos de negócio. Contudo a equipe de segurança percebeu que entre todas as informações do destinatário o CPF não é de fato necessário no sistema e, para piorar, não foi solicitado o **consentimento do usuário** para guardar este dado sensível no sistema. Por esse motivo, a equipe de segurança sugeriu ao desenvolvedor(a) remover o atributo `cpf` da entidade ou no mínimo **anonimizá-lo** dentro do banco de dados.

O desenvolvedor(a) decidiu anonimizar o CPF em vez de remover a coluna da tabela do banco de dados, pois ter o CPF no sistema é importante por causa da chave de unicidade. Para fazer algo do tipo, se faz necessário aplicar técnicas de **Data Masking** nos dados da aplicação.

## Data Masking: mascarando ou anonimizando dados sensíveis

De forma simplista, a idéia de Data Masking é mascarar ou anonimizar dados sensíveis (dados pessoais ou comerciais) em um sistema de tal forma que esses dados sejam **inutilizados** para usários não autorizados ou hackers, não servindo mais para identificar um individuo ou mesmo possibilitar seu uso em operações no mundo real.

Essa é uma prática comum em sistemas financeiros de grandes bancos e fintechs, serviços e operações que envolvem o uso ou armazenamento de dados de cartão de crédito, contas digitais etc. Esse tipo de preocupação com os dados alheios se tornou ainda mais importante após a chegada da LGPD (Lei Geral de Proteção de Dados Pessoais), sancionada em agosto de 2018.

Sabemos que precisamos anonimizar o CPF do destinatário, e para isso podemos escrever um algoritimo simples que substitua alguns números do CPF por um caractere coringa, como `'*'` ou `'x'`, ou seja, algo como:

```
691.915.220-72 -> 691.***.***-72
```

Esse simples "replacement" no CPF impossibilita a identificação do cliente ou o uso indevido do CPF para fraudes em caso do sistema vir a ser invadido ou ter os dados vazados na internet.

### Anonimizando o CPF do destinatário

Apesar de existirem bibliotecas Java que ajudam nesse tipo de tarefa, podemos implementar este algoritimo utilizando somente a linguagem Java. Por exemplo, podemos criar uma classe `CpfUtils` com um método estático chamado de `anonymize`, semelhante a este:

```java
public class CpfUtils {

    public static String anonymize(String cpf) {
        String regex = "([0-9]{3})\\.([0-9]{3})\\.([0-9]{3})\\-([0-9]{2})";
        String masked = cpf.replaceAll(regex, "$1.***.***-$4");
        return masked;
    }
}
```

O algoritimo acima é bem simples e se utiliza de uma expressão regular (Regex) para fazer a substituição dos números mais ao centro do CPF, não à toa ele assume que receberá um número de CPF formatado como entrada da aplicação.

Por fim, basta alterar o código da aplicação para anonimizar o CPF antes de gravá-lo no banco de dados. Portanto, o construtor da entidade `Destinatario` parece ser o melhor local para aplicar o data masking, logo teríamos o código a seguir:

```java
@Entity
class Destinatario {

    // atributos

    public Destinatario(String nome, String telefone, String cpf) {
        this.nome = nome;
        this.telefone = telefone;
        this.cpf = CpfUtils.anonymize(cpf); // anonimiza o cpf
    }

}
```

Ao exercitar o endpoint de cadastro, a alteração no código vai funcionar como esperado e gravará os novos destinatários no banco, mas desta vez com seus respectivos CPFs anonimizados:

```sql
SELECT d.*
  FROM destinatario d
;

id|nome        |telefone      |cpf           |
--+------------+--------------+--------------+
 1|Rafael Ponte|+5585988887777|691.***.***-72|
 2|Jordi       |+5511966665555|413.***.***-91|
```

Nossa solução de fato funcionou como esperado, mas o que acontece se tentarmos cadastrar 2 destinatários com os CPFs ligeiramente parecidos, como abaixo:

```
Yuri   , 540.602.880-47
Alberto, 540.301.164-47
```

Repare que ambos os CPFs possuem o prefixo e sufixo iguais, e isso que dizer que ao anonimiza-los eles acabarão com o mesmo valor:

```
Yuri   , 540.***.***-47
Alberto, 540.***.***-47
```

Para inofensivo, certo? Mas não é! Como já sabemos, por haver uma chave de unicidade na tabela o banco de dados não permitirá gravar o segundo destinatário, lançando um erro de violação de constraint semelhante a este:

```log
org.postgresql.util.PSQLException: ERROR: duplicate key value violates unique constraint "uk_zztn7wgfxo0xytrfby23f72up"
  Detail: Key (cpf)=(540.***.***-47) already exists.
```

 A verdade, é que no momento que anonimizamos o CPF por se tratar de um dado sensível, nós abrimos mão de sua caracteristica mais importante para nós nesse momento: sua unicidade. O que estou querendo dizer, é que a partir de agora o indice de conflitos entre CPFs de destinatários distintos aumentou substancialmente, o que inviabiliza seu uso no sistema. 

 Mas como anonimizar o CPF e ao mesmo tempo mantê-lo como chave de unicidade no nosso sistema?

## Encriptando dados sensíveis com Hash Functions

Uma Função Hash (ou Hash Function) é um algoritmmo de criptografia que recebe como entrada um texto qualquer ou um conjunto de bytes e produz como saída um número de tamanho fixado. Esse número de tamanho fixado (fixed-length value) gerado pelo algoritimo nós comumente chamamos de "hash".

Uma das principais características de uma função Hash é que ela é uma operação unidirecional (one-way function), ou seja, depois que um texto é encriptado num hash não há mais como revertê-lo para seu valor original. Não à toa, essa característica é muito útil por questões de segurança, por exemplo para encriptar senhas de usuários em sistemas web.

Outra característica importante deste algoritimo, é que um hash gerado é sempre único, ou seja, dois valores quaisquer nunca gerarão o mesmo hash. Se por acaso isso acontecer, nós chamamos de colisão. Embora seja possível acontecer, é extramente improvável, por esse motivo quanto melhor o algoritimo de Hash mais remota são as chances de colisão.

Falando em algoritimos, existem diversos algoritimos de Hash, e, sem dúvida, os mais conhecidos são MD5 e SHA-256. Embora o MD5 ainda seja bastante utilizado por sistemas a fora, seu uso é desencorajado no âmbito de proteção de dados (como senhas e PINs) há alguns bons anos por questões de vulnerabilidades de segurança. Por esse motivo, recomenda-se favorecer o uso de algortimos da familia SHA (Secure Hash Algorithm), que geram hashes mais fortes e com menores chances de conflito.

No mundo Java, podemos usar o algoritimo SHA3-256, na qual foi adicionado a plataforma em sua versão 9. Diferentemente do MD5, que gera hashes de 128 bits (16 bytes), esse algoritimo gera hashes de 256 bits (32 bytes). Para entender melhor o que estou tentando dizer, vamos gerar o hash do texto `"Zup Edu"` com ambos os algoritimos e imprimir suas representações em hexadecimal:

```
MD5     : c78f3f089f812efe2f94b133ff552b63
SHA3-256: ce6e066cff213bb0b9528dece4a3eeb7c3c7c357d9699a670c322822738a1ee6
```

Você pode experimentar a geração de hashes com MD5, SHA3-256 e diversos outros algoritimos de encriptação nesse site: [Online Tools](https://emn178.github.io/online-tools/index.html).

### Encriptando o CPF com SHA3-256

Agora que você já entende como algoritimos Hash funcionam, vamos implementar nossa função hash para encriptar o CPF de um destinatário. Para isso, vamos aproveitar a classe `CpfUtils` criada anteriormente e implementar um método estático que recebe uma `String` como entrada e devolve nosso hash gerado em formato textual (no caso, hexadecimal):

```java
public class CpfUtils {

    // outros métodos

    public static String hash(String cpf) {
        try {
            MessageDigest digest = MessageDigest.getInstance("SHA3-256");
            byte[] hash = digest.digest(cpf.getBytes(StandardCharsets.UTF_8));
            return toHex(hash);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
            throw new IllegalStateException("Erro ao gerar hash de um CPF: " + cpf);
        }
    }

    private static String toHex(byte[] bytes) {
        StringBuilder result = new StringBuilder();
        for (byte aByte : bytes) {
            result.append(String.format("%02X", aByte));
        }
        return result.toString();
    }
}
```

De maneira similar a anonimização do CPF, temos que gerar o hash do CPF antes de gravar a entidade no banco de dados, para isso vamos alterar nossa classe `Destinatario` com um novo atributo para armazenar o hash do CPF, na qual chamaremos de `hashDoCpf`; além disso, precisamos mover a constraint de unicidade do atributo `cpf` para este novo atributo. Por fim, não podemos esquecer de alterar o construtor da entidade para encriptar o CPF informado e atribui-lo ao novo atributo:

```java
@Entity
class Destinatario {

    // outros atributos
    
    @Column(nullable = false) // nao pode ser unico
    private String cpf

    @Column(nullable = false, unique = true) // unico
    private String hashDoCpf;

    public Destinatario(String nome, String telefone, String cpf) {
        this.nome = nome;
        this.telefone = telefone;
        this.cpf = CpfUtils.anonymize(cpf); 
        this.hashDoCpf = CpfUtils.hash(cpf); // gera hash do cpf
    }

}
```

Não menos importante, precisamos alterar a validação de unicidade no controller: no inicio do método devemos validar o novo atributo `hashDoCpf` em vez do atributo `cpf`:

```java
@Transactional
@PostMapping("/api/destinatarios")
public ResponseEntity<?> cadastra(@RequestBody ...) {

    String hashDoCpf = CpfUtils.hash(request.getCpf()); // encripta cpf informado
    if (repository.existsByHashDoCpf(hashDoCpf)) {
        throw new ResponseStatusException(HttpStatus.UNPROCESSABLE_ENTITY, "destinatário existente no sistema");
    }

    // restante do código
}
```

Consequentemente, temos que alterar a classe Repository para adicionar o novo método de consulta:

```java
@Repository
public interface DestinatarioRepository extends JpaRepository<Destinatario, Long> {

    public boolean existsByHashDoCpf(String hashDoCpf);
}
```

Pronto, não muito complicado, não é mesmo? Agora, ao exercitarmos o endpoint de cadastro novamente a aplicação irá inserir os novos destinatários no banco, mas desta vez com seus respectivos hashes dos CPFs gerados:

```sql
SELECT d.*
  FROM destinatario d
;

id|nome        |telefone      |cpf           |hash_do_cpf                                                     |
--+------------+--------------+--------------+----------------------------------------------------------------+
 1|Rafael Ponte|+5585988887777|691.***.***-72|21d7eda96d6d2ef88d80b390fc3b32079da0406481f89188727fdb61e48a1793|
 2|Jordi       |+5511966665555|413.***.***-91|4be3202f41835f914061636c7bd06006ccfda7177ff95411c04335dd5a9c21c2|
```

Se tentarmos cadastrar um novo destinatário com um CPF já existente a aplicação recusará a operação devido a validação de unicidade no controller ou, em caso de alta concorrência, via validação de contratint `UNIQUE` no banco de dados.

Se você reparou, após implementarmos encriptação via hash o atributo CPF anonimizado parece não ter mais utilidade para nós. Contudo, mantê-lo anonimizado na tabela pode ajudar em determinados fluxos de negócio, em relatórios internos, ou fluxos de confirmação de autenticidade por parte do usuário etc. De qualquer forma, lembre-se de consultar o especialista de negócio do seu time sobre a necessidade de manter a informação dentro do sistema.

Como vimos, graças a encriptação do CPF não só garantimos a unicidade dos destinatários de acordo com os requisitos de negócio como também evitamos armazenar dados sensíveis de forma aberta no banco de dados.

 ## Dicas do especialista

 ### Ajuste o tamanho da coluna de hash na tabela

Estamos armazenando o hash do CPF em uma coluna do tipo `Varchar(255)`, que é o default do Hibernate. Como temos certeza que um hash SHA3-256 terá sempre 32 bytes, podemos alterar o tamanho da coluna para `Varchar(64)`: defininos o tamanho como 64 caracteres pois o hash não será armazenado em formato binário, mas sim textual, neste caso hexadecimal.

Para configurar o tamanho da coluna, basta definir o atributo `length` da anotação `@Column`:

```java
@Entity
class Destinatario {

    // outros atributos
    
    @Column(nullable = false, unique = true, length = 64)
    private String hashDoCpf;

}
```

Definir corretamente o tipo e tamanho da coluna ajuda o banco de dados: armazenamento em disco, caching em memória, indexação etc. Embora armazená-lo em formato texto não seja o ideal do ponto de vista de otimização, é suficiente para maioria das aplicações e contextos, principalmente quando não há um grande volume de dados. Em caso de dúvidas, sempre consulte o DBA ou especialista de negócio sobre seu contexto, volumes de dados etc.

### Pensando de forma mais orientada a objetos

Criamos uma classe `CpfUtils` na aplicação para fazer a anonimização e hashing do número do CPF, contudo essa solução se trata de uma implementação producedural: separamos os dados de seus comportamentos. Idealmente poderíamos tirar proveito da orientação a objetos criando uma representação para CPF, como uma classe `CPF`, que não só armazenaria o número de um CPF como também encapsularia seus comportamentos.

Esse tipo de classe para representar tipos pequenos e simples do domínio chamamos de **Tiny Objects**. Uma possível implementação para esta classe `CPF` **dentro do nosso contexto** pode ser vista abaixo:

```java
@Embeddable
class final CPF {

    @Column(nullable = false)
    private String numero;

    @Column(nullable = false, unique = true, length = 64)
    private String hash;

    public CPF(String numero) {
        this.numero = anonymize(numero);
        this.hash   = hash(numero);
    }

    @Override
    public String toString() {
        return this.numero;
    }
}
```

A idéia de tiny objects é que eles sejam criados para encapsular lógicas de negócios, garantir integridade dos dados, aplicar validações, conversões, formatações etc. Eles podem ser bem úteis, mas nem sempre eles são necessários, principalmente quando não há comportamentos para determinado dado. O que estou querendo dizer, é que abusar deles em domínios triviais pode aumentar a complexidade do código sem trazer vantagens.

### Artigos que valem a pena a leitura

- [Regular expressions in Java - Tutorial](https://www.vogella.com/tutorials/JavaRegularExpressions/article.html)
- [Wiki: Data masking](https://en.wikipedia.org/wiki/Data_masking)
- [Data Masking: 8 Techniques and How to Implement Them Successfully](https://satoricyber.com/data-masking/data-masking-8-techniques-and-how-to-implement-them-successfully/)
- [Creating Hashes in Java](https://reflectoring.io/creating-hashes-in-java/)
- [SHA-256 and SHA3-256 Hashing in Java](https://www.baeldung.com/sha-256-hashing-java)

