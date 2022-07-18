# Entendendo o que é o Kafka

Primeiro precisamos entender que quando pensamos em Kafka estamos falando de uma ferramenta muito potente.
Ela passa de longe messages Brokers de mercado, por isso é recomendado que caso seu uso seja de simples troca de mensagens entre sistemas utilize um broker comum.
Isso porque com a potência do Kafka vem a complexidade de se manter e se utilizar o Kafka.
Quando falamos de Kafka, estamos falando de um sistema distribuído que nos permite trabalhar com pipelines, armazenamento , stream , reprocessar, ter uma alta tolerência a falha de dados e trabalhar com arquiteturas complexas de processamento de dados.

Precisamos entender que a menor unidade que vamos falar do Kafka e que talvez o desenvolvedor tenha mais ligação, são os tópicos.
Outro ponto que precisamos ter em mente que Kafka é mais orientado a eventos.
É possível que sua utilização não seja com eventos, mas ele foi pensado para lidar com a abordagem de eventos.

Vamos compreender melhor o que são Partições, clusters, replica..:

## Cluster 
Cada execução do Kafka é em sua maioria denominada como nó.
O Cluster é o conjunto desses nós em uma única unidade. Nesse caso o Zookeeper realiza esse agrupamento e gerenciamento.
Ou seja, esses "nós" são em si broker Kafka em execução que em conjunto vários formam um cluster Kafka.
O uso de mais um broker permite uma maior tolerância a falhas, visto que mais de um Broker estará disponível para execução.

## Tópico
O Kafka trabalha com Tópicos, o que significa que para cada evento é necessário ter um tópico especifico, que é a únicade de armazenamento e gerencialmente menor do kafka.

## Consumer
Cada consumer é considerado por um id de grupo, esse identificador entenderá que somente 1 evento deverá ser lido por esse identificador.
Podemos ter vários consumers groups relacionados a um mesmo tópico, cada um com uma posição diferente de leitura dos records.

## Producer
Aplicação responsável por produzir o record no tópico.

## Partição
Cada Broker dentro do cluster gerencia um grupo de tópicos e cada tópico possui uma ou N partições, falo uma mas na prática não faz sentido usar o Kafka sem este recurso, o ideal e orientado pelos construtores da ferramenta é que seja pelo menos 3 o número de partições.

## Offset
Um tópico possui offsets, que são a sequência dos eventos dentro da partição do tópico, isso é utilizado para controlar em que posição de leitura esta cada consumidor.

## Record
O termo Record é gerado para cada evento/mensage/dado trafegado. Podemos considerar algo semelhante com o gerar uma mensagem em uma fila.
Cada record possui um offset 

## Replicas
Replicas são como cópias das partições, fazem os mesmo que uma partição, atuam para redundância dos dados, o que torna partições Kafka altamente disponíveis.
Ou seja, caso onde esta a partição falhe, ele ativará uma réplica, isso permite que sempre a partição esteja disponível para consumo ou produção de records.

## Log file
Os tópicos salvam registros em formato de log, ou seja, de forma estrutura e sequencial, quando nos referimos a log file estamos falando dos arquivos que contem a informação de um tópico.

