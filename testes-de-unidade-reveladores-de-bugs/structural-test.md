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
As vezes garantir que apenas o blocos onde a condição atinge o valor de `true`, não pode ser suficiente, dado que não há garantias que o mesmo deve falhar quando resultados que não satisfaçam o valor `false` para condição. Nestas situações podemos incrementar a quantidade de blocos, buscando exercicitar o bloco da resultante da condição com valor false para cada decisão. É o que chamamos de `Cobertura Decisão (Filial)`.

## `Cobertura condicional ((Basic) condition coverage)`

As decisões geram sempre duas filiais ou ramificações, não importa o quão grande ou complexa pode ser a condição que esta sendo avaliada. Conforme a condição se torne mais complicada, os criterios de cobertura estudados anteriormente vão se tornando menos efetivos, isto significa que testar buscando a cobertura de filiais, não é mais suficiente.

Condições complexas aos serem testadas utilizando o criterio de cobertura filial não tem todas as decisões cobertas, dado que avaliamos o resultado da condição como um só. Então para que as demais decisões sejam cobertas precisamos de um criterio que vise exercitar todas condições para o resutlado `true` e pelo menos uma vez para `false`.

Para entender melhor observe o codigo, que visa identificar que dado 3 numeros, o primeiro é maior que o segundo, o segundo é maior que o terceiro, então o terceiro é menor que o primeiro. 

```java
public boolean crescente(int a, int b, int c){
    if(a > b && b > c && c < a){
        return true;
    }

    return false;
}
```
se informamos os valores para a,b e c, encontramos os seguintes resultados:
- a = 10, b = 8, c = 5 -> `true`;
- a = 10, b = 15, c = 5 -> `false`;

Dado os valores de entrada demostrados acima conseguimos validar os blocos que são gerados a partir da condição, porém, qualquer decisão que quebre uma sub-condição não foi coberta, por exemplo `(a = 20 , b = 30, c = 10)`. Desta forma não garantimos que todas as possibilidades de decisão estejam sendo exercitadas em nossa suite de testes.

A solução é fazer uma remodelagem em nosso grafico de blocos, agora para cada subcondição criaremos dois novos caminhos, onde iremos exercicitar de maneira individual. Isto significa que iremos criar testes que validam e invalidam as seguintes condicições:

- (a > b)
- (b > c)
- (c < a)

então, para realizar a cobertura teriamos que exercitar a condição `(a > b && b > c && c < a)` pelo menos uma vez para `verdadeiro` e outra para `falso`. O que equivaleria a 25% de cobertura dos blocos. Para alcancar os 75% restantes cobriremos as subcondições, pelo menos uma vez para `verdadeiro` e outra para `falso`. Teriamos uma suite de testes como:

``` java
class CrescenteTest {
    private Crescente numero;

    @BeforeEach
    void setUp() {
        this.numero = new Crescente();
    }

    @Test
    @DisplayName("a deve ser maior que b, e b maior que c, por fim c menor que a")
    void test() {
        assertTrue(this.numero.crescente(10, 8, 5));
    }


    @Test
    @DisplayName("a deve ser menor que b, e b maior que c, por fim c maior que a")
    void test1() {
        assertFalse(this.numero.crescente(10, 20, 15));
    }

    @Test
    @DisplayName("A deve ser maior que B")
    void test2() {
        assertTrue(this.numero.crescente(25, 20, 15));
    }

    @Test
    @DisplayName("A deve ser menor que B")
    void test3() {
        assertFalse(this.numero.crescente(15, 20, 15));
    }

    @Test
    @DisplayName("B deve ser maior que C")
    void test4() {
        assertTrue(this.numero.crescente(23, 22, 15));
    }

    @Test
    @DisplayName("b deve ser menor que C")
    void test5() {
        assertFalse(this.numero.crescente(15, 20, 25));
    }


    @Test
    @DisplayName("C deve ser menor que A")
    void test6() {
        assertTrue(this.numero.crescente(23, 22, 21));
    }

    @Test
    @DisplayName("C deve ser maior que A")
    void test7() {
        assertFalse(this.numero.crescente(21, 20, 25));
    }
}
```

Criar testes olhando apenas para cobertura de condição basica não é uma estratégia inteligente, dado que estamos analisando sempre as subcondições sem se importar com resultado final da condição, o que pode resultar em testes que sempre exercitam o caminho de sucesso ou falha. A solução é sempre trazer a cobertura filial junto, pois então cobriremos cada possivel tomada de decisão e os caminho que o software poderá percorrer, ao juntar as duas técnicas trabalhamos com o que chamamos de `Cobertura Condição + Decisão (Filial) -  (Condition + Branch coverage)`.

