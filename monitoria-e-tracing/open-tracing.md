# O que é OpenTracing

Para facilitar a implementação de "Distributed tracing" ou "Tracing distribuido" como costumamos chamar no Brasil, podemos utilizar de ferramentas para nos auxiliar.
O OpenTracing é uma especificação que determina como devem ser implementadas ferramentas e bibliotecas para tracing distribuído.

A maioria dos modelos mentais para rastreamento vem do artigo Dapper do Google. OpenTracing usa substantivos e verbos semelhantes.

Através do Open Tracing são feitas especificações de conceitos e terminologias.

O Trace/rastreamento é feito através dos Spans, um fluxo de negócio no final das contas vai gerar um grafo de spans.

Trace: A descrição de uma transação conforme ela se move por um sistema distribuído.


Cada Span encapsula o seguinte estado:
Um nome de operação
Um carimbo de data/hora de início
Um carimbo de data/hora de término
Um conjunto de zero ou mais tags de extensão de chave:valor . As chaves devem ser strings. Os valores podem ser strings, bools ou tipos numéricos.
Um conjunto de zero ou mais Span Logs , cada um dos quais é um mapa de chave:valor emparelhado com um carimbo de data/hora. As chaves devem ser strings, embora os valores possam ser de qualquer tipo. Nem todas as implementações do OpenTracing devem oferecer suporte a todos os tipos de valor.
Um SpanContext (veja abaixo)
Referências a zero ou mais Spans causalmente relacionados ( por meio do SpanContext desses Spans relacionados )

Cada SpanContext encapsula o seguinte estado:

- Qualquer estado dependente de implementação do OpenTracing (por exemplo, IDs de rastreamento e extensão) necessário para se referir a um Span distinto em um limite de processo
- Itens de bagagem , que são apenas pares chave:valor que cruzam os limites do processo

Um Span pode fazer referência a zero ou mais outros SpanContexts que estejam relacionados causalmente.
Atualmente, o OpenTracing define dois tipos de referências: ChildOf e FollowsFrom.
Ambos os tipos de referência modelam especificamente as relações causais diretas entre um Span filho e um Span pai. 

