# Como enviar logs no Cloudwatch

Geralmente nossas aplicações geram um arquivo de log, para monitorarmos esse arquivo de log usando o Cloudwatch teremos que instalar o agente no servidor da aplicação, além de ser necessária a configuração do agente, será necessário configurar polices e rules para autorizar o agente realizar envio de informações para o Cloudwatch.

Esse agente tem o intuito que coletar não só informações do arquivo de log mas também pode capturar informações sobre a aplicação e infraestrutura e enviá-las para a AWS Cloudwatch.

Geralmente a instalação do agente e configuração ficam abstraídas para o desenvolvedor , sendo necessário apenas que você entenda qual o grupo de logs ao qual sua aplicação pertence para que consiga realizar pesquisas no Cloudwatch.

Você terá disponível não só consulta de logs e sim, uma gama de ferramentas de monitoria, como configuração de alertas, dashboards.


