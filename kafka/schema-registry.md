# Schema Registry

O Kafka por si só não valida o que esta sendo recebido ou enviado, o problema só ocorrerá quando a aplicação consumidora tiver problema para consumir a mensagem no formato não esperado.
Para resolver essa situação a Confluente criou o Schema Registry, que nos permite validar o contrato e metadados das mensagens trafegadas.
De forma resumida, essa aplicação valida se a mensagem enviada é compatível com o modelo esperado que foi registrado .
É possível utilizar vários formatos de arquivos para validarmos nosso modelo como Json, XML, CSV.


## Problema

Existe um grande problema no dia a dia de consumidores e produtores de eventos Kafka que são os modelos que são trafegados, muitas das vezes o produtor gera um modelo de mensagem diferente e isso quebra os consumidores daquele evento que esperavam um outro modelo.
Para facilitar essa operacionalização do schema/contrato desses dados trafegados utilizamos os Apache Avro e para versionar os estados desse contrato utilizamos o Schema Registry.


## Evolução e Compatibilidade

Ao criar um schema no Schema Registry você deverá ter em mente qual a estratégia de evolução do contrato durante o ciclo de vida do mesmo.

Tipos de Estratégia:

* BACKWARD: novo esquema pode ser utilizado para ler a versão imediatamente anterior
* BACKWARD_TRANSITIVE: novo esquema pode ser utilizado para ler qualquer versão anterior
* FORWARD: uma versão imediatamente anterior é capaz de ler eventos escritos (serializados) com o novo esquema
* FORWARD_TRANSITIVE: qualquer versão anterior é capaz de ler eventos escritos (serializados) com o novo esquema
* FULL: esquema é BACKWARD e FORWARD ao mesmo tempo
* FULL_TRANSITIVE: esquema é FORWARD_TRANSITIVE e BACKWARD_TRANSITIVE ao mesmo tempo

Caso não configure a estratégia do schema, será herdada da configuração global.
O Schema Registry tem apis que nos permitem validarmos configurações e registros, dentre elas para lista a configuração global:

```shell
curl -X GET http://localhost:8081/config
```

### DOCKER
Subindo o Schema Registry 
```yaml
schema_registry:
    image: confluentinc/cp-schema-registry:latest
    hostname: schema_registry
    container_name: schema_registry
    networks:
      - broker-kafka
    depends_on:
      - zookeeper
      - kafka1
    ports:
      - "8081:8081"
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema_registry
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: kafka1:29092
      SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL: "zookeeper:2181"
      SCHEMA_REGISTRY_KAFKASTORE_LISTENERS: http://0.0.0.0:8081
      SCHEMA_REGISTRY_ACCESS_CONTROL_ALLOW_ORIGIN: "*"
      SCHEMA_REGISTRY_ACCESS_CONTROL_ALLOW_METHODS: GET,POST,PUT,DELETE,OPTIONS
      SCHEMA_REGISTRY_LISTENERS: "http://0.0.0.0:8081"

```

Subindo a ferramenta da Lences.io, acessar a interface gráfica pelo localhost:3030

```yaml
version: '3.7'

services:
  kafka-cluster:
    image: landoop/fast-data-dev:latest
    environment:
      ADV_HOST: 127.0.0.1         # Change to 192.168.99.100 if using Docker Toolbox
      RUNTESTS: 0                 # Disable Running tests so the cluster starts faster
      FORWARDLOGS: 0              # Disable running 5 file source connectors that bring application logs into Kafka topics
      SAMPLEDATA: 0               # Do not create sea_vessel_position_reports, nyc_yellow_taxi_trip_data, reddit_posts topics with sample Avro records.
    ports:
      - 2181:2181                 # Zookeeper
      - 3030:3030                 # Landoop UI
      - 8081-8083:8081-8083       # REST Proxy, Schema Registry, Kafka Connect ports
      - 9581-9585:9581-9585       # JMX Ports
      - 9092:9092                 # Kafka Broker
```

## Referências

https://docs.confluent.io/platform/current/schema-registry/index.html
https://blog.kafkabr.com/posts/esquemas-avro/
https://hub.docker.com/r/landoop/schema-registry-ui/
https://github.com/lensesio/schema-registry-ui
https://github.com/lensesio/fast-data-dev