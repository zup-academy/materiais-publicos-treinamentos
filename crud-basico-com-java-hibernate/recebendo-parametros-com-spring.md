# Como receber parâmetros com Spring MVC

Quando estamos criando um serviço para web precisamos nos atentar na modelagem de cada endpoint, a escolha do método e de como as informações serão recebidas devem estar alinhadas com a especificação do HTTP e também devem atender as necessidade do sistema.

Serviços que atendem chamadas HTTP  com verbo GET não recebem informações no corpo da requisição, porém, recebem informações de consulta no proprio endereço do serviço, estas informações para consultas são chamados de query params.

Já serviços que atendem as requisições do verbo POST HTTP, podem receber tanto os query params quanto informações no corpo da requisição. Existe outra forma de se receber informações em uma chamada HTTP que é universal e deve ser usada sempre que desejamos realizar uma operação em um recurso especifico, que são as variaveis do caminho (path variable).

Neste guia você verá diversos metodos para recebimento de informações utilizando Spring MVC. 

## Recebendo Path Variable

Para receber variaveis no caminho utilizando Spring devemos nos atentar a duas premissas, primeiro o mapeamento do path e segundo anotação da variavel que recebera o valor.

Observe abaixo os detalhes em um metodo de cadastro de um controllador.

```java
    @GetMapping("/imoveis/{id}")
    public ResponseEntity<DetalharImovelResponse> consultar(@PathVariable Long id){
        Imovel imovel = repository.findById(id)
            .orElseThrow(()-> new ResponseStatusException(HttpStatus.NOT_FOUND,"Imovel não cadastrado no sistema."));


        return ResponseEntity.ok(new DetalharImovelResponse(imovel));

    }
```
Quando lidamos com variaveis no caminho temos que adicionar uma chave abrindo, o nome da variavel  e outra fechando, isto indicará ao Spring que este valor é variavel. O fim do mapeamento se da ao definir na entrada do metodo uma variavel com nome identico ao destinado ao path, previamente anotado com @PathVariable. Caso não esteja identico podemos utilizar a propriedade value da anotação @PathVariavle para definir, por exemplo: @PathVariable("id") Long imovelId.


## Recebendo Query Params

O recebimento de valores como consulta não é muito diferente, pode ser definido atráves de anotação, caso seja obrigatorio, ou caso seja mais de um e precise aplicar algum tipo de validação pode ser recebido através de composição.

### Recebendo através de anotação

```java
    @GetMapping("/imoveis")
    public ResponseEntity<?> consultar(@RequestParam(required=true, defaultValue="") String descricao){
        if(!descricao.isBlank()){
            Imovel imovel = repository.findByDescricao(id)
                                            .orElseThrow(()-> new ResponseStatusException(HttpStatus.NOT_FOUND,"Imovel não cadastrado no sistema."));

            return ResponseEntity.ok(new DetalharImovelResponse(imovel));
        }

        List<DetalharImovelResponse> imoveis = repository.findAll().stream()
                                                                        .map(DetalharImovelResponse::new)
                                                                        .collect(toList());



        return ResponseEntity.ok(imoveis);

    }
```

Anotação @RequestParam as seguintes propriedades:
- required: campo de valor booleano para definir se o valor é obrigatorio ou não.
- defaultValue: campo para definir um valor para caso o campo seja obrigatorio e seja preenchido vazio.
- name: campo para definir o nome do parâmetro que sera buscado na requisição.
- value: faz referência ao campo campo name()

### Recebendo parâmetros com validação

Neste caso iremos criar uma classe para representar nossa entrada.

```java
    public class ImovelParam{
        @Postive
        @Max(6)
        private Integer quantidadeQuarto;
        
        @Positive 
        @Min(30)
        private Integer areaEmMetrosQuadrados;

        //getters omitidos aqui
    }
```

Agora em iremos definir a entrada em nosso controllador da seguinte forma:

```java
     @GetMapping("/imoveis")
    public ResponseEntity<?> consultar(@Valid ImovelParam entradas){
        
        if(Objects.nonNull(entradas)){
                Imovel imovel = repository.findByAreaEmMetrosQuadradosAndQuantidadeQuartos(entradas.getAreaEmMetrosQuadrados(),entradas.getQuantidadeQuartos())
                                                                            .orElseThrow(()-> new ResponseStatusException(HttpStatus.NOT_FOUND,"Imovel não cadastrado no sistema."));


        return ResponseEntity.ok(new DetalharImovelResponse(imovel));

        }
        
        List<DetalharImovelResponse> imoveis = repository.findAll().stream()
                                                                        .map(DetalharImovelResponse::new)
                                                                        .collect(toList());



        return ResponseEntity.ok(imoveis);

    }
```

