# Implementando API REST com chave de unicidade para dados sensíveis

Nesse conteúdo veremos como podemos implementar uma API REST responsável por cadastrar um registro único no sistema, onde sua chave de unicidade será uma informação sensível que não pode ser armazenada abertamente no banco de dados. Vamos partir da construção da API desde a escrita do controller via módulo Spring Web até a persistência com Spring Data, JPA e Hibernate.

## Cadastrando endereço de entrega do cliente

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

Sabemos que precisamos anonimizar o CPF do destinatário, e para isso podemos escrever um algoritimo bastante simples que substitua alguns números do CPF por um caractere de coringa, como `'*'` ou `'x'`, ou seja, algo como:

```
691.915.220-72 -> 691.***.***-72
```

Esse simples "replacement" no CPF impossibilita a identificação do cliente ou o uso indevido do CPF para fraudes em caso do sistema vir a ser invadido ou ter os dados vazados na internet. Apesar de existirem bibliotecas Java que ajudam nesse tipo de tarefa, podemos implementar este algoritimo utilizando somente a linguagem Java. Por exemplo, podemos criar uma classe `CpfUtils` com um método estático chamado de `anonymize`, semelhante a este:

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

Nossa solução de fato funcionou como esperado, mas o que acontece se tentarmos cadastrar 2 destinatários com os CPFs abaixo?

```
Yuri   , 540.602.880-47
Alberto, 540.301.164-47
```

Repare que ambos os CPFs possuem o prefixo e sufixo iguais, e isso que dizer que ao anonimiza-los eles acabaram com o mesmo valor:

```
Yuri   , 540.***.***-47
Alberto, 540.***.***-47
```

E, como já sabemos, por haver uma chave de unicidade na tabela o banco de dados não permitirá gravar o segundo destinatário, lançando um erro de violação de constraint semelhante a este:

```log
org.postgresql.util.PSQLException: ERROR: duplicate key value violates unique constraint "uk_zztn7wgfxo0xytrfby23f72up"
  Detail: Key (cpf)=(540.***.***-47) already exists.
```

 A verdade, é que no momento que anonimizamos o CPF por se tratar de um dado sensível, nós abrimos mão de sua caracteristica mais importante para nós nesse momento: sua unicidade. O que estou querendo dizer, é que a partir de agora o indice de conflitos entre CPFs de destinatários distintos aumentou substancialmente, o que inviabiliza seu uso no sistema. 

 Mas como anonimizar o CPF e ao mesmo tempo manter como chave de unicidade no nosso sistema?

 ## 
