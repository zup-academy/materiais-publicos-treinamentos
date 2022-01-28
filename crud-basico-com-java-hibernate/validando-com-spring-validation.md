# Validações com Spring Validation

Uma das tarefas mais realizadas na construção de um software é aplicar validações nas informações que estão sendo transferidas e processadas, é importante que cada um destas informações estejam de acordo com uma restrição de valor ou formato. A forma mais simples de aplicar uma validação é utilizando uma estrutura de decisão o famoso SE (if), porém, com a evolução da unidade de codigo que trabalhamos, podemos deixar classes com dificil manutenção e alem de escrever um mesmo codigo repetidas vezes para objetos diferentes. Imagine que você precisa validar se o valor de um campo não é nulo, para cada classe você provavel mente definira um metodo com retorno booleano  que recebe uma informação, e no corpo deste metodo, estara um if que verifica se o valor informado é diferente de null. Veja um exemplo abaixo.

```java
    public class ImovelRequest {
    private Integer quantidadeQuartos;
    private Integer quantidadeBanheiros;
    private Integer areaConstruidaEmMetrosQuadrados;
    private Integer anoDeConstrucao;
    private Boolean possuiGaragem;

    public boolean naoNulo(Object valor){
        return valor!=null;
    }

    public boolean maiorQueZero(Integer valor){
        return valor>0;
    } 
}
```

Escrever metodos e validações genericos custa muito de tempo que poderia ser utilizado para construção de codigo que agregue mais valor a entrega. Uma possivel solução a este problema é utilizar bibliotecas e APIs especializadas neste contexto, e o hoje vamos falar sobre o Spring Validation e Bean Valitation. O Spring validation é um framework que disponibiliza uma serie de recursos para criação e disparo de validações, sejam elas vindas da Bean Validation ou custumizadas. Bean Validation é uma especificação que define um conjunto de regras e recomendações fromando um padrão para frameworks de validação em Java. O Hibernate Validator da RedHat é um dos frameworks que implementa esta especificaçã, e oferece funcionalidades para verificação da integridade dos dados, essas funcionalidades são disponibilizadas através de meta informação, ou seja, atráves de anotações. Agora que estamos contextualizados vamos refatorar nossa classe ImovelRequest para utilizar as Anotações da Bean Validation.

```java
    public class ImovelRequest {
    @NotNull 
    @Postive
    private Integer quantidadeQuartos;

    @NotNull 
    @Postive
    private Integer quantidadeBanheiros;

    @NotNull 
    @Postive
    @Min(40)
    private Integer areaConstruidaEmMetrosQuadrados;

    @NotNull 
    @Postive
    private Integer anoDeConstrucao;

    @NotNull
    private Boolean possuiGaragem;

}
```

## Agora vamos listar anotações que são amplamente utilizadas

- @NotBlank: verifica se um campo de texto, esta vazio.
- @Past: verifica se um campo de Data e DataHora esta no passado.
- @PastOrPresent: verifica se um campo de Data e DataHora esta no passado ou presente.
- @Future: verifica se um campo de Data e DataHora esta no futuro.
- @FutureOrPresente: verifica se um campo de Data e DataHora esta no futuro ou presente.
- @Min(valor): verifica se um campo numerico é no minimo o valor informado.
- @Max(valor): verifica se um campo numeirco é no maximo o valor informado.
- @Size(min= valor, max =valor): verifica se uma coleção tem no minimo e/ou maximo os valores informados.
- @Email: verifica se o campo tem um padrão de email bem formado.
- @CPF: verifica se o campo é um cpf valido.
- @NotNull: verifica se um campo não é nulo.
- @Pattern(valor): verifica se o campo corresponde a alguma expressão regular (regex)
- @Length(min =valor, max = valor): se a String informada tem no minimo e/ou maximo um valor.

Se interessou pelo assunto? consulte esta [documentação](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/) para saber mais sobre outras anotações e funcionalidades.