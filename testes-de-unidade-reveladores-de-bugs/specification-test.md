# Specification Test (Teste de Especificação)

Construir software é uma tarefa extremamente complexa, isso não se deve somente a complexidade de transformar a regra de negócio em um sistema, mas, também de testar este sistema e garantir que o mesmo funcione conforme o esperado.  

Uma tarefa bastante desafiadora é identificar quais são os comportamentos que determinada funcionalidade terá com entradas diferentes, isso porque dado ao valor de uma entrada, um fluxo alternativo pode ser tomado e levar a outro resultado.  Isto implica que devemos ter diversos cenários que contemplem toda a complexidade de sistema.


Uma estratégia que pode otimizar a escrita da `Suite de testes` (conjuto) é observar a especificação da funcionalidade e extrair as combinações de valor de entrada e regra de negocio da funcionalidade para poder identificar quais são as saidas, e se estas saidas  satisfaçam diversos cenários de execução da funcionalidade, esta estratégia é conhecida com Specification Test ou Testes de Especificação em português.

A aplicação dessa estrategias pode ser definida em 4 etapas:

1. Identificação dos parametros de entrada
2. Identificação das caractertisticas dos parametros de entrada
3. Adicionando restrições a partir das caracteristicas das entradas e diminuindo o conjunto de testes
4. Combinando valores de entrada  e obtendo casos de testes.

Parece bem abstrato, mas você já ira entender como utilizar esta técninca para criar testes a partir de uma especificação e até mesmo validar se a especificação corresponde ao problema.

Para melhor entendimento observe a seguinte especificação e vamos construir cenarios aplicando a técnica.


`Necessidade: dado o ID de um funcionario, caso ele tenha mais de um ano de adimissão deve acrescentar um aumento de 10% ao seu salario.` 


### entendendo Specification Test

O primeiro passo diz que devemos identificar os parametros de entrada, seja de um metodo ou classe. No nosso caso temos um id que corresponde a um funcionario.

1. identificador de funcionario;

Na segunda etapa iremos identificar as caracteristicas dos parâmetros de entrada, isto significa que iremos observar qual é seu tipo, se deve ser maior que zero ou se pode ser nulo.  Na maioria das vezes estas caracteristicas podem ser encontradas na propria especificação.

2. Caracteristicas: valor numerico inteiro entre 1 e N. Este valor se refere a um Funcionario, com uma dataDeAdmissão, que pode ser menor que um ano ou maior.

Iniciando a terceira etapa iremos construtir as restrições que iremos aplicar as nossas entradas. Para fazer isso iremos observar a entrada e as caracteristicas e estipular possibilidades. 

Como nossa entrada é um valor numerico inteiro, entre 1 e N, significa que a função não aceita valores menores que 1. E também não aceita valores nulos; Outra caracteristica é que este valor numerico se refere a um identificador de um funcionario que pode ter mais ou menos que um ano de admissão.


Por fim na quarta etapa iremos combinar as restrições com as caracteristicas e obter nossos casos de teste, como:

1. id menor que 1
2. id é nulo;
2. id positivo para funcionario com mais de um ano de adimissão
3. id positivo para funcionario com menos de um ano de adimissão


feito isso podemos criar nossos teste utilizando JUnit, primeiro observaremos a implementação da funcionalidade.

```java
   public class Funcionario{
       private Long id;
       private String nome;
       private BigDecimal salario;
       private LocalDate dataAdmissao;

    public Funcioario(Long id, String nome, BigDecimal salario, LocalDate dataAdmissao){
        this.id = id;
        this.nome = nome;
        this.salario = salario;
        this.dataAdmissao = dataAdmissao;
    }

    public Funcionario(){}

    //getters e setters omitidos
   } 
```


```java
    public class AdptarSalario{
        private Map<Long, Funcionario> funcionarios;

        public AdaptarSalario(Map<Long, Funcionario> funcionarios){
            this.funcionarios = funcionarios;
        }


        public void adapta(Long id){
            if(id == null){
                throws new IdInvalidoException("o id nao deve ser nulo");
            }
            
            if(id < 1L){
                throws new IdInvalidoException("o id deve ser maior igual a um");
            }

            Funcionario funcionario = funcionario.get(id);

            int anos = Period.beetween(
                funcionario.getDataAdmissao(), LocalDate.now())
                .getYears();

            if(anos >= 1){
                BigDecimal salario =  funcionario.getSalario()
                funcionario.setSalario(salario.multiply( new BigDecimal("1.1"));
            }

        }
    }
```

