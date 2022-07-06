# O que é Mensageria?

Antes de falarmos o que é mensageria, temos que entender sobre o que é integração de sistemas.

## Integração de Sistemas

Vamos pensar em uma aplicação Web e Mobile de pedido de comida, essa aplicação tem como expertise a gestão de tudo que envolve o fluxo do pedido de comida, desde gestão do restaurante, dos pedidos, da entrega, porém para realização do pagamento será necessário falar com uma outra aplicação que tenha o dominio da parte financeira de pagamentos.
Dito isso entendemos que é impossível que sua empresa tenha dominio de tudo, em algum dado momento ela precisará falar com outra aplicação.
Essa comunicação entre estes dominios/sistemas pode se dar de diversas formas.
Desde o bom e velho txt ou xml em formatos de arquivos, até usando sistemas intermediários para a gestão dessa comunicação.
Precisamos entender que integrar sistemas não é algo simples, cada empresa modela seu sistema de forma diferente e tentar manter essa comunicação com o minimo de acoplamento pode ser extremamente desafiador.

Integrar sistemas pode ser muito mais comum hoje onde arquitetura de microsserviços são mais frequentes, tornando cada microsserviço uma possível peça nesse quebra cabeça da integração.
Porém integrar sistemas é muito mais antigo do que imaginamos, são décadas que ferramentas e bibliotecas vem evoluindo e tornando essa comunicação melhor.

## Voltando a Mensageria

No uso de mensageria existem alguns pontos principais que devemos entender, dentre eles que vocês utilizarão um sistema intermediário, este sistema geralmente é chamado de Barramento, Message Broker, Broker, MOM (Message oriented middleware). 
O Broker tem como objetivo gerenciar as mensagens entre produtores e consumidores.
Os produtores(producer) são quem geram a mensagem e os consumidores(consumer) são quem recebem a mensagem.
Quando utilizamos o termo Queue estamos nos referindo a uma fila que pode ter o propósito de receber mensagens especificas, pode ser que em um projeto vocês sejam produtores ou consumidores de várias filas.

O uso de mensageria pode se confundir muitas vezes com o uso de eventos, então para não errar, entenda que um evento é uma mensagem, porém nem toda mensagem é um evento. O evento é sobre o como o esquema de mensagens esta arquitetado e sobre a intenção da mensagem. O papel dos produtores e consumidores mudam um pouco nesse tipo de abordagem. Mas a nível de implementação de código são bem semelhantes.
Cada Broker de mercado pode abordar modelos de gestão das mensagens diferentes.
Entre elas, fila e tópico, sendo que cada uma pode ter um esquema de organização diferenciado.

### Broker
O Broker por si só é um sistema pronto de mercado que irá trabalhar com o modelo de filas e tópicos, fazendo a gestão de recebimento, armazenamento, consumidores por fila ou tópico e a leitura de mensagens.
Ele vai implementar protocolos de comunicação de Mensageria, possíbilitando a comunicação de seu sistema com ele através desses protocolos.
Existem brokers open source e pagos. É claro que para os modelos open source algum time que terá que gerenciar a infraestrutura dessa aplicação, além da manutenção e parametrização.
Existem diversos Brokers famosos, dentre eles :
- SQS
- SNS
- Kafka
- RabbitMQ
- ActiveMQ

### Filas
Você provavelmente já foi a um mercado e lá teve que pegar uma fila.
Vamos pensar que os caixas são os consumidores, as pessoas na fila são as mensagens que estão em uma fila do Broker e o produtor das mensagens nesse caso é o próprio mercado.
Se analisarmos o que percebemos logo de cara, cada pessoa / mensagem na fila é único, cada caixa/consumidor processa uma mensagem por vez mas ambos tem a mesma finalidade que é processar a compra, uma mensagem/ pessoa não é processada mais de uma vez, existe ordenação.
Sobre a ordenação, chamamos o tipo FIFO(First in, first out) o tipo de ordenação mais comum que utilizamos para filas, ou seja no mercado quem entra na fila primeiro é quem vai ser atendido primeiro, o mesmo ocorre para filas que possuem esse tipo de ordenação, as mensagens geradas primeiro, são processadas primeiro.
Existem outros modelos, mas isso pode depender do modelo de Broker que você irá utilizar.

### Tópicos
Quem nunca fez inscrição para receber alertas/notificações de publicações, ou de um podcast que lançou um episódio novo, uma atualização de um amigo em uma rede social.
Então, o tópico trabalha no modelo pub/sub, onde você pode ter vários producers e consumers, porém aqui cada consumer tem um objetivo diferente.
Nesse modelo existem formas de realizar ordenação e garantia de entrega dependendo do broker utilizado.

## Comunicação Asincrona
Quando trabalhamos com mensageria, começamos a lidar com as vantagens e desvantagens da comunicação asincrona.
Diferente de quando requisitamos um serviço diretamente, por exemplo, usando o protocolo http, onde realizamos a requisição e esperamos sua resposta, quando trabalhamos com mensageria não sabemos quando teremos a mensagem, então devemos trabalhar em paralelo para sabermos quando essa mensagem chegará e o que deve ser executado quando a mensagem for processada pelo consumidor.

Claro que quando falamos de comunicação asincrona entre sistemas não existe apenas mensageria, mas esta é a mais popular dentre elas.
Muitas vezes estamos habituados a pensarmos apenas em fluxos sincronos, entendermos fluxos asincronos requer que mude a forma como exergamos uma solução.

## Payload

O conteúdo da mensagem assim como trafegamos por uma api, possui o body e cabeçalho, geralmente quando usamos o termo Payload estamos nos referindo ao conteúdo da mensagem que esta sendo trafegada.

#### Materiais de Apoio 

http://www.argonavis.com.br/cursos/eip/Livro_EIP.pdf
https://aws.amazon.com/pt/message-queue/
https://aws.amazon.com/pt/message-queue/benefits/
https://en.wikipedia.org/wiki/Message_broker
https://microservices.io/patterns/communication-style/messaging.html
