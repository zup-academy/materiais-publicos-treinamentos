# Problemas Kafka

Existe muitos cenários relacionados a problemas no Kafka e perda de dados.

## Primeira Situação - Problemas de sincronismo pós indisponibilidade

Quando temos um problema de indisponibilidade de algum broker do cluster, este estará fora de sicnronia com o líder, se por algum motivo todos os brokers ficarem fora e até o líder e os brokers que não estavam como líderes voltarem antes dele, essas mensagens podem ser perdidas.

Para estes casos é sugerido que o producer utilize como configuração :

ack = all
retries = 2147483647 (max integer)
max.block.ms = 9223372036854775807 (max long)

Lembrando que é necessário entender o cenário e o que seria prioridade: 
- consistência
- desempenho

## Segunda Situação - Problema com a configuração auto.offset.reset

Quando configuramos esse processamento como latest.
Exemplo: 
- Existem 3 instâncias de consumidores no mesmo consumer group 
- Todas estão configuradas como auto.offset.reset latest
- uma das instâncias fica indisponível durante a leitura de uma das mensagens, porém não consegue fazer o commit da leitura da mensagem
- É feito o rebalanceamento, uma das instâncias assume a partição da instância que ficou inativa
- Como esta parametrizado no consumidor para não ler mensagens desde o começo ele irá começar ler as mensagens a partir do momento em que se registrou na partição
- Resultará que aquela mensagem que não foi dado o commit não seja consumida.

## Terceira situação - Garantindo Ordenação

O kafka garante e não garante a ordenação das mensagens, então é necessário entender as necessidades para ordenação.

Por exemplo a organização do envio do evento dependerá da chave do evento enviado, então caso queira que mensagens correlacionadas sejam produzidas em sequencia, exemplo, você deseja que os e-mails de uma venda sejam encaminhados de acordo com a ordem de envio, nesse caso você pode utilizar como chave a venda, dessa forma eles ficarão na mesma partição e serão consumidas na sequência em que foram geradas.

Porém se precisa que todas as mensagens de um tópico como um todo sejam consumidas em ordem você não conseguirá se beneficiar do processamento e uso de partições, pois teria que ter somente uma.