Sabemos que a classe `AdptarSalario` precisa de um mapa de funcionario por id, então para execução do nosso teste, iremos fornecer essa depêndencias.


```JAVA
public class AdptarSalarioTest{
    private AdptarSalario adptaSalario;
    private Map<Long, Funcioario> funcionarios;

    @BeforeEach
    public void setUp(){
        Funcionario rafael = new Funcioario(1,"Rafael Ponte",new BigDecimal("1000"), LocalDate.of(2017, 5, 26));
        Funcionario yuri = new Funcioario(1,"Yuri Matheus",new BigDecimal("1000"), LocalDate.now());
        this.funcionarios = Map.of(
           rafael.getId() , rafael
            yuri.getId(), yuri
        );

        this.adptaSalario= new AdptarSalario(funcionarios);
    }


      @Test
    @DisplayName("o id nao pode ser menor que um")
    void test() {
        IdInvalidoException idInvalidoException = assertThrows(IdInvalidoException.class, () -> {
            this.adptarSalario.adapta(0L);
        });

        assertEquals("o id deve ser maior igual a um", idInvalidoException.getMessage());
    }

    @Test
    @DisplayName("o id nao pode ser nulo")
    void test1() {
        IdInvalidoException idInvalidoException = assertThrows(IdInvalidoException.class, () -> {
            this.adptarSalario.adapta(null);
        });

        assertEquals("o id nao deve ser nulo", idInvalidoException.getMessage());
    }


    @Test
    @DisplayName("nao deve ter o salario reajustado")
    void test2() {
        Funcionario funcionario = funcionarios.get(2L);
        BigDecimal antesDaAdptacao = funcionario.getSalario();

        this.adptarSalario.adapta(funcionario.getId());
        assertEquals(antesDaAdptacao,funcionario.getSalario());
    }

    @Test
    @DisplayName("deve ter o salario reajustado")
    void test3() {
        Funcionario funcionario = funcionarios.get(1L);
        BigDecimal antesDaAdptacao = funcionario.getSalario();

        this.adptarSalario.adapta(funcionario.getId());
        assertEquals(antesDaAdptacao.multiply( new BigDecimal("1.1")),funcionario.getSalario());
    }
}
```
 Após sair da quarta etapa com os casos de testes estabelecidos podemos observar que para cada caso existem diversas entradas que proporcionam o mesmo resultado, isso quer dizer que podemos satisfazer o teste: `deve ter o salario reajustado` para qualquer funcionario com id maior que um e com data de admissão maior que um ano.  Isto significa que podemos olhar para entradas de maneira particionadas, ou seja, existem partições que agrupam diversos valores de entrada, que proporcionam o mesmo resultado. 

 Estes grupos chamamos de partição ou classes de equivalencia, e sempre que construimos cenarios de teste optamos por encontrar estas partições para reduzir a quantidade de teste que criamos. Um bom exemplo é que o teste: `o id nao pode ser menor que um`, poderia lançar a exceção de idInvalido para os valores 0, -1, -3, -273, porém ao testar com valor de entrada zero, assumimos que as demais entradas também satisfazem aquele caso, evitando escrever um teste informando cada um destes valores.


## Identificando partições de equivalencia e construindo Testes com Specification Test

Como dito anteriormente nosso ponto de partida é a especificação de um sistema ou funcionalidade, esta especificação determina quais são as entradas, regras do negócio e saídas, e a partir destas informações identificamos  quais entradas satisfazem alguma regra  ou condição, o que nos retorna uma classe de teste.  Abaixo se encontra uma especificação da funcionalidade de descontos da Mercearia pão com vinho.


```
Necessidades: Dado um produto e um dia da semana é necessário retornar o preço do produto com ou sem desconto.

Regras: 

* Um produto do tipo Verdura tem desconto na segunda-feira, de 10%;
* Um produto do tipo Carne tem desconto na terça-feira de 7%;
* Um produto do tipo Higiene pessoal tem desconto na quarta-feira de 5%;
* Um produto do tipo Limpeza de casa tem desconto na quinta-feira de 4%;
* Um produto do tipo Padaria tem desconto na sexta-feira de 6%;
```


Ao observar a especificação podemos imaginar alguns testes a se fazer como:

