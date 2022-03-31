# Centralização de Logs

Muitas vezes para que um fluxo de negócio seja executado ele passa por diversos sistemas, cada sistema, pode ter multiplas instâncias em execução.
O grande desafio é juntar esses logs que foram produzidos em multiplos sistemas e máquinas diferentes em um local só para facilidar a pesquisa deles.
A centralização de logs é essencial para a análise de log distribuído de vários sistemas ou vários componentes de um aplicativo distribuído.
Tenha em mente que essas ferramentas podem armazenar, possuir interface de pesquisa e permitem criar dashboard, alertas e uma gama de funcionalidade que permite fazer melhor o gerenciamento através delas.


## Ferramentas

Elas fazem o trabalho de receber os logs de múltiplas origens e armazená-los e disponibilizar maneiras eficientes de busca considerando o volume de dados que é gerado através de logs.

Existem várias ferramentas disponíveis, dentre elas estão:

- Splunk
  - Splunk facilita o acesso aos dados em uma organização ao capturar, indexar e correlacionar esses dados em tempo real em um repositório pesquisável de onde o usuário pode gerar gráficos, relatórios, alertas, painéis e visualizações. O software pode identificar padrões de dados, fornecer métricas, diagnosticar problemas e fornecer inteligência para operações de negócios. O Splunk tem como função principal dar significado aos dados, e em muitos casos sem que o usuário precise fazer alguma coisa. Dessa forma, problemas que parecem complexos se tornam simples uma vez que é possível visualizar os todos os dados em um só lugar, em vez de ter que caçá-los e depois descobrir como agrupá-los.
  - Prós: Consulta rápidas, possui uma grande comunidade e muitos materiais de treinamento.
  - Contras: é pago.
  
- Logstash
  - É uma parte da pilha ELK, que inclui o ElasticSearch (sistema de busca e armazenamento em cluster) e o Kibana (web frontend para o ElasticSearch). Com centenas de plugins, o Logstash pode se conectar a uma variedade de fontes e transmitir dados em escala para um sistema de analytics centralizado. Construído com extensibilidade em mente, o Logstash fornece uma API para desenvolvimento rápido de plugins pela comunidade. Com as recentes melhorias do ecossistema de plugins, os colaboradores podem publicar novos plugins a qualquer momento. 
  - Prós : Livre e open source, possui grande integração com outros produtos Elastic, Funcionalidade estendida através de plugins, Os filtros são código
  - Contras : Não possui interface de usuário, os filtros podem ser difíceis de escrever, não possui alerta nativo

- Graylog
  - É um empacotador de código aberto que desempenha as mesmas funções do Splunk. Não tem a capacidade de ler diretamente de arquivos syslog. Em vez disso, é preciso enviar as mensagens diretamente para o Graylog, o que é menos conveniente. É possível executar buscas de dados exatamente como no Splunk e com funções de pesquisa semelhantes. Os alertas também são possíveis, mas os e-mails de alerta são menos informativos por conta própria, fornecendo apenas uma referência aos resultados de pesquisa contidos na interface web do Graylog.
  - Prós: Livre e open source, Streamings permitem identificar eventos em tempo real e executar ações, configuração fácil, a funcionalidade do lado servidor pode ser estendida através de plugins, possui REST API e Interface de pesquisa intuitiva
  - Contras : O Graylog só contempla suporte para syslog e GELF
 
- CloudWatch Logs 
  - O CloudWatch Logs permite centralizar os logs de todos os sistemas, aplicações e produtos da AWS que você usa em um único serviço altamente escalável. Você pode facilmente visualizá-los, pesquisá-los por códigos de erro ou padrões específicos, filtrá-los com base em campos específicos ou arquivá-los com segurança para análise futura. O CloudWatch Logs permite consultar todos os logs, qualquer que seja a origem, como um fluxo único e consistente de eventos ordenados por tempo. É possível consultá-los e classificá-los com base em outras dimensões, agrupá-los por campos específicos, criar cálculos personalizados com uma linguagem de consulta poderosa e visualizar dados de log em painéis.
  - Prós : facilidade em integrar com aplicações que estão na AWS, 
  - Contras : pago

    
#### Fonte original das informações:

    - https://blog.mandic.com.br/artigos/as-5-ferramentas-mais-populares-para-gerenciamento-de-log/
    - https://elven.works/conheca-as-12-ferramentas-mais-populares-para-gerenciamento-de-log/
    - https://docs.aws.amazon.com/pt_br/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html