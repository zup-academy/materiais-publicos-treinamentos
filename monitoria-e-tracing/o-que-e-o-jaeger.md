# O que é o Jaeger?

É um sistema para tracing distribuído que é de código aberto, criado pela Uber. Por isso seu nome em alemão significa "caçador".

Ele permite monitorar e ajudar a resolução de problemas em arquiteturas dsitribuídas e de microserviços.

O Jaeger foi aceito como um projeto de incubação da Cloud Native Computing Foundation (CNCF) em 2017 e promovido ao status de graduado em 2019.

Utilizar o Jeager nos permite ter :

- Propagação de contexto distribuído
- Monitoramento de transações distribuídas
- Análise de causa raiz
- Análise de dependência de serviço
- Otimização de desempenho/latência

## Rastreamento distribuído

Como costuma acontecer, mudar para um ecossistema de microsserviços traz muitos desafios. 

Entre eles está a perda de visibilidade do sistema e as interações complexas que agora ocorrem entre os serviços.

Fazer esse controle entre 10, 20, 30 microserviços já pdoe ser desafiador, agora imaginem quando essa quantidade ultrapassa a casa do milhares.

E foi assim que em 2015 o Uber se viu com a necessidade de além de monitoria padrão com métricas e análises era necessário ter visibilidade sobre a iteração ENTRE OS MICROSERVIÇOS.

Ferramentas de rastreamento distribuído nos permite olhar no contexto amplo entre o que ocorre não só em um serviço, mas entre toda uma cadeia de serviços que são executados durante um fluxo de negócio.


## Terminologia e componentes Jaeger

O Jaeger apresenta as solicitações de execução como traces, que mostram os dados/caminhos de execução através de um sistema.

Um trace é formado por um ou mais spans, que são as unidades de trabalho lógicas do Jaeger. Cada span inclui o nome da operação, a data/hora de início e a duração. Os spans podem estar aninhados e ordenados.

O Jaeger tem vários componentes que funcionam juntos para coletar, armazenar e visualizar spans e traces.

O Jaeger Client inclui implementações específicas da API OpenTracing por linguagem para rastreamento distribuído. Elas podem ser usadas manualmente ou com uma variedade de frameworks open source.

O Jaeger Agent é um daemon de rede que detecta spans enviados por meio do User Datagram Protocol (UDP). O agente deve estar no mesmo host da aplicação instrumentada. Normalmente, isso é implementado por meio de um sidecar em ambientes de containers, como o Kubernetes.

O Jaeger Collector recebe os spans e os coloca em uma fila para processamento.

Esses coletores precisam de um back-end de armazenamento persistente para que o Jaeger também tenha um mecanismo conectável para armazenar os spans.

Query é um serviço que recupera os traces do armazenamento.

O Jaeger Console é uma interface do usuário que permite visualizar os dados do rastreamento distribuído.



https://eng.uber.com/distributed-tracing/

https://www.jaegertracing.io/docs/1.34/

https://www.redhat.com/pt-br/topics/microservices/what-is-jaeger