- Informar uma verdura na segunda-feira e obter o desconto;
- Informar uma verdura em qualquer outro dia da semana e não obter desconto
- Informar uma carne na terça-feira e obter o desconto;
- Informar uma carne em qualquer outro dia da semana e não obter desconto
- Informar um produto de higiene pessoal na quarta-feira e obter o desconto;
- Informar um produto de higiene pessoal em qualquer outro dia da semana e não obter desconto


Sem perceber aqui conseguimos extrair duas classes de entradas: 

- Um produto deve conter desconto caso seja comprado no dia em que sua categoria é apta.
- Um produto não deve conter desconto caso seja comprado no dia em que sua categoria não é apta.

Quando olhamos para as classes de entradas podemos notar que se testamos um produto para obter desconto no dia correto, independente do tipo ou dia, devemos obter um resultado que contemple a regra, então temos uma relação de equivalência, se testarmos a compra de um produto da padaria na sexta-feira ou uma carne na terça-feira nosso teste deve garantir que o preço seja devolvido com desconto.

E se testarmos um produto de higiene pessoal na segunda-feira ou uma verdura na sexta-feira não devemos obter desconto. Isto significa que antes escreveríamos no mínimo dois testes,  para cada tipo e dia da semana, um  para satisfazer a condição de desconto e outro para satisfazer. Utilizando a classe de equivalência vamos escrever uma quantidade menor de testes que satisfaçam estas condições.

Uma implementação da calculadora de descontos da mercearia é: 

```java
public enum Categoria {
    CARNE, HIGIENE_PESSOAL, LIMPEZA, VERDURA, PADARIA
}
```
```java
public class Produto {
    private String nome;
    private Categoria categoria;
    private BigDecimal preco;


    public Produto(String nome, Categoria categoria, BigDecimal preco) {
        this.nome = nome;
        this.categoria = categoria;
        this.preco = preco;
    }

   //getters omitidos
}
```
```java
public class CalculadoraDeDesconto {

    public BigDecimal desconto(DayOfWeek hoje, Produto produto) {

        if (hoje.equals(MONDAY) && produto.getCategoria().equals(VERDURA)) {
            return produto.getPreco().multiply(new BigDecimal("0.9"));
        }

        if (hoje.equals(TUESDAY) && produto.getCategoria().equals(CARNE)) {
            return produto.getPreco().multiply(new BigDecimal("0.93"));
        }

        if (hoje.equals(WEDNESDAY) && produto.getCategoria().equals(HIGIENE_PESSOAL)) {
            return produto.getPreco().multiply(new BigDecimal("0.95"));
        }

        if (hoje.equals(THURSDAY) && produto.getCategoria().equals(LIMPEZA)) {
            return produto.getPreco().multiply(new BigDecimal("0.96"));
        }

        if (hoje.equals(FRIDAY) && produto.getCategoria().equals(PADARIA)) {
            return produto.getPreco().multiply(new BigDecimal("0.94"));
        }

        return produto.getPreco();
    }
}
```

