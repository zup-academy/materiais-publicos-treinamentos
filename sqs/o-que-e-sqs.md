# O que é o SQS

Como falamos antes para trabalharmos com mensageria temos a necessidade de um Broker de mensagens.
A AWS sabendo que muita da comunicação entre sistemas distribuídos se dá através de mensageria, criou os serviços para abstrair e facilitar o uso de mensageria utilizando o Broker como serviço.

Quando falamos que o uso se dá através de serviço, significa que não será necessário gerenciar a infraestrutura para o broker e apenas realizar o uso dele, além de ter facilidade em implementar juntamente com os serviços que já estão rodando na nuvem, bem como implementar segurança por exemplo com muito mais facilidade.

Assim como qualquer serviço novo que se adiciona da AWS a sua arquitetura, é esperado um maior vendor lockin, ou seja, quando não quiser mais utilizar o serviço ou quiser migrar de serviço de nuvem o acomplamento acaba sendo um pouco maior e gerando retrabalho nos pontos que utilizam o serviço.

No SQS há dois tipos de filas de mensagens, o tipo padrão que oferece throughput máximo, o melhor esforço de classificação e entrega pelo menos uma vez e o tipo de filas FIFO que garante que ordena da forma exata em que forem enviadas.

É um ponto de atenção entender o produto em si que é vendido.

As mensagens tem limitação de tamanho então é algo a se pensar para payloads de mensagem grandes. A mensagem pode conter até 256 KB, sendo que o modelo de cobrança considera cada 64kb uma cobrança, ou seja, para mensagens que atinjam 128kb seria gerada duas conbranças na mesma mensagem.
Em casos de exceção onde o tamanho da mensagem ultrapasse os 256kb, caso utilize Java em seu projeto poderá contar com uma biblioteca que foi desenvolvida para Java que compacta o payload da mensagem e descompacta.

O serviço armazena as mensagens por até 14 dias.

O serviço garante segurança na consistência da leitura de mensagens, visto que bloqueia a mensagem quando um consumidor esta lendo a mesma.

O Serviço não possui valor minimo para cobrança e permite a utilização de 1 milhão de solicitações gratuitamente, por mês. 

Os valores variam de região para região e considerem que as filas do tipo Padrão são mais baratas que as filas do tipo FIFO.

#### Link de referência

https://aws.amazon.com/pt/sqs/