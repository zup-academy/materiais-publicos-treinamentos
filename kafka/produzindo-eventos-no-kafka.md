# Produzindo eventos no Kafka

A produção do evento terá uma chave, se essa chave for preenchida o evento irá para uma partição de acordo com a chave.

### Acked
O acked é a confirmação do recebimento do evento no Kafka, ou seja, dessa forma o produtor não ficará tentando enviar a mensagem novamente.
Geralmente temos essa configuração no produtor.
Existem 3 opções no envio de mensagens : 0 ,1 e ALL
Sendo:
- acks=0 -> Apenas dispare e esqueça. Produtor não vai esperar por um reconhecimento.
- acks=1-> A confirmação é enviada pelo broker quando a mensagem é escrita com sucesso no líder.
- acks=all -> A confirmação é enviada pelo broker quando a mensagem é gravada com sucesso em todas as réplicas.

### Properties do Producer no Spring Kafka

**spring.kafka.producer.acks** : Número de confirmações que o produtor exige que o líder tenha recebido antes de considerar uma solicitação concluída.

**spring.kafka.producer.buffer-memory**: O tamanho total da memória que o produtor pode usar para armazenar em buffer os registros aguardando para serem enviados ao servidor.
Por padrão utiliza 32 MB de memória. Isso denota a memória total (em bytes) que o produtor pode usar para armazenar em buffer os registros a serem enviados ao broker. O produtor bloqueia até max.block.ms se buffer.memory for excedido. Se o Produtor estiver enviando registros mais rápido do que o broker pode receber registros, uma exceção será lançada.

**spring.kafka.producer.batch-size** : O valor padrão é 16K bytes. Ele é usado pelo Produtor para registros em lote. Essa configuração permite menos solicitações e permite que vários registros sejam enviados para a mesma partição.
Use essa configuração batch.size para melhorar a taxa de transferência e o desempenho de E/S no produtor e no servidor (e no consumidor).
Usar um batch.size maior torna a compactação mais eficiente. Se um registro for maior que o tamanho do lote, ele não será agrupado. Essa configuração permite que o Produtor envie solicitações contendo vários lotes.
Os lotes são por partição. Quanto menor o tamanho do lote, menor a taxa de transferência e o desempenho. Se o tamanho do lote for muito grande e muitas vezes for enviado antes de ficar cheio, a memória alocada para o lote será desperdiçada.

**spring.kafka.producer.client-id** : ID a ser passado ao servidor ao fazer solicitações. Usado para log do lado do servidor.

**spring.kafka.producer.compression-type** : Tipo de compactação para todos os dados gerados pelo produtor.

**spring.kafka.producer.key-serializer** : Classe de serializador para chaves.

**spring.kafka.producer.properties.** :  Propriedades adicionais específicas do produtor usadas para configurar o cliente.

**spring.kafka.producer.retries** : Quando maior que zero, permite a repetição de envios com falha.

**spring.kafka.producer.security.protocol** : Protocolo de segurança usado para se comunicar com corretores.

**spring.kafka.producer.ssl.** : demais propriedades relacionadas a ssl

**spring.kafka.producer.transaction-id-prefix** : Quando não vazio, habilita o suporte a transações para o produtor.

**spring.kafka.producer.value-serializer** : Classe de serializador para valores.