Ao utilizar JUnit e [ParameterizedTest](#) podemos construir nossos testes de maneira que satisfaçam todos os casos, utilizando partições de equivalência.

O primeiro passo é construtir nosso metodo de teste, que deve receber as informações de um produto, e o dia que sera executado o teste.

teremos um metodo com a seguinte assinatura:


```java

 @DisplayName("deve obter o desconto caso seja comprado no dia em que sua categoria é apta")
void test(String nome, Categoria categoria, BigDecimal preco, DayOfWeek dia, BigDecimal desconto) {

        
    }
```

Após modelar o metodo de teste para receber as informações, podemos construtir a logica de validação, que é dado um produto validar se o preço consta com desconto após a execução da calculadora. 

Começaremos instanciando o produto, informando o os argumentos de nome, preco, e categoria.Em seguida sera executada a calculadora de desconto, proximo passo é executar a calculadora de desconto, e atribuir em uma variavel o resultado. Por fim iremos validar se o resultado é equivalente ao preço com desconto aplicado.

```java
 @DisplayName("deve obter o desconto caso seja comprado no dia em que sua categoria é apta")
void test(String nome, Categoria categoria, BigDecimal preco, DayOfWeek dia, BigDecimal desconto) {
    Produto batataInglesa = new Produto(nome, categoria, preco);

    BigDecimal resultado = this.calculadora.desconto(dia, batataInglesa);

        assertEquals(
                preco.multiply(desconto),
                resultado,
                "O resultado deve corresponder ao preco aplicado com desconto"
        );
        
}
```

Com metodo construido iremos agora definir quais serão as entradas para cada argumento do metodo, então construiremos um provider, que é um metodo que retornar um conjunto de entradas. É importante que aqui seja construido entradas que validem que um produto de determinada categoria receba desconto no dia que sua categora é apta.


```java
 private static Stream<Arguments> descontoProvider() {
        return Stream.of(
                Arguments.of("Batata Inglesa", Categoria.VERDURA, new BigDecimal("10"), MONDAY, new BigDecimal("0.9")),
                Arguments.of("Alcatra Bovina", Categoria.CARNE, new BigDecimal("15"), TUESDAY, new BigDecimal("0.93")),
                Arguments.of("Colgate Total 12", Categoria.HIGIENE_PESSOAL, new BigDecimal("10"), WEDNESDAY, new BigDecimal("0.95")),
                Arguments.of("OMO MULTIAÇÃO", Categoria.LIMPEZA, new BigDecimal("10"), THURSDAY, new BigDecimal("0.96")),
                Arguments.of("Pão Frances", Categoria.PADARIA, new BigDecimal("10"), FRIDAY, new BigDecimal("0.94"))
        );
    }
```

Por fim basta configurar o nosso metodo para receber essas entradas, e atribuir cada um dos campos ao argumento correto no metodo teste, então conheceremos duas anotações: `@MethodSource` e `@ParameterizedTest`.

- `@MethodSource` é a anotação responsavel por definir qual provider atenderá o metodo de teste.

- `@ParameterizedTest` define um teste parametrizado, e também como os argumentos são informados ao metodo de teste.


Um `Arguments` recebe em cada indice um valor, no metodo  `descontoProvider` o primeiro argumento é responsavel por armazenar o nome do produto, então entendemos que o indice `0` é nome. 

Para finalizar a construção do nosso teste, iremos informar as anotações sobre sua assinatura, resultando em: 




```java
    @ParameterizedTest(
            name = "{index} => nome={0}, categoria={1}, preco={2}, dia={3}, desconto={4}"
    )
    @DisplayName("deve obter o desconto caso seja comprado no dia em que sua categoria é apta")
    @MethodSource("descontoProvider")
    void test(String nome, Categoria categoria, BigDecimal preco, DayOfWeek dia, BigDecimal desconto) {

        Produto batataInglesa = new Produto(nome, categoria, preco);

        BigDecimal resultado = this.calculadora.desconto(dia, batataInglesa);

        assertEquals(
                preco.multiply(desconto),
                resultado,
                "O resultado deve corresponder a 90% do preco"
        );
    }
```

E para finalizar construiremos os testes parametrizados para a classe de testes que não obtem desconto.

```java
    @ParameterizedTest(
            name = "{index} => nome={0}, categoria={1}, preco={2}, dia={3}, desconto={4}"
    )
    @DisplayName("nao deve obter o desconto caso seja comprado no dia em que sua categoria não é apta")
    @MethodSource("semDescontoProvider")
    void test2(String nome, Categoria categoria, BigDecimal preco, DayOfWeek dia, BigDecimal desconto) {

        Produto batataInglesa = new Produto(nome, categoria, preco);

        BigDecimal resultado = this.calculadora.desconto(dia, batataInglesa);

        assertEquals(
                preco,
                resultado,
                "O resultado deve corresponder ao preco do produto"
        );
    }

    private static Stream<Arguments> semDescontoProvider() {
        return Stream.of(
                Arguments.of("Batata Inglesa", Categoria.VERDURA, new BigDecimal("10"), THURSDAY, new BigDecimal("0.9")),
                Arguments.of("Alcatra Bovina", Categoria.CARNE, new BigDecimal("15"), WEDNESDAY, new BigDecimal("0.93")),
                Arguments.of("Colgate Total 12", Categoria.HIGIENE_PESSOAL, new BigDecimal("10"), FRIDAY, new BigDecimal("0.95")),
                Arguments.of("OMO MULTIAÇÃO", Categoria.LIMPEZA, new BigDecimal("10"), TUESDAY, new BigDecimal("0.96")),
                Arguments.of("Pão Frances", Categoria.PADARIA, new BigDecimal("10"), MONDAY, new BigDecimal("0.94"))
        );
    }

```
