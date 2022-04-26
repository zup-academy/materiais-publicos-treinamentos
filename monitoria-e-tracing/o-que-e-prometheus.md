# O que é o Prometheus?

Um conjunto de software open source de monitoramento e monitoria de aplicações.
Desenvolvido pela SoundCloud em 2012 e que logo foi abraçado por outras empresas.
O sucesso e adoção pelas outras empresas foi tanto que em maio de 2016, a Cloud Native Computing Foundation aceitou o Prometheus como seu segundo projeto incubado, depois do Kubernetes.

Características do Prometheus:

Os dados são salvos em linha do tempo.
Salva os dados com modelo chave valor.
Tem autonomia em seu armazenamento, ou seja, ele não precisa de um armazenamento externo para funcionar.
Possui uma linguagem própria(PromQL) para queries de dados em formato time series.
Tem suporte a outros tipos de forma de consulta.
A coleta das métricas ocorre via HTTP. Ou seja, a aplicação expõe um endpoint com as métricas e o Prometheus coleta através desse endpoint os dados de forma ativa, conhecida também como "scrape". Porém estes dados devem estar em um formato que o Prometheus consiga capturar corretamente.
Também é possível enviar métricas através de um gateway intermediário.
A definição dos serviços a serem monitorados pode ser feita através de uma configuração estática ou através de descoberta.
Possui vários modos de suporte a gráficos e painéis.

Porquê Prometheus?

Porque é uma ferramenta simples de usar e open source.
Permite configurar alertas.


Tipos de Métricas:

- Gauge - esse tipo de métrica permite soma e subtração de valores, podemos usar para casos de consumo de recurso, como por exemplo uso de memória ou CPU
- Counter - ele é o tipo de métrica acumulativa, ele sempre incrementa valor e zera em caso de reinicialização, podemos usar por exemplo como números de chamadas em uma api, número de consultas, etc
- histogram - Trabalha com comparação de dados, nesse caso ele é mais utilizado para comparação de intervalos de valores
- summary - Trabalha semelhante ao histogram porém é utilizado para comparação de porcentagem.

Exporter

Nem sempre é possível conectar o Prometheus ao serviço ou item que deseja capturar informações, neste caso são usados os exporters.
São processos que coletam as métricas dos itens desejados e expõe no modelo esperado pelo Prometheus.

Pushgateway

É um recurso de cache que permite armazenar os dados temporariamente que ainda não foram ingeridos pelo Prometheus. Usado em casos de aplicações ou operações que tenha uma curta duração de vida.

Prometheus WebUI

Podemos utilizar essa interface para criar dashboards, porém por não ter tantos recursos essa interface acaba não sendo muito utilizada.
A maioria das empresas acabam usando ferramentas que são bem mais avançadas para a visualização de métricas e que integram super bem com o Prometheus.
Uma das mais utilizadas é o Grafana.

Alert manager

Esse processo identifica quais regras estão configuradas e para onde e quando gerar esses alarmes.


Como configurar o Prometheus?

O arquivo de configuração do Prometheus se chama prometheus.yml.
O arquivo é dividido basicamente em 3 principais partes.
Configurações globais, configurações de alertas e manipulação dos dados que são coletados, e por fim a parte de scraps que são as configurações dos endpoints que o Prometheus vai capturar informações.

Exemplo de um arquivo de configuração do Prometheus:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'livraria'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8080']
```

Exemplo de docker-compose para subir via docker o Prometheus e o Grafana:

```yaml
version: '3.0'
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - prometheus-volume:/etc/prometheus/
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
  grafana:
    image: grafana/grafana
    volumes:
      - grafana-volume:/var/lib/grafana
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
volumes:
  grafana-volume:
  prometheus-volume:

```