## Cobertura de Caminho `(Path Coverage)`

Ao utilizar o criterio de `Cobertura Condição + Decisão (Filial)`, estamos criando cenarios para cada `condição` e `branch` de maneira individual, isto proporciona alta quantidade de ramificações para construção de testes, ainda mais se comparamos ao criterio de cobertura por `linhas`. Embora cada condição esteja sendo exercitada para `true` e `false`, isso não garante que todos os caminhois que um programa pode ter estejam cobertos.

A cobertura de `caminho` visa criar testes apartir da combinação **completa** das condições de uma decisão. Cada combinação é um caminho que deve ser exercicitado por teste. 

A maneira de calcular uma cobertura por caminhos é simular as demais, devemos multiplicar por cem, o resultado da divisão dos caminhos percorridos pela quantidade total de caminhos. 

```
cobertura de caminhos = (caminhos percorridos/total de caminhos) * 100

caminhos percoridos= 2
total de camimhos = 6

cobertura de caminhos ->  (2/6) * 100 =  33,33%
```

### Testando através da cobertura de caminhos.

O primeiro passo para exercitar a cobertura de caminho, é encontrar todos os caminhos possivéis, uma maneira simples e rapida é construir a tabela verdade, para isto vamos precisar identificar as condições. Após ter as condições, criaremos as possibilidades. Observe o codigo abaixo.

```java
public boolean condition(int a, int b, int c){
    if(a > b && b < c || (b+c) < a){
        return true;
    }

    return false;
}
```

Agora iremos extrair as sub-codições:
- A = (a > b)
- B = (b < c)
- C = ((b+c) < a )

Com as condições podemos calcular o numero de casos de teste, com a seguinte formula:  `2 ^ (NumeroCondicoes)`. O proximo passo é criar a tabela verdade.


| `A` | `B` | `C` | `A AND B` | `(A AND B) OR C` |
|:---:|:---:|:---:|:---:|:---:|
|V|V|V|V|V|
|V|V|F|V|V|
|V|F|V|F|V|
|V|F|F|F|F|
|F|V|V|F|V|
|F|V|F|F|F|
|F|F|V|F|V|
|F|F|F|F|F|

Agora basta criar teste de exercite cada uma das possibilidades  da coluna `(A AND B) OR C`, que obteriamos 100% de cobertura de caminhos. 

É importante notar que a medida que as condições aumentarem o custo de fazer uma cobertura por caminho aumentaram exponencialmente, isto significa que não pode ser uma estrategia eficiente  dado alguma restrição de tempo para entrega.

## MC/DC - Condição Modificada/Cobertura de decisão `(Modified Condition/Decision Coverage)`

Testar todas as combinações de caminho de um software pode ser algo tão custoso a ponto de se tornar inviavel, dado que a cada condição o numero de testes ira crescer consideravelmente. Uma boa pergunta a se fazer é será que não existe um subconjuto de testes que garantiriam 100% da cobertura de caminho? A resposta é **`sim`**, existem algumas combinações mais importantes, onde dada uma condição ela consiga alternar o resultado de toda a decisão, e é nestas combinações que o `MC/DC` se foca.

A ideia principal é buscar testes onde, uma condição possa alterar o resultado de uma decisão de maneira independente das outras condições. Isto significa que todas as condições devem influenciar o resultado da decisão pelo menos uma vez. A influencia no resultado é dada quanto encontramos um **`Par de Independência`**, que é quando olhamos para  uma condição de decisão, e dada uma subcondição, encontramos um par de combinação onde, a penas a troca do falor de uma condição tem o poder de alterar o resultado.

Caso avaliemos o bloco de decisão dos caminhos da condição `A && (B || C)`, devemos encontrar pelo menos um Par de independencia para cada condição. 

### Como identificar um Par de Independencia? 

Primeiro deve-se escolher uma condição, por exemplo A. Dada esta condição precisamos observar o conjunto de testes da decisão, e buscar  casos de testes onde apenas o valor de A, seja capaz de alterar o valor da Decisão, isto significa, que se ao alterar o valor de A de `true` para `false` o valor final deve mudar. Então olhando apenas para o valor da condição A:  

