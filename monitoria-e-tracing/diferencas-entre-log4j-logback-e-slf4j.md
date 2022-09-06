# Diferenças entre Log4j, Logback e slf4j

O Java possui uma api que faz parte da plataforma Java, no pacote java.util.logging é possível encontrar classes que permitem estruturar e criar os registros dentro de uma aplicação.
É importante entender que quando esse pacote foi implementado algumas outras estruturas externas da plataforma já eram amplamente utilizadas pelo mercado como a biblioteca do "Apache Commons Logging".
Então existia uma falta de padrão ao implementar logs, para isso surgiram wrappers para resolver esses problemas.

### Slf4j

Simple Logging Facade for Java - SLF4J é uma api que abstrai toda a estrutura para logs no Java. Ela surgiu como uma alternativa para integrar com o pacote do Java java.util.logging.
O Slf4j é uma abstração que facilita que ao utilizar ela você não terá problema com demais outras implementações que seguem o mesmo modelo da abstração.
Basicamente ao utilizar o SLF4J você estará tranquilo em relação a qual implementação esta utilizando, isso permite que diferentes estruturas de log coexistam. E ajuda a migrar de uma estrutura para outra. Por fim, além da API padronizada, também oferece uma sintaxe simples.

### Log4j

Log4j é uma biblioteca parte do Apache Logging Services, um projeto da Apache. Criado em 2002
É uma das estruturas existentes para trabalhar com Log em aplicações Java. Ela possui basicamente as mesmas funcionalidades que já constam no pacote de Logging do Java, porém acrescentando algumas novas funcionalidades.
Essa biblioteca permite criar configuração através de códigoou de arquivos, xml, json ou yaml.

#### Log4j 2

É uma evolução do projeto do Log4j, com suporte a muitos recursos novos. Além disso, se inspirou nas melhorias implementadas pelo Logback.
Um dos destaques na biblioteca é a possibilidade de registro dos logs de forma assincrona.


### Logback

Logback foi construido com base no que o Log4j não tinha, por isso assim que ele foi lançado foi amplamente utilizado.
A construção dele partiu da insatisfação com a biblioteca do Log4j e na demora da implementação de novas atualizações.

