# O que é Micrometer?

O Micrometer é uma biblioteca de instrumentação de métricas para aplicativos baseados em JVM, foi criado com o intuito de ser um facade para implementações de Métricas.

O Micrometer incorpora conceitos de normalização de convenção de nomenclatura, unidade básica de escala de tempo e suporte para expressões proprietárias de estruturas como dados de histograma que são essenciais para fazer as métricas brilharem em cada sistema de destino.
Ao longo do caminho, foram adicionados filtragem de medidores, permitindo que você exerça maior controle sobre a instrumentação de suas dependências upstream.

Por padrão quando utilizamos o Micrometer com o Spring são geradas métricas automáticamente de:
- JVM
- Pools de memória e buffer 
- Estatísticas relacionadas à coleta de lixo 
- Utilização de thread 
- Número de classes carregadas/descarregadas 
- utilização do CPU 
- Latências de solicitação Spring MVC e WebFlux 
- Latências RestTemplate 
- Utilização de cache 
- Utilização da fonte de dados, incluindo métricas do pool HikariCP
- Fábricas de conexão RabbitMQ
- Uso do descritor de arquivo
- Logback: registre o número de eventos registrados no Logback em cada nível
- Uptime: informe um medidor de tempo de atividade e um medidor fixo representando o tempo de início absoluto do aplicativo 
- Uso do Tomcat

É possível gerar métricas para :
- Netflix Atlas
- CloudWatch 
- Datadog 
- Ganglia 
- Graphite 
- InfluxDB 
- JMX 
- New Relic 
- Prometheus 
- SignalFx 
- StatsD (Etsy, dogstatsd, Telegraf, and proprietary formats)
- Wavefront 
- AppOptics 
- Azure Application Insights 
- Dynatrace 
- Elasticsearch 
- StackDriver