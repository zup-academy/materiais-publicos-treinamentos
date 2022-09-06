# Consumindo Eventos no Kafka

Um dos grandes pontos que devem ser levados em consideração ao consumir eventos do Kafka é a quantidade de partições, pois nada impacta para o produtor do evento mas o número de partições afetará muito no comportamento dos consumidores.

Cada consumer é identificado por um id de grupo, esse identificador entenderá que somente 1 evento deverá ser lido por esse grupo.

Toda aplicação que irá consumir um tópico deverá indicar qual o nome do seu consumer group, ou seja, básicamente o que indentificará que aquela aplicação esta consumindo eventos do tópico.

Através do consumer group o Kafka irá armazenar o offset que esta aquele consumer group e qual tópico ele consome.

Podemos ter vários consumers groups relacionados a um mesmo tópico, cada um com uma posição diferente de leitura dos records.

Ou seja, se analisarmos como um consumidor funciona em um sistema de filas, ele consome um evento apenas uma vez, porém diferentes consumers-groups podem consumir paralelamente a mesma mensagem usando assim o conceito de publish subscribe.

Outro adendo é que o fato de uma mensagem ser consumida não a remove do tópico, essa remoção é feita pelo tempo configurado de retenção das mensagens no Kafka.

Se a sua aplicação tem um consumer group e possui 4 instâncias em execução significa que um evento será consumido apenas 1 vez por alguma dessas máquinas, porém além disso precisamos entender que cada partição permite apenas um processamento consumir o evento, ou seja, cada instância ficará responsável por processar, o que indica que se o tópico tiver 3 partições, teríamos 1 instância ociosa, pois ela não teria uma partição para processar.

Em caso de ter mais partições do que consumidores no consumer group significa que cada instância de consumer irá receber eventos de mais partições.

O cenário ideal é quando temos um para um, pois isso possibilita um maior throughput.

### Commited
Indica estado da mensagem, se ela está disponível para ser consumida. Só fica disponível quando a mensagem está em todos os ISRs;
Quando ela se torna commited os consumidores podem ler.
Existem duas formas de realizar o Commit:
- Assincrona
  É uma confirmação não bloqueante, dessa forma a sua thread não será bloqueada. Se houver algum erro, o mesmo é passado através de um callback se fornecido, se não o erro é descartado.
- Sincrona
  Isto significa que a sua thread ficará bloqueada até receber uma confirmação de sucesso ou falha do Apache Kafka.

### Commited Offsets
É o parâmetro que indica o último offset com a leitura confirmada pelo consumidor. 
Por padrão o consumidor nunca irá ler essa mensagem novamente.


Quando todos os brokers de um cluster estão disponíveis, os produtores e consumidores podem escrever e ler a partir da partição líder do tópico sem problemas. 
Infelizmente os líderes e as réplicas podem falhar e precisamos saber lidar com cada uma dessas situações.

### Properties do Consumer Kafka

**spring.kafka.consumer.auto-offset-reset** : O que fazer quando não há deslocamento inicial no Kafka ou se o deslocamento atual não existe mais no servidor. Esse parâmetro somente é válido para a primeira vez que o consumer group se registra no tópico.
Opções:
- earliest: lê todos os eventos desde o inicio da partição do tópico
- latest: lê eventos que foram gerados depois que ele se registrou

**spring.kafka.consumer.auto-commit-interval** : Frequência com a qual os deslocamentos do consumidor são confirmados automaticamente no Kafka se 'enable.auto.commit' estiver definido como verdadeiro.

**spring.kafka.consumer.bootstrap-servers** : Lista delimitada por vírgulas de pares host:porta a serem usados ​​para estabelecer as conexões iniciais com o cluster Kafka. Substitui a propriedade global, para os consumidores.

**spring.kafka.consumer.enable-auto-commit** : utilizado para desabilitar o commit automático
No Spring Kafka o commit é automático e assíncrono. Porém se quisermos trabalhar com commit manual e sincrono não faremos através das properties da aplicação. Mas podemos desabilitar o commit automático.
Neste caso teríamos além de utilizar o parâmetro enable-auto-commit como instanciar um novo bean do listener passando acMode para manual e syncCommits como true:

**spring.kafka.consumer.client-id** : ID a ser passado ao servidor ao fazer solicitações. Usado para log do lado do servidor.

**spring.kafka.consumer.fetch-max-wait** : Quantidade máxima de tempo que o servidor bloqueia antes de responder à solicitação de busca se não houver dados suficientes para satisfazer imediatamente o requisito fornecido por "fetch-min-size".

**spring.kafka.consumer.fetch-min-size** : Quantidade mínima de dados que o servidor deve retornar para uma solicitação de busca.

**spring.kafka.consumer.group-id** : String exclusiva que identifica o grupo de consumidores ao qual esse consumidor pertence.

**spring.kafka.consumer.heartbeat-interval** : Tempo esperado entre pulsações para o coordenador do consumidor.

**spring.kafka.consumer.isolation-level** : Nível de isolamento para leitura de mensagens que foram gravadas transacionalmente.

**spring.kafka.consumer.key-deserializer** : Classe de desserializador para chaves.
