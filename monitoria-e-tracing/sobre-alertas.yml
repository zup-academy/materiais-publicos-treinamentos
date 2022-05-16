#### Fonte
Este material foi baseado no livro Effective Monitoring and Alerting do autor Slawek Ligus.
Para mais informações recomendamos a leitura do mesmo.

# Alertas

A forma com que configuramos e somos acionados por notificações que nos informem erros ou problemas em tempo de execução em nossos sistemas.
Existem formas diferentes que as maiorias das empresas adotam, nem sempre o que é considerado uma anomalia para uma empresa é considerada para outra.

Existem formas de analisar o nível de urgência de uma ocorrência.
Para isso precisamos entender as prioridades da empresa, a capcidade de recuperação no sistema monitorado e do impacto gerado quando as coisas dão errado.

É impossível garantir que estaremos acompanhando cem por cento do tempo gráficos e dashboards da nossa aplicação, ter pessoas dedicadas para isso é inviável.
Dessa forma trabalhar com alertas é uma maneira de garantir que quando algo não esta bom seremos informados , de preferência antes dos clientes reclamarem.

Existe uma preocupação maior em três pontos principais:

- Qualidade do Serviço
- Tempo de Resposta
- Disponibilidade do Serviço

A gravidade estabelecidade do Alerta irá determinar o tempo e esforço para reagir ao problema informado.
O receptor do alerta deve isolar e identificar a origem do problema e mitigar o impacto no menor tempo possível conforme a gravidade estabelecida.
  
Trabalhar com alertas deve facilitar a resposta humana a falhas mecânicas e impulsionar a melhoria contínua. 
Mas para atingir esses objetivos, o alerta requer alguns componentes básicos da infraestrutura de TI, sem os quais uma organização madura simplesmente não pode existir.

É necessária a utilização de uma plataforma de monitoramento que suporte e seja facil de implantar agentes de coleta de dados, que tenha um mecanismo de plotagem flexível e rico em recursos que permita representar graficamente várias séries temporais em um único gráfico, e que tenha um mecanismo de alerta que possa suportar configurações de alertas, como agregação.

A configuração de alerta começa com a percepção do custo da falha e a importância de reagir a tempo.
O custo pode ser entendido e expresso de várias maneiras:
  - tempo
  - dinheiro
  - esforço
  - qualidade
  - prestígio
  - confiança

Cada organização pode apresentar sua própria definição.
O resultado final é que cada falha incorre em custos correspondentes à extensão do impacto da falha. 
O primeiro passo no caminho para um alerta eficaz é apresentar um alcance claro de significado para definir as prioridades corretas.

## Estabelecendo Significado
  
#### Crítico
  Problema instantaneo com impacto grave que bloqueia acesso ou utilização do usuário ou prejudica gravemente o funcionamento do sistema. 
  Uma falha na prevenção de eventos críticos aumenta o tempo de inatividade do sistema e se traduz diretamente em perdas irrecuperáveis de produtividade ou receita.
  
#### Urgente
  Problemas que resulta em perda parcial da disponibilidade, quando uma fração dos usuários não consegue acessar o sistema ou uma parte significativa do sistema deixa de responder. 
  Esses tipos de eventos exigem intervenção rápida para minimizar seu impacto e evitar que se transformem em eventos críticos.
  
#### Intervenção necessária
  Problemas que necessitam intervenção para evitar que se intensifiquem em eventos críticos. 
  Eles não são imediatamente catastróficos, mas o sistema corre um alto risco de entrar em colapso sem tratamento.
  Por exemplo, o sistema pode acumular uma lista de pendências que leva a atrasos no processamento.
  
#### Recuperável
  Problemas que geram efeito relativamente não crítico no sistema, mas com potencial para evoluir para problemas maiores se não forem recuperados dentro de um prazo esperado. 
  Por exemplo, a falha de uma tarefa de redução de mapa em segundo plano pode não ser crítica para a operação do sistema e nenhum alarme é necessário se uma tarefa subsequente for bem-sucedida.

#### Inacionável
  Um montante de erros em que uma pequena fração de usuários enfrenta de maneira transitória, mas graves, que desaparecem imediatamente por si mesmas. 
  Devido à alta velocidade de recuperação, este tipo de evento é resolvido antes que um operador possa responder, portanto, seria ineficaz emitir alertas para eles. 
  A frequência desses eventos inacionáveis deve, no entanto, ser verificada ocasionalmente para garantir que a degradação momentânea do desempenho permaneça isolada e insignificante.
  
#### Tarefas de automação
  Falhas sem impacto imediato no funcionamento do sistema, mas com potencial de ser um fator coadjuvante em outras falhas.

#### Indicadores iniciais
  Problemas aleatórios com impacto pequeno a moderado que afetam uma pequena fração dos usuários. 
  Eles vêm como resultado da saturação momentânea de recursos e pequenos bugs e podem ser indicadores precoces da incapacidade de um sistema de lidar com o alto fluxo de requisições.
  
#### Tarefas de otimização
  Problemas que não afetam a funcionalidade de um sistema, mas causam ineficiências na utilização de recursos que podem ser eliminadas pela reestruturação arquitetônica ou otimizações de software.
  
#### Não-problemas
  Anomalias sem impacto perceptível, observáveis como parte de qualquer operação do sistema, principalmente em escala.
  

É indicado que apenas nos quatro primeiros grupos seja aplicado alerta, pois somente nesses casos o sistema está realmente em risco e só então alguém deve intervir e alterar o estado do sistema.
Nestes quatro grupos de situações de alertas, podemos divídilos em dois:
  - O primeiro deve ter seu alerta disperado imediatamente, não sendo necessária a coleta de métricas por um período, exemplo se o sistema sai fora do ar ele deve desparar o alerta em seguida e não esperar por um tempo para então gerar o alerta.
  - O segundo é um tipo urgente mas que é interessante analisarmos os números por um período para então gerar o alerta, diferente da primeira opção.

### Tipos de Alertas

- E-mail
- SMS
- Ligação
- Notificação por aplicativo, como Slack ou Telegram.


