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

## Referências

https://docs.confluent.io/platform/current/schema-registry/index.html
https://blog.kafkabr.com/posts/esquemas-avro/
https://hub.docker.com/r/landoop/schema-registry-ui/
https://github.com/lensesio/schema-registry-ui
https://github.com/lensesio/fast-data-dev