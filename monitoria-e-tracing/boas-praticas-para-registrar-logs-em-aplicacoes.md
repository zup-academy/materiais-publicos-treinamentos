#### Esse conteúdo foi baseado no vídeo do Alberto e nos conteúdos ele cita, que esta no seguinte Link :
https://www.youtube.com/watch?v=KEjteewlBJY&t=1702s

# Boas práticas para registrar logs em aplicações.

### Fique atento aos itens abaixo:

 - Escolha bem a biblioteca de implementação de logs, prefira as que utilizam a especificação.
 - É necessário que você sempre implemente o nível dos logs corretamente.Os níveis mais usados são: 
    FATAL : Algo muito errado aconteceu, preferível ser investigado imediatamente.
    ERROR : Algo errado aconteceu. Alguém deveria dar uma olhada.
    WARN : O processo pode continuar, mas tome cuidado extra.
    INFO : O processo de negócios importante mudou de estado/concluído. Devemos ser capazes de entender as mensagens INFO e descobrir rapidamente o que o aplicativo está fazendo.
    DEBUG : Notas de ajuda do desenvolvedor.
    TRACE : Informação muito detalhada, destinada apenas ao desenvolvimento. Geralmente, isso é para itens temporários.
 - Tenha objetos com o método toString(). Isso será útil para colocar dados importantes no log. 
 - A LGPD trouxe a tona a preocupação maior em não logar dados de PII ou PCI. É muito importante que nenhuma informação pessoal seja registrada. Para casos assim é utilizado mascaramento dos dados através de configurações que poderem ser realizadas.
 - Não registre senhas, chaves secretas, etc. As informações que não estão disponíveis para toda a equipe, não devem ser registradas.
 - Não devemos comprometer o fluxo da aplicação com logs.( Por isso vale muito usar bibliotecas/ frameworks de logs que já são construídos pensando nisso)
 - Coloquem mensagens que fazem sentido não só para você que é desenvolvedor e conhece o código.
 - É importante tratar o Stacktrace no log e utilizar ele de forma correta, existem muitas técnicas para logar o stacktrace limpo, sem ter que logar tudo de uma vez pois logar todo o stacktrace pode gerar um volume alto de dados.
 - Separar o que é informação técnica e o que é informação de negócio no log.
 - Se puder, estruture seu log em json
 - Manter uma estrutura de log com consistência entre as aplicações, ter um padrão para a aplicação de logs é fundamental.

## Qual a melhor forma?

De um alto nível, os melhores logs informam exatamente o que aconteceu, quando, onde e como. Esses logs são adequados para análises manuais, semiautomáticas e automatizadas. Idealmente, você pode analisá-los sem ter o aplicativo que os produziu – e definitivamente sem ter o desenvolvedor do aplicativo de plantão. Do ponto de vista do gerenciamento de logs, os logs podem ser centralizados para análise e retenção. Finalmente, eles não retardam o sistema e podem ser comprovados como confiáveis, se usados como evidência forense.

### O que logar

Criar uma lista abrangente de “o que registrar” para cada aplicativo e organização é impossível.
No entanto, a seguinte lista deve fornecer um ponto de partida útil para os seus aplicativos personalizados, especialmente aqueles que lidam com dados regulamentados, como cartões de pagamento ou informações pessoais confidenciais.

- Autenticação, autorização e eventos de acesso:
  - decisões de autenticação ou autorização bem-sucedidas e malsucedidas; 
  - acesso ao sistema, acesso a dados e acesso a componentes de aplicativos; 
  - acesso remoto, incluindo de um componente de aplicativo para outro em um ambiente distribuído.

- Mudanças:
  - alterações de sistema ou aplicativo (especialmente alterações de privilégios), 
  - alterações de dados (incluindo criação e destruição)
  - instalação e alterações de aplicativos e componentes.

- Problemas de disponibilidade:
  - inicialização e desligamento de sistemas, aplicativos e módulos ou componentes de aplicativos; 
  - falhas e erros, especialmente erros que afetam a disponibilidade do aplicativo; e 
  - sucessos e falhas de backup que afetam a disponibilidade. O quarto tipo são problemas de recursos:
  - recursos esgotados, capacidades excedidas e assim por diante; 
  - problemas e problemas de conectividade; e 
  - limites atingidos.

- Ameaças:
  - entradas inválidas e outros abusos de aplicativos prováveis  
  - outros problemas de segurança conhecidos por afetar o aplicativo.

### O que incluir 

Em seguida, quais dados você deve registrar para cada evento e em que nível de detalhe você deve registrá-los? A filosofia de detalhes de log relevantes remonta à Grécia antiga (http://en.wikipedia.org/wiki/Five_Ws) e se concentra nos “Seis Ws”:

• Quem estava envolvido?
• O que aconteceu?
• Onde isso aconteceu?
• Quando isso aconteceu?
• Por que isso aconteceu?
• Como isso aconteceu?

(Não, não podemos explicar por que o sexto W é na verdade um H.) Essa sabedoria antiga se aplica perfeitamente aos logs e ajuda a definir uma entrada de log útil e inequívoca. Com base nos seis Ws, a lista a seguir fornece um ponto de partida para o que incluir:
• O nome de usuário ajuda a responder “quem” para os eventos relevantes para as atividades do usuário ou administrador. Além disso, é útil incluir o nome do provedor de identidade ou domínio de segurança que atestou o nome de usuário, se essa informação estiver disponível.
• O objeto ajuda a responder “o quê” indicando o componente do sistema afetado ou outro objeto (como uma conta de usuário, recurso de dados ou arquivo).
• O status também ajuda a responder “o quê”, explicando se a ação direcionada ao objeto foi bem-sucedida ou falhou. (Outros tipos de status são possíveis, como “adiado”.)
• O sistema, aplicativo ou componente ajuda a responder “onde” e deve fornecer o contexto do aplicativo relevante, como o iniciador e os sistemas, aplicativos ou componentes de destino.
• A fonte ajuda a responder “de onde” para mensagens relacionadas à conectividade de rede ou operação de aplicativos distribuídos.
• O carimbo de hora e o fuso horário ajudam a responder “quando”. O fuso horário é essencial para aplicativos distribuídos. Além do carimbo de hora e fuso horário, alguns sistemas de alto volume usam um ID de transação.
• O motivo ajuda a responder “por quê”, para que a análise de log não exija muita pesquisa por um motivo oculto. Lembre-se, os clientes do log são a equipe de segurança e auditoria
• A ação ajuda a responder “como” fornecendo a natureza do evento.
• Além disso, a prioridade ajuda a indicar a importância do evento. No entanto, uma escala uniforme para classificar eventos por importância é impossível porque organizações diferentes terão prioridades diferentes. (Por exemplo, diferentes empresas podem ter políticas diferentes em relação à disponibilidade de informações versus confidencialidade.)




