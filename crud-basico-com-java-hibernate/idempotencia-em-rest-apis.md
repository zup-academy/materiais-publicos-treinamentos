# 3 estratégias para implementar Idempotência em REST APIs

Em arquiteturas distribuidas, é comum que a maior parte das transações de negocio estejam fragmentas em microsserviços disponiveis na rede. E bem como já ouvimos falar nas [8 Falacias dos sistemas distribuidos](https://dev.to/hugaomarques/sistemas-distribuidos-sao-estranhos-5bkp), a rede não é segura, e falhas podem acontecer a qualquer momento, então é importante que nós desenvolvedores projetamos nossas APIs para lidar com este tipo de eventualidade.
 
Uma falha de rede, pode proporcionar processamentos desnecessários, e dependendo da criticidade do projeto, é possível que ate prejuizos sejam adquiridos, como por exemplo, em um marketplace o processo de venda é composto por varias etapas, iniciando na busca e escolha do produto, e finalizando no pagamento. Porém, o que acontece, caso o servidor processe o pagamento e por motivo caia, e não retorne resposta ao cliente. Caso o cliente esteja programando para fazer retentativas, um novo pagamento pode ser gerado para a venda, mesmo que não tenha necessidade. Para evitar este tipo de erro, é importante que a operação de pagamento seja, idempotente, ou seja dado que realizado uma vez, sempre que chamada para um conjunto de argumentos, deverá retornar a mesma resposta. 

## Como escrevo REST APIs Idempotentes?

Antes de adentrarmos nos principios técnicos, devemos entender os fundamentos, e compreender os conceitos. O protocolo HTTP definiu na [RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231#section-4.2.2) que os verbos/metodos possuem propriedades em comum, e um metodo pode ter mais de uma propriedade, fazendo com que os metodos sejam **Seguros**, **Idempotentes** ou **Cacheaveis**.

Os metodos **seguros**, são aqueles que não causam efeitos colaterais no servidor, ou seja, não proporcionam mudança de estado no servidor. Portanto, os metodos seguros são operações que executam leituras no servidos.

Já os metodos **idempotentes**, são aqueles onde a operação executada tem o mesmo efeito colateral no estado do servidor, idempendente se foi executada uma ou várias vezes. Não se confunda, isto não signigica que a operação receba a mesma resposta, ou status HTTP.

Por fim, não menos importante, os metodos **cacheaveis** são aqueles que suas respostas podem ser armazenadas para reutilização em operações futuras. Na maior parte das vezes as operações seguras, podem ser cacheadas, caso não precisem de autorização no servidor.

### Quais metodos são HTTP Idempotentes?

Os verbos HTTP _GET, HEAD, PUT, DELETE, OPTIONS E TRACE_ são por natureza idempotentes, em geral, perceba que as operações de leitura não oferecem mudança de estado no servidor, então operações para o verbo _GET_, não necessitam de cuidado em especial para garantir a propriedade de idempotencia. Já os metodos _HEAD_, _OPTIONS_ e _TRACE_ se encaixam nas mesmas caracteriticas. O verbo _PUT_ por natureza, sempre faz uma sobreescrita do recurso, portanto, o efeito colateral é sempre a substituição completa do estado, garantindo assim a propriedade de idempotencia.

#### Mas será que não devo considerar um Design onde os verbos POST e PATCH sejam Idempontes?

A resposta é sim, vão existir casos como o do pagamento em market place, onde uma requisição que está criando um novo recurso, deverá ser idempotente, e para estes casos, é necessário estratégias afim de garantir que um pagamento já realizado, não seja, feito novamente. E estas estratégias são: Chave Condicional (Conditional Key), Chave Secundaria (Secondary Key) e Chave de Idempotencia (Idempotency Key).

* [Chave Condicional](https://opensource.zalando.com/restful-api-guidelines/#182)

   Nesta estratégia iremos atribuir na requisição uma chave especifica por recurso através do Header HTTP [`If-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Match). Esta chave em geral é um meta informação do recurso que permita identificar a existencia do mesmo, pode ser implementada através de um _hash_ ou um _número de versão_. Esta chave é geralmente armazenada com recurso, e sua inserção permite que o sistema identifique criações e atualizações simultâneas garantindo assim a propriedade de idempotencia. Afim de implementar um controle mais fino, pode ser considerado utilizar como suporte os Headers [ETag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag) e [If-None-Match](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match).

* [Chave Secundaria](https://opensource.zalando.com/restful-api-guidelines/#231)

   Nesta estratégia, a chave é inserida no corpo (body) da solicitação HTTP _POST_. E deverá ser armazenada permanentemente com recurso, afim de garantir que não haverão criações simultaneas para o recurso. A chave secundaria pode ser uma propriedade exclusiva do recurso ou a combinação de mais de uma. Desta forma no processo de criação do recurso, a chave de exclusividade deverá consultada no inicio do processo de criação, caso exista um recurso atrelado a mesma, as demais solicitações são revertidas, e a resposta HTTP deveram contér um Status 409 Conflict. Outra opção é retornar o Status 204 No Content, e um corpo (body) vazio.

* [Chave de Idempotencia](https://opensource.zalando.com/restful-api-guidelines/#230)

   Nesta estratégia, o cliente fornece uma chave através da criação de um Header HTTP nomeado como `Idempotency-Key`. Que é utilizado para garantir o comportamento de idempotência em situações de criação ou atualização de recursos. A idempotency key deve ser armazenada em um cache de chaves, onde o valor deverá ser a reposta HTTP da primeira requisição, para que nas proximas solicitações a mesma resposta seja retornada. Ao contrátrio das estratétigias de  _**chave condicional**_ e _**chave secundaria**_, que visam garantir que atualizações simultaneas não conflitem, a _**chave de idempotencia**_ garante que a mesma reposta será retornada.

   As chaves de idempotencia devem ser armazenadas temporariamente em cache, junto com a resposta HTTP, independente se a solicitação tiver sucesso ou falha. O servidor no inicio da solicitação, deverá consultar a chave em cache no inicio do processo, caso a chave exista, a mesma resposta da primeira solicitação será retornada. Em caso contrário, a solicitação será executada, e a resposta será armazenada no cache de chaves. 

**OBS**: Para garantir que o comportamento idempotente, o recurso e o cache de chaves devem estar sincronizados, considerando as armadilhas e falhas referente a sistemas distribuídos.

## Implentando as Estratégias de Idempotencia em Aplicações Spring Boot

Para facilitar o entendimento utilizaremos o exemplo descrito no inicio do artigo, onde é aberto uma solicitação para criar um recurso de  pagamento.

### Conditional Key

Como a chave condicional deverá ser informada via Header, e também deverá ser armazenada junto ao recurso de pagamento, é interessante que ela carrege um identificador exclusivo, e aqui temos a oportunidade de utilizar o id do pedido que esta no carrinho.

```JAVA
@RestController
public class ProcessaPagamentoController{
   @Autowired
   private ProcessaPagamentoService processaPagamentoService;
   
   @PostMapping("/api/v1/pagamentos")
   public ResponseEntity<?> processarPagamento(@RequestHeader("If-Match") UUID idPedido, @RequestBody @Valid DadosPagamentoRequest dadosPagamentoRequest, UriComponentsBuilder uriBuilder){
           
           boolean exists = processaPagamentoService.existePagamentoParaOPedido(idPedido);
           if(exists){
                throw new ResponseStatusException("Já existe um pagamento para este pedido", PRECONDITION_FAILED);
           }
           
           DadosPagamento dadosPagamento = request.toModel();
           Pagamento pagamento = processaPagamentoService.processarPagamento(idPedido, dadosPagamento);
           
           URI location = uriBuilder.path("/api/v1/pagamentos").buildAndExpand(pagamento.getId()).toUri();
           return ResponseEntity.created(location).build();        
   }
}
```

No código apresentado acima, cada pagamento tem como chave de condição o id do pedido, desta forma, uma vez processado a criação de pagamento para o pedido informado, as demais solicitações para este pedido, irão verificar se já não existe um pagamento, caso exista, a transação é revertida por quebrar a pre-condição de pagamento unico por pedido. E portanto a recomendação é que a resposta HTTP contenha o status de 412 Precondition falied.

### Secundary Key

Como dito anteriormente, a estátegia de chave secundaria foca em utilizar propriedades únicas do recurso como um chave de exclusividade, portanto a chave será informada no payload da requisição.

```JAVA
@RestController
public class ProcessaPagamentoController{
   @Autowired
   private ProcessaPagamentoService processaPagamentoService;
   
   @PostMapping("/api/v1/pagamentos")
   public ResponseEntity<?> processarPagamento(@RequestBody @Valid DadosPagamentoRequest dadosPagamentoRequest, UriComponentsBuilder uriBuilder){
           
           boolean exists = processaPagamentoService.existePagamentoParaOPedido(dadosPagamentoRequest.getIdPedido());
           
           if(exists){
                throw new ResponseStatusException("Já existe um pagamento para este pedido", CONFLICT);
           }
           
           DadosPagamento dadosPagamento = request.toModel();
           Pagamento pagamento = processaPagamentoService.processarPagamento(idPedido, dadosPagamento);
           
           URI location = uriBuilder.path("/api/v1/pagamentos").buildAndExpand(pagamento.getId()).toUri();
           return ResponseEntity.created(location).build();        
   }
}
```

O código apresentado acima é bem similar a estratégia [Conditional Key](#conditional-key), o que difere entre eles, é que a estratégia de chave secundaria, instrui que todas as respostas de falha por chave, deveram retornar o Status HTTP 409 Conflict, ou Status 204 No Content, caso a resposta não possua corpo (body).

A verdade é que nem sempre é possível obter uma chave de negocio exclusiva, e para estes casos é necessário combinar informações do payload para obter garantia de unicidade, uma boa alternativa é o fazer um `Hash` que combine caracteristicas da requisição.

### Idempotency Key

Aqui é onde as estratégias mais se diferem, enquanto a [chave de condição](#conditional-key) e [chave secundaria](#secondary-key) focam em garantir idempotencia evitando criações e atualizações simultaneas, a Idempotency Key trabalha garantindo que uma vez que uma resposta foi gerada para uma chave, independente se a reposta foi de falha ou sucesso, a mesma deverá ser retornada para todas as demais solicitações. Esta estratégia poderá ser um pouco mais custosa, já que exige que as respostas sejam armazenadas temporariamente, e dado a isso é necessário que seja implementado uma politica de desepejo das chaves, ou uma instalação de um provedor da caching, e para ambientes clusterizados se justifica um provedor distribuido.

```JAVA
@RestController
public class ProcessaPagamentoController{
   @Autowired
   private ProcessaPagamentoService processaPagamentoService;
   @Autowired
   private RedisCachingService cache;
   
   @PostMapping("/api/v1/pagamentos")
   public ResponseEntity<?> processarPagamento(@RequestHeader("Idempontecy-Key") UUID idempotencyKey, @RequestBody @Valid DadosPagamentoRequest dadosPagamentoRequest, UriComponentsBuilder uriBuilder){
           
           ResponseEntity response = cache.get(idempotencyKey);
           
           if(response != null){
                return response;
           }
           
           DadosPagamento dadosPagamento = request.toModel();
           Pagamento pagamento = processaPagamentoService.processarPagamento(dadosPagamento);
           
           URI location = uriBuilder.path("/api/v1/pagamentos").buildAndExpand(pagamento.getId()).toUri();
           ResponseEntity  response = ResponseEntity.created(location).build();
           cache.put(idempotencyKey, response, Duration.ofHour(1));
           
           return response;     
   }
}
```

O código apresentado acima pode ser resumido da seguinte forma, primeiro é consultado em cache se já existe uma resposta para esta chave de idempotencia, caso exista, é retornada a resposta para o cliente, caso contrario o pagamento é processado e armazenado em cache. Para que de fato a semantica da idempotencia seja garantida, é necessário que o caching esteja sincronizado com estado do servidor. E que o servidor esteja preparado para armadilhas comuns de sistemas distribuidos.


**OBS:** Independente da estratégia escolhida apenas a inserção de uma chave, não é suficiente para garantir que criações e atualizações simultaneas não sejam feitas, pois este codigo é candidato a sofrer com problemas de race conditions. Para enxergar os problemas e aprender a implementar soluções [assista a esta talk](https://www.youtube.com/live/80I5zv1sDHo?feature=share&t=790).

## Referências

- [Manual de Design de REST APIs da Zalando](https://opensource.zalando.com/restful-api-guidelines/#_zalando_restful_api_and_event_guidelines)
- [Implementando Requisições Idempotentes com Time da Stripe](https://stripe.com/docs/api/idempotent_requests)