- Deve conter um caso de teste onde o valor de A seja **`Verdadeiro`** e o valor da Decisão seja `Verdadeiro`, chamaremos de `caso 1`. Com por exemplo :

    | `Teste`  | `A` | B | C |  `Decision`: `A AND (B OR C)` |
    |:---:|:---:|:---:|:---:|:---:|
    |1|**`V`**|V|V|**V**|


- Deve conter um caso de teste onde o valor de A seja **`Falso`** e o valor da Decisão seja `Falso`, chamaremos de `caso 2`.
    | `Teste`  | `A` | B | C |  `Decision`: `A AND (B OR C)` |
    |:---:|:---:|:---:|:---:|:---:|
    |2|**`F`**|V|V|**F**|
- Quanto no `caso 1` tanto no `caso 2`, os valores de `B` e `C` devem ser iguais, isto nos dara o que chamamos de par de independencia.
    | `Teste`  | `A` | B | C |  `Decision`: `A AND (B OR C)` |
    |:---:|:---:|:---:|:---:|:---:|
    |1|V|**`V`**|**`V`**|**V**|
    |2|F|**`V`**|**`V`**|**F**|

Os pares de independência tem como responsabilidade auxiliar na escolha de casos de testes, de forma que sejam reduzidos de **`2^N`** para **`N+1`** que é o criterio medio de cobertura do **`MC/DC`**. 


### Como selecionar os casos de Testes que satisfaçam o **`MC/DC`**?

Selecionar os `Pares de Independência` para cada uma das subcondições. Adicionar um Par para cada condição, favoreca os pares das condições que conténham apenas um Par. Selecionar um teste que faça par com os casos já selecionados nas condições restantes. Por Exemplo: 

- A: {T1, T5}, {T2, T6}, {T3, T7}  
- B: {T2, T4}
- C: {T3, T4}

Como as condições `B e C` contém apenas um par cada, adicionaremos os testes que estão em seus pares, resultando nos casos: `T2, T3 e T4`. Até o momento existem 3 casos, o que corresponde ao **`N`**, então nosso papel é encontrar um  ou dois testes para completar nossa cobertura.

Ao observar os pares da condição `A` notamos que quanto o caso T3 e T2  estão presentes, então podemos escolher um caso que faça par com algum deles para completar nossa suite. Caso a gente não escolha acabaremos com 5 casos de testes, o que não é errado, porém, se temos a oportunidade de favorecer testes que já estão na suitem é o caminho mais eficiente. Então poderiamos ter as seguintes Suites.

- Suite 1: `T2, T3, T4 e T6`.
- Suite 2: `T2, T3, T4 e T7`.

## Aplicando o `MC/DC`

Como dito anteriormente, o `MC/DC` é traz de forma inteligente um guia para construção de testes que cobrem 100% dos caminhos do software. E Após conhecer os conceitos de `Pares de Independência` e como reduzir nossos casos, podemos sintetizar um passo a passo de como aplica-lo.

1. Identificar as subcondições da condição de decisão.
2. Construir todas as combinações de caminhos do software (Tabela Verdade).
3. Encontrar os Pares de Independencia para cada uma das subcondições.
4. Selecionar os casos de testes que forme uma suite com N + 1 testes.

Após ter ciência de cada uma das etapas necessárias, iremos construir os casos de testes para o metodo que permite que um aluno se matricule a uma universidade. Para que um aluno se matricule em um curso, é necessário que seja maior de idade, ou que o Pai assine sua declaração e page a taxa de matricula. Uma implementação esta disponivel abaixo.

```java
 public boolean aptoAMatricula(Pessoa pessoa){
	if((pessoa.paiAssina() && pessoa.matriculaPaga()) ||  pessoa.isMaiorDeIdade()){
		return true;
	}
	return false;
  }
```


### Etapa 01: Identificar as subcondições da condição de decisão.
Neste metodo existe três condições, que são representadas por:

- A: Pai assina
- B: taxa de matricula paga
- C: Maior de idade

Podemos rescrever a condição de decisão como: `(A && B) || C`. 

### Etapa 02: Construir todas as combinações de caminhos do software (Tabela Verdade)

Para construção da tabela verdade precisamos elevar a dois, o numero de subcondições da condição de decisão. Neste caso temos 3 sub-condições, o que resulta em 2^3 = 8 possibilidades.

