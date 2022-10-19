# Resiliência utilizando Kafka

O conceito de resiliência não se aplica só na computação, quando falamos desse tema dentro da computação, estamos nos referindo a capacidade de lidarmos com falhas e problemas e o quão rápida será nossa recuperação.
Quando estamos falando de kafka, ao produzirmos ou consumirmos eventos podemos nos deparar com  falhas e erros que podem ocorrer, sendo de duas formas: os recuperáveis e os irrecuperáveis.

- Erros irrecuperáveis: por exemplo, se o broker retornar uma exceção INVALID_CONFIG, se o código executar a mesma solicitação novamente não mudará o resultado.
- Erros recuperáveis: esses erros podem ser resolvidos após uma retentativa. Por exemplo, se o broker retornar a exceção NotEnoughReplicasException, o produtor poderá tentar enviar a mensagem novamente - talvez os replica brokers voltem a ficar online e a segunda tentativa seja bem-sucedida. 

Para lidarmos com problemas que podem ocorrer existem diversas técnicas, vamos analisar mais sobre elas.

## Retry Simples

A técnica de retentativa é um abordagem muito utilizada quando o assunto é sistemas distribuidos.
Retentativas são utiliadas para lidarmos com erros transitórios, como, por exemplo, quando não há disponibilidade, problemas relacionados a rede, ao invés de falhar diretamente.

Porém, quando falamos de retentativa em fluxos relacionados ao Kafka precisamos entender quando e como usar para extrairmos o melhor dessa abordagem.

### Quando

Existentem situações que quando ocorrem erros há possibilidades de recuperação:

- Indispoibilidade do broker
- Erro de conexão de rede
- Exceção interna do Kafka, como um offset que não esta disponível.

### Como

- A configuração de retry determina quantas vezes o produtor tentará enviar uma mensagem antes de marcá-la como falha.
- Geralmente é combinado o uso com a configuração do parâmetro delivery.timeout.ms para o controle de tempo de repetição
- É recomendado o uso do parâmetro que habilita idempotência, para evitar que uma mensagem seja registrada mais de uma vez, a documentação oficial indica que se o parâmetro enable.idempotence estiver como false e o parâmetro max.in.flight.requests.per.connection para 1 gerará provavelmente alteração na ordem dos registros, pois se tivermos dois envios e o primeiro deles não for bem sucessido e o segundo sim, os registros do segundo estariam registrados em primeiro lugar.Claro que se você não tem necessidade de garantir a ordem de envios deve analisar se vale a pena se preocupar com isso visto que isso diminuiu potencialmente a performance.
- É importante termos em mente que essa abordagem soluciona a maioria dos casos de problemas
- É importante ter em mente que será preciso delimitar um tempo para retry

```yaml
spring:  
  kafka:
    retry:
      topic:
        #        Número total de tentativas de processamento feitas antes de enviar a mensagem ao DLT.
        attempts:
        #        Período de backoff canônico. Usado como valor inicial no caso exponencial e como valor mínimo no caso uniforme.
        delay:
        #        Espera máxima entre as tentativas. Se for menor que o atraso, o padrão de 30 segundos será aplicado.
        max-delay:
        #        Se as tentativas sem bloqueio baseadas em tópicos devem ser habilitadas.
        enabled:
        #        Multiplicador a ser usado para gerar o próximo atraso de backoff.
        multiplier:
        #        Se deve ter os atrasos de backoff.
        random-back-off:
```

## Referências

