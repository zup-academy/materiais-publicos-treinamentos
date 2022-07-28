# O que é o SQS

Como falamos antes para trabalharmos com mensageria temos a necessidade de um Broker de mensagens.
A AWS sabendo que muita da comunicação entre sistemas distribuídos se dá através de mensageria, criou os serviços para abstrair e facilitar o uso de mensageria utilizando o Broker como serviço.

Quando falamos que o uso se dá através de serviço, significa que não será necessário gerenciar a infraestrutura para o broker e apenas realizar o uso dele, além de ter facilidade em implementar juntamente com os serviços que já estão rodando na nuvem, bem como implementar segurança por exemplo com muito mais facilidade.

Assim como qualquer serviço novo que se adiciona da AWS a sua arquitetura, é esperado um maior vendor lockin, ou seja, quando não quiser mais utilizar o serviço ou quiser migrar de serviço de nuvem o acomplamento acaba sendo um pouco maior e gerando retrabalho nos pontos que utilizam o serviço.

No SQS há dois tipos de filas de mensagens, o tipo padrão que oferece throughput máximo, o melhor esforço de classificação e entrega pelo menos uma vez e o tipo de filas FIFO que garante que ordena da forma exata em que forem enviadas. A fila FIFO deve conter o prefixo .fifo em seu nome para que a fila seja criada com esse tipo e deve ser habilitado o parâmetro de tipo fifo, caso o mesmo não seja habilitado será assumido o tipo padrão.


É um ponto de atenção entender o produto em si que é vendido.

As mensagens tem limitação de tamanho então é algo a se pensar para payloads de mensagem grandes. A mensagem pode conter até 256 KB, sendo que o modelo de cobrança considera cada 64kb uma cobrança, ou seja, para mensagens que atinjam 128kb seria gerada duas conbranças na mesma mensagem.
Em casos de exceção onde o tamanho da mensagem ultrapasse os 256kb, caso utilize Java em seu projeto poderá contar com uma biblioteca que foi desenvolvida para Java que compacta o payload da mensagem e descompacta.

O serviço armazena as mensagens por até 14 dias.

O Serviço não possui valor minimo para cobrança e permite a utilização de 1 milhão de solicitações gratuitamente, por mês. 

Os valores variam de região para região e considerem que as filas do tipo Padrão são mais baratas que as filas do tipo FIFO.

## Diferenças entre SQS, MQ e SNS

O Serviço do SQS é serviço de fila e SNS serviço de tópico , eles são escaláveis e não exigem configuração de intermediário.

Já o MQ é um serviço de intermediário de mensagens gerenciado que fornece compatibilidade com muitos intermediários de mensagens populares. É recomenda o Amazon MQ para migrar aplicativos de agentes de mensagens existentes que dependem da compatibilidade com APIs como JMS ou protocolos como AMQP, MQTT, OpenWire e STOMP.

## Parâmetros da fila

- Tempo limite de visibilidade (Visibility timeout) - O intervalo é de 0 segundos a 12 horas. O valor padrão é 30 segundos.
- Período de retenção da mensagem (Message retention period) - O intervalo é de 1 minuto a 14 dias. O valor padrão é 4 dias.
- Atraso na entrega (Delivery delay) - O intervalo é de 0 segundos a 15 minutos. O valor padrão é 0 segundos.
- Tamanho máximo da mensagem (Maximum message size) - O intervalo é de 1 KB a 256 KB. O valor padrão é 256 KB. 
- Tempo de espera de recebimento da mensagem (Receive message wait time) - O intervalo é de 0 a 20 segundos. O valor padrão é 0 segundos, que define a sondagem curta . Qualquer valor diferente de zero define a sondagem longa.
- Habilitar a desduplicação de mensagem (Enable content-based deduplication) - Para tipo FIFO - para habilitar a desduplicação baseada em conteúdo. A configuração padrão está desabilitada.
- Politica de Permissão de redirecionamento (Redrive allow policy) -  escolha Allow all (o padrão), By queue ou Deny all . Ao escolher Por fila , especifique uma lista de até 10 filas de origem pelo nome de recurso da Amazon (ARN).
- Quantidade máxima de recebimento de mensagem ( Maximum receives) - quantidade de retentativas que uma mensagem foi tentada ser lida e voltou para a fila

## Usan SQS pelo AWS CLI
Vocês podem fazer uso e gestão das filas pelo console da AWS ou pelo AWS CLI.
Seja utilizando a conta da AWS ou Localstack você conseguirá utilizar AWS CLI.

Vamos ver alguns comandos que podemos realizar utilizando o Localstack:

- Listar url das filas existentes
  - aws --endpoint-url=http://localhost:4566 sqs list-queues
- Consultar a url de uma fila especifica
  - aws --endpoint-url=http://localhost:4566 sqs get-queue-url --queue-name teste
- Criar uma fila ( o nome da fila criada no exemplo é email)
  - aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name teste
- Criar uma fila Fifo
  - aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name teste.fifo --attributes FifoQueue=true
- Criar fila com DLQ
  - aws sqs create-queue --queue-name teste \
    --attributes '{"RedrivePolicy": "{\"deadLetterTargetArn\":\"arn:aws:sqs:sa-east-1:000000000000:teste_dlq\",\"maxReceiveCount\":\"2\"}"}' \
    --endpoint-url=http://localhost:4566
- Enviar uma mensagem para uma fila
  - aws --endpoint-url=http://localhost:4566 sqs send-message --queue-url http://localhost:4576/queue/teste --message-body Mensagem de teste
- Criar um listener / consumidor no CLI
  - aws --endpoint-url=http://localhost:4566 sqs receive-message --queue-url http://localhost:4566/000000000000/email
- Produzir uma mensagem
  - aws --endpoint-url=http://localhost:4566 sqs send-message --queue-url http://localhost:4566/000000000000/teste --message-body "teste paula"
- Deletar uma fila
  - aws --endpoint-url=http://localhost:4566 sqs delete-queue --queue-url http://localhost:4566/000000000000/teste
- Receber uma mensagem 
  - aws --endpoint-url=http://localhost:4566 sqs receive-message --queue-url http://localhost:4566/000000000000/teste
- Receber várias mensagens 
  - aws --endpoint-url=http://localhost:4566 sqs receive-message --queue-url http://localhost:4566/000000000000/teste --attribute-names All --message-attribute-names All --max-number-of-messages 10
- Consultar informações sobre a fila
  - aws --endpoint-url=http://localhost:4566 sqs get-queue-attributes --queue-url http://localhost:4566/000000000000/teste_dlq --attribute-names All


Estes e outros comandos do SQS no AWS CLI você consegue ver nesse link da documentação oficial:
[Documentação Oficial SQS AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/sqs/)

#### Link de referência
https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues.html
https://aws.amazon.com/pt/sqs/
https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-message-metadata.html#sqs-message-attributes