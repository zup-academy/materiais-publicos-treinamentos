# Spring Cloud AWS

O starter Spring Cloud é um conjunto de projetos que nos ajuda a construir uma aplicação aderente aos princípios do 12 factors.

[12 Fatores](https://12factor.net/pt_br/)

Neste guarda-chuva de dependências, temos a parte relacionada aos serviços da AWS. que nos permite utilizar os principais serviços da AWS no nosso projeto.
O módulo que faz a integração com o serviço do SQS é o Spring Cloud AWS Messaging.

[Maven Respository - baixar dependência](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-aws-messaging)

O serviço do SQS permite apenas receber mensagens com tipo String, então qualquer objeto enviado ao SQS deve ser transformado para String antes de ser enviado na fila. 
O Spring Cloud AWS permite enviar objetos Java para SQS convertendo-os em String no formato JSON.

## Producer

Para enviarmos mensagens vamos utilizar a classe QueueMessagingTemplate, que possui os métodos para envio de mensagem, porém é necessário criarmos o Bean desse Objeto.
Criaremos o Bean através da classe client AmazonSQSAsync.

Devido ao limite de tamanho da mensagem que há no SQS, objetos que ultrapassem esse valor não serão enviadas. 

## Consumer
O SQS possui transações, então, as mensagens podem ser lidas duas vezes. Imagine que dois consumers da mesma aplicação leiam a mesma mensagem, isso pode gerar duplicidade e inconsistência nos dados ou ações geradas através da leitura da mensagem.
As aplicações precisam ter seu código desenvolvido de forma idempotente para lidar com essa duplicidade de leitura de mensagens que pode acontecer.


## Properties



[Documentação das configurações](https://docs.awspring.io/spring-cloud-aws/docs/current/reference/html/appendix.html)


#### Links de referência

https://docs.awspring.io/spring-cloud-aws/docs/current/reference/html/index.html#messaging
https://spring.io/projects/spring-cloud-aws
https://www.baeldung.com/spring-cloud-aws-messaging
https://montivaljunior.medium.com/utilizando-spring-cloud-com-aws-sqs-e-localstack-d5bf66ea3151
https://reflectoring.io/spring-cloud-aws-sqs/
https://cloud.spring.io/spring-cloud-static/spring-cloud-aws/2.0.0.RELEASE/multi/multi__messaging.html