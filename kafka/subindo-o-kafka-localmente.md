# Subindo o Kafka Localmente

## Subindo o Kafka pelo binário

Normalmente para executarmos o kafka localmente faríamos os seguintes passos:

- Acessar https://kafka.apache.org/downloads e fazer o download da versão mais recente. Baixar o arquivo compactado e descompactar ele.

- Subir o Zookeeper:
  - Acessar a pasta do kafka que você descompactou, entrar na pasta e em seguida abrir o terminal neste diretório. Executar o comando no seu terminal para subir o Zookeeper:

    ```shell
    bin/zookeeper-server-start.sh config/zookeeper.properties
    ```
    Ao final do log ele irá informar que se concetou a porta 2181.
- Subir o Kafka:
  - Acessar a pasta do kafka que você descompactou, entrar na pasta e em seguida abrir o terminal neste diretório. Executar o comando no seu terminal para subir o Kafka:
  
    ```shell
    bin/kafka-server-start.sh config/server.properties
    ```


## Subindo o Kafka pelo Docker

Porém durante esse curso vamos utilizar o docker para subir os serviços do Zookeeper, Kafka e KafDrop.

No exemplo abaixo estamos trabalhando com 3 instâncias de Broker pois iremos trabalhar com 3 replicas e 3 partições.

```yaml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    networks:
      - broker-kafka
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka1:
    image: confluentinc/cp-kafka:latest
    networks:
      - broker-kafka
    depends_on:
      - zookeeper
    ports:
      - 9092:9092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
  kafka2:
    image: confluentinc/cp-kafka:latest
    networks:
      - broker-kafka
    depends_on:
      - zookeeper
    ports:
      - 9093:9093
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:29092,PLAINTEXT_HOST://localhost:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
  kafka3:
    image: confluentinc/cp-kafka:latest
    networks:
      - broker-kafka
    depends_on:
      - zookeeper
    ports:
      - 9094:9094
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:29092,PLAINTEXT_HOST://localhost:9094
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
  kafdrop:
    image: obsidiandynamics/kafdrop:latest
    networks:
      - broker-kafka
    depends_on:
      - kafka1
    ports:
      - 19000:9000
    environment:
      KAFKA_BROKERCONNECT: kafka1:29092,kafka2:29092,kafka3:29092

networks:
  broker-kafka:
    driver: bridge
```

## Comandos Kafka pelo terminal 

Como vamos utilizar o comando através do Container Docker, vamos fazer o comando no broker líder. 

### Verificar comandos

Para saber os comandos possíveis você vai escolher qual o tipo de comando deseja encontrar:
- Usando o Binário:
```shell
bin/kafka-topic.sh 
```
- Usando o Docker:
```shell
docker-compose exec kafka1 kafka-topics
```

### Criar um Tópico

Executar o comando utilizando o Docker localmente:
```shell
docker-compose exec kafka1 kafka-topics --create --topic TESTE --partitions 3 --replication-factor 3 --bootstrap-server 127.0.0.1:9092
```

Executar o comando utilizando o binário localmente
```shell
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 3 --topic TESTE
```

### Consultar um Tópico

Executar o comando utilizando o Docker localmente:
```shell
docker exec -it kafka  kafka-topics --bootstrap-server :9092 --describe --topic TESTE
```

Executar o comando utilizando o binário localmente
```shell
bin/kafka-topics.sh --bootstrap-server :9092 --describe --topic TESTE
```

### Listar os Tópicos

Executar o comando utilizando o Docker localmente
```shell
docker exec kafka kafka-topics --list --bootstrap-server 127.0.0.1:9092
```

Executar o comando utilizando o binário localmente
```shell
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

### Enviar mensagem

Executar o comando utilizando o binário localmente
```shell
bin/kafka-console-producer.sh --bootstrap-server 127.0.0.1:9092 --topic TESTE
```

Executar o comando utilizando o Docker localmente
```shell
docker exec -it kafka kafka-console-producer --bootstrap-server 127.0.0.1:9092 --topic TESTE
```

### Consumir Mensagens

Executar o comando utilizando o binário localmente
```shell
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic TESTE
```

Executar o comando utilizando o Docker localmente
```shell
docker exec -it kafka_v1_kafka_1 kafka-console-consumer --bootstrap-server localhost:9092 --topic TESTE
```
### Consultar Consumer Group

Executar o comando utilizando o binário localmente
```shell
bin/kafka-consumer-groups.sh --bootstrap-server :9092 --describe --all-groups
```

Executar o comando utilizando o Docker localmente
```shell
docker exec -it kafka kafka-consumer-groups --bootstrap-server :9092 --describe --all-groups
```

#### Links de Referência.
https://www.baeldung.com/ops/kafka-docker-setup
https://github.com/obsidiandynamics/kafdrop
