# Spring Kafka

Dependência para pom.xml:
```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```
Dependência para build.gradle:
```groovy
implementation group: 'org.springframework.kafka', name: 'spring-kafka', version: '2.9.0'
```

## Propriedades:

_spring.kafka.properties.* é considerada para qualquer propriedade comum pro producer e consumer_

**spring.kafka.bootstrap-servers** : Lista delimitada por vírgulas de pares host:porta a serem usados ​​para estabelecer as conexões iniciais com o cluster Kafka. Aplica-se a todos os componentes, a menos que seja substituído.

### Producer

**spring.kafka.producer.acks** : Número de confirmações que o produtor exige que o líder tenha recebido antes de considerar uma solicitação concluída.

**spring.kafka.producer.buffer-memory**: O tamanho total da memória que o produtor pode usar para armazenar em buffer os registros aguardando para serem enviados ao servidor.

**spring.kafka.producer.batch-size** : Tamanho de lote padrão. Um tamanho de lote pequeno tornará o lote menos comum e pode reduzir o rendimento (um tamanho de lote igual a zero desabilita totalmente o lote).

**spring.kafka.producer.client-id** : ID a ser passado ao servidor ao fazer solicitações. Usado para log do lado do servidor.

**spring.kafka.producer.compression-type** : Tipo de compactação para todos os dados gerados pelo produtor.

**spring.kafka.producer.key-serializer** : Classe de serializador para chaves.

**spring.kafka.producer.properties.** :  Propriedades adicionais específicas do produtor usadas para configurar o cliente.

**spring.kafka.producer.retries** : Quando maior que zero, permite a repetição de envios com falha.

**spring.kafka.producer.security.protocol** : Protocolo de segurança usado para se comunicar com corretores.

**spring.kafka.producer.ssl.** : demais propriedades relacionadas a ssl

**spring.kafka.producer.transaction-id-prefix** : Quando não vazio, habilita o suporte a transações para o produtor.

**spring.kafka.producer.value-serializer** : Classe de serializador para valores.

### Consumer

**spring.kafka.consumer.auto-offset-reset** : O que fazer quando não há deslocamento inicial no Kafka ou se o deslocamento atual não existe mais no servidor.

**spring.kafka.consumer.auto-commit-interval** : Frequência com a qual os deslocamentos do consumidor são confirmados automaticamente no Kafka se 'enable.auto.commit' estiver definido como verdadeiro.

**spring.kafka.consumer.enable-auto-commit** : Se a compensação do consumidor é confirmada periodicamente em segundo plano.


### Retry

**spring.kafka.retry.topic.attempts** :(Valor Padrão 3) Número total de tentativas de processamento feitas antes de enviar a mensagem ao DLT.

**spring.kafka.retry.topic.delay** : ( Valor Padrão 1s) Período de backoff canônico. Usado como valor inicial no caso exponencial e como valor mínimo no caso uniforme.

**spring.kafka.retry.topic.enabled** : (Valor Padrão False ) Se as tentativas sem bloqueio baseadas em tópicos devem ser habilitadas.

**spring.kafka.retry.topic.max-delay** : ( Valor Padrão 0) Espera máxima entre as tentativas. Se for menor que o atraso, o padrão de 30 segundos será aplicado.

**spring.kafka.retry.topic.multiplier** : ( Valor Padrão 0) Multiplicador a ser usado para gerar o próximo atraso de backoff.

**spring.kafka.retry.topic.random-back-off** : ( Valor Padrão false ) Se deve ter os atrasos de backoff.

#### Links de referência 

- https://docs.spring.io/spring-boot/docs/current/reference/html/messaging.html#messaging.kafka
- https://spring.io/projects/spring-kafka/#overview
- https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties.integration.spring.kafka.admin.client-id
- 