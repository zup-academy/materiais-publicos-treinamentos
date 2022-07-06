# Usando o Localstack para o ambiente local

Muitas vezes para o desenvolvimento local o ideal é utilizarmos um serviço local que simule o Broker ou o próprio Broker em casos como ActiveMq ou outros projetos Open Source.
No caso do SQS podemos utilizar o LocalStack que é um projeto desenvolvido pela Atlassian e atualmente independente.
Este projeto nos permite subir localmente via Docker os principais serviços da AWS.

No nosso caso vamos subir o serviço do SQS.

Para Executar a AWS CLI pelo docker-compose: 

```shell
docker-compose exec localstack bash
```

#### Links de Referência 

https://www.zup.com.br/blog/localstack-simule-ambientes-aws-localmente
https://github.com/localstack
https://localstack.cloud/