Para construir a tabela iremos construtir uma matriz de 8 Linhas por N sub-condições em colunas. A primeira ira conter 4 valores verdadeiros consecutivos e 4 falsos, nas proximas colunas a quantidade de valores consecutivos sera dividido por 2. Isto signfica que na segunda coluna teremos 2 valores verdadeiros conseguitivos e 2 falsos, e a mesma regra se aplica a terceira coluna, resultando na seguinte matriz.


|`Teste`| `A` | `B` | `C` | `A AND B` | `(A AND B) OR C` |
|:---:|:---:|:---:|:---:|:---:|:---:|
|1|V|V|V|V|V|
|2|V|V|F|V|V|
|3|V|F|V|F|V|
|4|V|F|F|F|F|
|5|F|V|V|F|V|
|6|F|V|F|F|F|
|7|F|F|V|F|V|
|8|F|F|F|F|F|


### Etapa 03: Encontrar os Pares de Independencia para cada uma das subcondições.

Nesta etapa iremos encontrar os pares de independencia para cada uma das condições, comecando pela condição A.

#### Pares de idependencia condição A.


Nesta etapa iremos procurar para cada caso de teste, um teste onde invertemos o valor da condição A e mantemos os valores das condições B e C, e temos o resultado da decisão invertida. 

Olhando para condição A encontramos o par de 2 e 6.

|`Teste`| `A` | `B` | `C` | `A AND B` | `(A AND B) OR C` |
|:---:|:---:|:---:|:---:|:---:|:---:|
|1|V|V|V|V|V|
|`2`|`V`|V|F|V|`V`|
|3|V|F|V|F|V|
|4|V|F|F|F|F|
|5|F|V|V|F|V|
|`6`|`F`|V|F|F|`F`|
|7|F|F|V|F|V|
|8|F|F|F|F|F|

#### Pares de idependencia condição B.

Nesta etapa iremos procurar para cada caso de teste, um teste onde invertemos o valor da condição B e mantemos os valores das condições A e C, e temos o resultado da decisão invertida. 

Olhando para condição B, encontramos o par de 2 e 4.

|`Teste`| `A` | `B` | `C` | `A AND B` | `(A AND B) OR C` |
|:---:|:---:|:---:|:---:|:---:|:---:|
|1|V|V|V|V|V|
|`2`|V|`V`|F|V|`V`|
|3|V|F|V|F|V|
|`4`|V|`F`|F|F|`F`|
|5|F|V|V|F|V|
|6|F|V|F|F|F|
|7|F|F|V|F|V|
|8|F|F|F|F|F|

#### Pares de idependencia condição C.

Nesta etapa iremos procurar para cada caso de teste, um teste onde invertemos o valor da condição C e mantemos os valores das condições A e B, e temos o resultado da decisão invertida. 

Olhando para condição C, encontramos os seguintes pares: {3,4}, {5,6} e {7,8}.

|`Teste`| `A` | `B` | `C` | `A AND B` | `(A AND B) OR C` |
|:---:|:---:|:---:|:---:|:---:|:---:|
|1|V|V|V|V|V|
|2|V|V|F|V|V|
|`3`|V|F|`V`|F|`V`|
|`4`|V|F|`F`|F|`F`|
|`5`|F|V|`V`|F|`V`|
|`6`|F|V|`F`|F|`F`|
|`7`|F|F|`V`|F|`V`|
|`8`|F|F|`F`|F|`F`|

### Etapa 04: Selecionar os casos de testes que forme uma suite com N + 1 testes.

Após ter disponivel os pares de independencia para cada condição, iremos selecionar os testes que satisfaçam a cobertura do `MC/DC`. Lembrando que as condições que possuem um unico par devem ser favorecidas.

- Condição A: {2,6}
- Condição B: {2,4}
- Condição C: {3,4}, {5,6} e {7,8}

Como dito anteriormente é importante que exista pelo menos um par de independencia para cada condição, dado isso favorecemos os pares das condições que possuem apenas um par. 

O caso de teste 2, se repete em A e B, portanto temos o primeiro caso da nossa suite. O caso 4 e 6 também farão parte do conjunto de testes, então a tarefa é escolher um caso de teste entre os pares da condição C para completar a suite.

Os casos 4 e 6 já fazem parte do conjunto, o que é um bom indicativo para escolhar de um par da condição C, logo podemos fazer duas suites que contemplem `N + 1` casos, como:

- Suite 1: 2, 4, 6 e 3
- Suite 2: 2, 4, 6 e 5