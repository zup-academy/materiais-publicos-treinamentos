No decorrer da jornada de aprendizado do desenvolvedor é comum que o seu cinto de utilidades vá crescendo e contemple habilidades relacionadas aos mais diversos nichos. Quando olhamos para o nicho de Testes é comum que aplicamos testes de unidade, integração, performance, escalabilidade e outros.  Um ponto comum em todas as categorias é que existem técnicas que maximizam a qualidade da escrita destes testes, e indiretamente a qualidade do sistema.

Os testes de unidade tem baixo custo de execução e podem ser implementados em um tempo razoável, porém, se a construção destes não for guiada por boas técnicas, o resultado pode ser uma suíte que não contemple toda complexidade do sistema.

No decorrer deste treinamento vem sendo estudado técnicas baseadas em requisitos como  [Specification Test](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/testes-de-unidade-reveladores-de-bugs/specification-test.md) e [Boundary Test](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/testes-de-unidade-reveladores-de-bugs/boundary-test.md) que oferecem um guia para criação de casos de testes sem olhar para qualquer linha de código. E agora iremos conhecer técnicas que guiam a construção de testes utilizando o código como referência, conhecidas como Structural Test.

 
## `Structural Test`

Structural Test ou Testes Estruturais, são testes construídos guiados pelo **código-fonte**. Estes testes tem como responsabilidade garantir um **critério de cobertura**, este critério este intimamente relacionado as unidades do código que são executas a partir de um teste. 

Para entender melhor, o conceito de cobertura iremos, analisar o código da calculadora, e da função soma.

```java
public class Calculadora{
1.
2.	public int soma(int a, int b){
3.		return a+b;
4.	}
}
```
Um código simples, onde existe uma classe sem atributos, com um único método, que recebe dois números inteiros e retorna a soma destes, facilmente escreveríamos um teste de unidade para função soma. 

```java
@Test
void deveSomarDoisNumeros(){
	Calculadora  calc = new Calculadora();
	
    int resultado =  calc.soma(1,1);	

	assertEquals(
        2,
        resultado,
        "a soma de um mais um deveria corresponder a dois"
        );
}
```
Agora ao executar o nosso teste, quais métodos da classe Calculadora são executados?  Neste caso um, o método soma, então caso o critério de cobertura seja por método, obtemos 100% de cobertura para classe Calculadora.  

A cobertura é medida da seguinte forma, dividimos a quantidade de métodos executados da classe Calculadora a partir do nosso teste, pela quantidade de métodos que a classe possui, por fim multiplicamos por 100. 

Para nosso exemplo: 
```
Métodos executados pelo teste: soma.

Métodos existentes na classe: soma.

1/1 = 1 *100 = 100%
```

Após entender o que é uma cobertura ou Coverage em inglês, podemos conhecer quais são os critérios que serão estudados neste artigo.

- Line Coverage (and statement coverage) 
- Block coverage
- Branch/Decision coverage
- Condition (Basic and condition+branc) coverage
- Path coverage
- MC/DC coverage

### `Uma boa pergunta é qual a motivação criar testes a partir das técnicas de Structural Test?`  

As principais motivações estão relacionadas a como extrair  testes que satisfaçam o codigo criado e como identificar quantos testes fazer ou qual a hora de parar.

Ao utilizar as técnicas de especificação, primeiro extraimos as classes de entradas e criamos casos de testes a partir das mesmas. Nas técnicas de Testes Estruturais fazemos um processo analogo, onde, sistematicamante nos preocuparemos em extrair testes para cobertura de linha, blocos, condições ou fluxo do código, para cada criterio teremos testes diferentes.

Outro ponto importante é que ao deverivar testes utilizando qualquer criterio, nunca sabemos, quantos testes fazer, o que pode impactar em alto tempo criando testes, aumentando custo de tempo para entrega de uma feature.


## **`Line Coverage`** (Cobertura por Linhas)

Nesta técnica o objetivo é cobrir as linhas de codigo de determinada unidade, ao executar um teste. O indice de cobertura de linhas é medido de maneira similar a cobertura por metodos, dado a quantidade de linhas de uma classe ou metodo, quantas destas são exercitadas por um teste. 

```
cobertura = (linhasExecutadasPorUmTeste/totalDeLinhas) * 100
```

#### Vamos explorar esta técnica na pratica? 

Para entender melhor a crição de testes por cobertura de linhas, iremos explorar o exemplo classico da divisão de numeros. 

Dado a divisão entre dois numeros, onde exista um dividendo positivo, onde o divisor é zero, o resultado é uma tendencia ao infinto positiva. Caso o dividendo seja negativo, o resultado é uma tendência ao infinito negativo. Caso o dividendo seja maior ou menor que zero o resultado é a subtração sucessiva do divisor ao dividendo.

Uma implementação desta especificação é: 


```java
public class Calculadora {
1.
2.    public double dividir(double a, double b) {
3.        if (b == 0.0 && a > 0) {
4.           return Double.POSITIVE_INFINITY;
5.        }
6.
7.        if (b == 0.0 && a < 0) {
8.           return Double.NEGATIVE_INFINITY;
9.        }
10.
11.        return a / b;
12.    }
}
```

Dado ao codigo agora iremos criar testes que executem as linhas do nosso sistema, por exemplo: 

```java
@Test
public void deveDividirDoisNumeros(){
    Calculadora calc = new Calculadora();

    assertEquals(2,calc.dividir(4,2));
}
```

