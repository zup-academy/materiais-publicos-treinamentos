# O que é APM 

APM é Application Performance Monitoring ou Gerenciamento de desempenho de aplicativos, é uma forma de realizar o gerenciamento de desempenho de aplicações para trazer uma melhor experiência para os usuários e maior visibilidade para os desenvolvedores e responsáveis pela infraestrutura.

O Gartner criou um framework que determina as dimensões do APM. Essas dimensões vão orientar as áreas e objetivos do que devemos analisar e acompanhar em nossos softwares.

- End User Experience 
- Runtime Application Architecture
- Business Transactions 
- Deep Dive Component Monitoring 
- Analytics / Reporting

De maneira geral esses pontos vão derivar em métricas que serão analisadas e acompanhadas da Aplicação. Cada grupo com uma finalidade.

Vamos entender qual a finalidade de cada uma:

### End User Experience
Este monitoramento é realizado de forma sintética/proativa ou real. 
A forma sintética/proativa é realizada com a implementação de robôs emuladores que simulam o comportamento dos usuários.
A forma real é aplicada pela coleta de logs de aplicação ou ainda com plugins de softwares especializados em APM que são entregues junto com o código da aplicação. 
Os 2 tipos de monitoramento normalmente servem para popular dashboards e disparar alarmes quando ocorrer qualquer violação (ou tendência de violação) de tempo.

### Runtime Application Architecture
Permite monitorar o fluxo das transações, podendo ter visibilidade des caminhos de uma transação.
Isso é muito importante especialmente em arquiteturas distribuídas, onde um fluxo de negócio envolve muitas aplicações.

### Business Transactions
Métricas relacionadas ao negócio, como por exemplo quantidade de vendas realizadas no dia, quantidade de vendas por faixa etária.
Ter noção da volumetria que é produzida pela aplicação permite determinar estratégias para lançamentos ou até SLA para resolução de problemas.

### Deep Dive Component Monitoring
Métricas para monitoria relacionadas ao sistema e a infraestrutura , ter acesso a informações desse nível permite uma resolução de problemas mais rápida.
Porém esse tipo de monitoria muitas vezes pode gerar um custo e sobrecarga para a aplicação.

### Analytics / Reporting
Esse caso é referente a coleta e armazenamento de dados que podem ser analisados posteriormente.


Existem muitas ferramentas de APM no mercado, cada software com esse foco nos permite realizar coleta de multiplas dimensões do software.
Desde uma métrica de desempenho, tempo de resposta para uma determinada funcionalidade ou quantidade de erros no total comparado com a quantidade de requisições recebidas no dia.

