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
- https://dev.to/kafkabr/kafka-consumindo-registros-com-spring-40h8
- http://cloudurable.com/blog/kafka-tutorial-kafka-producer-advanced-java-examples/index.html
- 