Se a gente fizer teste de mesa, ou executar nosso codigo, chegaremos na conclusão que ele toca nas linhas: 3, 7 e 11. Na linha 3 ele verifica se o 2 é igual a zero e se 4 é maior que zero, como a primeira condição não é satisfeita, ele segue o caminho e se testa na linha 7, onde nenhuma das condições são satisfeitas, logo a linha 11 é executada e temos como retorno o resultado da divisão.

Ao olhar para nossa classe, verificamos que as linhas: 4 e 8 não são exercitadas. então nosso indice de cobertura é: 

```
linhas totais de codigo: 5
linhas exercitadas por testes: 3

cobertura = 3/5 * 100
cobertura = 60%
```

Caso nosso criterio seja alcançar 100% de linhas cobertas por testes, ainda temos que extrair novos casos de teste do nosso codigo.

Então testaremos a divisão por zero, com dividendo positivo.

```java
@Test
@DisplayName("deve tender ao infinito positivo, caso um numero maior que zero seja divido por zero")
void test() {
    assertEquals(Double.POSITIVE_INFINITY, calc.dividir(4,0));
}
```
Agora ao executar nosso conjuto de testes, notamos que além das linhas 3, 7 e 11,  a linha 4 também é executada, então nosso coverage sobe de 60 para 80%. Por fim para atigir nosso criterio sabemos que teremos que construir mais testes, especificamente um que exercite a linha 8.

```java
@Test
@DisplayName("deve tender ao infinito negativo, caso um numero menor que zero seja divido por zero")
void test1() {
    assertEquals(Double.NEGATIVE_INFINITY, calc.dividir(-4,0));
}
```

Por fim conseguimos atingir nosso criterio, 100% de cobertura de linhas, e suite de testes completa. Por mais que a cobertura por linhas seja simples e direta devemos nos atentar com esta estratégia, pois ela é totalmente dependente do estilo do codigo que foi escrito, caso alguma mudança seja feita, a cobertura podera diminuir ou aumentar.

O proximo tipo criterio será a cobertura por bloco de codigo, ou Block converage.

## **`Block Coverage`** (Cobertura por blocos)

Nesta abordagem nosso objetivo é extrair testes visando a cobertura dos blocos de codigos do nosso programa. Estes blocos podem ser divididos em duas categorias: basicos e decisão.

### Bloco Basico

Os blocos basicos são aqueles onde existem diversas operacoes que são executadas de maneira continua. veja um exemplo no codigo abaixo: 

```java
public void somaDosQuadrados(int a, int b){ 
1.    int quadradoDeA = a*a;
2.    int quadradoDeB = b*b;
3.    int soma = quadradoDeA + quadradoDeB;
4.    System.out.println("Soma dos quadrados: "  + soma);
}
```

O codigo das linhas 1 a 4, são executados de maneira continua e sem bifurcação, estas são as caractertisticas  de um bloco basico.

### Bloco de Decisao

Os blocos de decisão como o nome sugere, são aqueles que a partir de determinada condição são criados dois caminhos, um otimo exemplo para testes blocos são as estruturaas de decisao `if` e `else`, também as estruturas de repetição como `while` e `for`.

Observe o bloco, na funcao de verificacao se um numero é primo, lembrando que um numero é primo caso ele seja divisivel apenas por 1 e ele mesmo.

```java
public boolean isPrimo(int numero){ 
01.    int foiDividido = 0;
02.
03.    for (int i = 1; i <= numero; i++) {
04.        if (numero % i == 0) {
05.            foiDividido++;
06.        }
07.    }
08.
09.    if (foiDividido == 2) {
10.        return true;
11.    }
12.    return false;
}
```

Esta funão é minimamente complexa, pois existem diversos blocos, como por exemplo: da linha 03 a 07 existe um bloco formado pela condição `i<=numero`, caso seja verdadeiro o for sera executado, caso contrario o programa seguira para linha 08.

na linha 09, existe outro bloco de decisão, formado pela condição `foiDividido == 2`, caso seja verdadeiro, sera executado a linha 10, caso contrario o programa ira seguir para linha 12;

Existe também um bloco de decisão na linha 04, onde esta a condição `numero % i == 0`, onde se caso o resultado seja verdadeiro, a linha 05 é executada, caso contrario o fluxo podera ser retornado para linha 04 ou 08.

A maneira de calcular a cobertura é parecido,  ao cobertura por linhas. Para obter o indice de blocos cobertos, dividimos a quantidade de blocos exercitados por testes, pela quantidade de blocos totais, e multiplicamos por 100.

```
cobertura por blocos = (quantidadeDeBlocosExecutadosPorTestes/quantidadeTotalDeBlocos) * 100
```

Para exercitar todos os blocos, devemos buscar atigir as condições que nos levam a cada bloco.
Então a seguinte suite sera suficiente para cobrir 100% dos blocos.

```java
    @Test
    @DisplayName("o numero zero não deve ser primo")
    void test() {
        NumeroPrimo numero = new NumeroPrimo();
        assertFalse(numero.isPrimo(0));
    }

    @Test
    @DisplayName("o numero 2 deve ser primo")
    void test1() {
        NumeroPrimo numero = new NumeroPrimo();
        assertTrue(numero.isPrimo(2));
    }

    @Test
    @DisplayName("o numero 15 nao deve ser primo")
    void test2() {
        NumeroPrimo numero = new NumeroPrimo();
        assertFalse(numero.isPrimo(15));
    }
```


## `Cobertura condicional`