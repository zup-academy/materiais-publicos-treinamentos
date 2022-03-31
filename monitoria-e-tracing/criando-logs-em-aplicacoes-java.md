# Criando logs em Aplicações Java.

A Maioria das bilbiotecas possui 3 classes principais de uso:
- Logger
- Appender
- Layout

- Logger é a classe de criação de mensagens de logs, a classe que será utilizada no seu código para criar os logs.
- Appender é a parte que determina para onde as mensagens de logs vão, um Logger pode ter mais de um appender. Quando pensamos em appenders, acabamos fazendo referência em qual arquivo de texto a mensagem será direcionada.
- O Layout é como a mensagem será gerada. É possível formatar como a mensagem final será gerada.

### Alguns pontos a consideras:

- É recomendado você utilizar as classes do Slf4j, pois elas permitem você trabalhar com as demais bibliotecas.
- Ao construir o seu aplicativo Java com Spring Boot não precisará se preocupar, pois, terá já a dependência do Slf4j para utilizar no momento que for implementar seus logs. 
- Apesar do Spring usar como padrão para os logs internos a biblioteca Commons Logging da Apache.
- O spring tem toda a estrutura necessária de que precisamos para gerar os logs. Se houver necessidade de configurar um ‘item’ diferente conseguirá. As configurações padrão são forneceidas para Java Util Logging , Log4J2 e Logback.
- A implementação padrão que o Spring adota é Logback.
- Por padrão, o Spring Boot gera logs apenas no console e não grava arquivos de log. Se desejar gravar arquivos de log além da saída do console, precisará definir uma propriedade logging.file ou logging.path no seu arquivo de configuração(por exemplo, em seu application.properties). Mas a grande maioria acaba implementando pelo arquivo de configuração do Logback.
- O nível de log padrão do Logger é predefinido para INFO, o que significa que as mensagens de nível TRACE e DEBUG não são visíveis. Mas é possível ativar esses logs através do parâmetro debug=true / trace=true no seu arquivo de configuração do projeto ou subir sua aplicação com o parâmetro --debug --trace. Mas também é possível determinar o nível de log pelo arquivo de configuração da biblioteca.
- Se o nível de log de um pacote for definido várias vezes usando as diferentes opções mencionadas acima, mas com diferentes níveis de log, será usado o nível mais baixo.
- Os arquivos de log são recriados quando atingem 10 MB e, como na saída do console, as mensagens ERROR-level, WARN-level e INFO-level são registradas por padrão. Os limites de tamanho podem ser alterados usando a logging.file.max-sizepropriedade. Os arquivos rotacionados anteriormente são arquivados indefinidamente, a menos que a logging.file.max-historypropriedade tenha sido definida. Estas opções também podem ser configuradas no Arquivo de configuração da biblioteca/implementação.

### Spring 
#### Os logs gerados pelo Spring em seu aplicativo seguirá o seguinte modelo:

Data e hora: precisão de milissegundos e facilmente classificável.
Nível de registro: ERROR, WARN, INFO, DEBUGou TRACE.
Identificação do processo.
Um ---separador para distinguir o início das mensagens de log reais.
Nome do thread: Entre colchetes (pode ser truncado para saída do console).
Nome do registrador: geralmente é o nome da classe de origem (geralmente abreviado).
A mensagem de registro.

Entender como funcionam os logs do Spring pode ajudar a entender melhor o comportamento e algum erro não esperado que foi lançado pelo Framework.

### Configurando o Logback para seus logs

Apesar de o spring Boot já vir com o Logback por padrão, as aconfigurações para as necessidades da sua aplicação podem ser diferentes e isso exije que você entenda as configurações do Logback.
Desde configurar Appenders, cores, a tamanho e formato do arquivo de log.
Para que o Spring Boot considere sobrepor as configurações o seu arquivo deve ser criado na pasta main/resources e conter um dos seguintes nomes:
logback-spring.xml
logback.xml
logback-spring.groovy
logback.groovy

#### Sintaxe do arquivo de configuração:

O arquivo de configuração é iniciado com a tag <configuration>, contendo zero ou mais <appender>, seguidos de zero ou mais <logger>, seguidos de no máximo um <root>.
O Logback não diferencia maiúsculas de minúsculas. Porém o nome da tag precisa estar no mesmo formato tanto na tag inicial quanto a final, caso ambas estejam em formatos diferentes não serão consideradas parte do mesmo par.
A tag root é essencial e determina o logger padrão, mas é possível que em seu arquivo tenha outros loggers com níveis diferentes e com appenders diferentes.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```
O layout define o formato de cada mensagem gravada. No logback é possível escrever seu próprio layout, mas a implementação padrão, o PatternLayout, é bem flexível. Ela utiliza variáveis (padrões) que são substituídas de acordo com a ocasião. Essas variáveis são precedidas pelo caractere %. Uma lista resumida de padrões dessa classe é a seguinte:

* %class – imprime o nome da classe de onde o método foi chamado
* %date (%d) – data atual (possível formatar)
* %level – o nível de logging
* %logger – imprime o nome do logger
* %line – número da linha de onde o método foi chamado
* %message (%msg) – a mensagem passada como parâmetro
* %thread – nome da thread
* %n – quebra de linha. Equivalente ao \n

```xml
<layout class="ch.qos.logback.classic.PatternLayout">
    <Pattern>
        %black(%d{ISO8601}) %highlight(%-5level) [%blue(%t)] %yellow(%C{1.}): %msg%n%throwable
    </Pattern>
</layout>
```
É importante entender que no caso do appender , layout e enconder você terá que determinar uma classe a ser utilizada.
Nesse caso é importante olhar na documentação quais classes possuem para cada item e quais possibilidades de parametrização uma classe esta habilitada ou não.
No caso acima citei sobre o PatternLayout e como é possível configurar a saída da mensagem no log através do uso dessa implementação.

#### Mas porquê multiplos Appenders?

Vamos considerar que você tem o arquivo de log geral e que você gostaria de gerar um log de uma funcionalidade em especial em outro arquivo, neste caso se faz necessário um Appender novo.
O caso mais trivial que temos nas aplicações é um Appender para console e outro para Arquivo.
O fato é que o Appender será necessário quando a fonte e a saíde de logs precisam estar separadas e providas de maneiras diferentes.

