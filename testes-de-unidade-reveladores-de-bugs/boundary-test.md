# Boundary Test

Ao estudar a técnica [Specification Test](https://github.com/zup-academy/materiais-publicos-treinamentos/blob/main/testes-de-unidade-reveladores-de-bugs/specification-test.md) aprendemos que quando  entradas diferentes para uma funcionalidade ou metodo geram o mesmo resultado, estas pertencem a uma partição ou classe.

Uma funcionalidade tende a possuir diversas classes, e a fronteira de uma classe com outra é dado por um limite que uma entrada pode atingir. Um otimo exemplo disto é o codigo civil que determina que uma pessoa tem a maioridade apartir dos 18 anos completos. Se traduzimos esta lei para um codigo, é muito provavel que encontremos o seguinte:

```java
public boolean isMaiorDeIdade(int idade){
    if(idade>=18){
        return true;
    }

    return false;
}
```

Olhando para o nosso codigo é facil identificar duas partições de entradas:

1. idades com maioridade, que são os valores que são igual ou maiores que de 18, como: 18, 19, 25, 34 e 76.
2. idades sem maioridade, que são os valores que por natureza são menores que 18, como:  1, 2, 3, 16 e 17.


Olhando para o codigo fica simples de identificar a fronteira entre as clases, que é o valor de 18, quando a idade atinge este valor, a entrada deixa de pertencer a partição de idades sem maioridade e passa a fazer parte da partição de idades com maioridade. 

Estamos falando disso porque é muito comum que os bugs de um sistemas se concentrem em entradas proximas aos limites entre as classes. E é para estas sistuações que utilizaremos Boundary Test para garantir que o software funcione corretamente nos limites das classes.


## Entendendo Boundary Test

Esta técnica visa encontrar os limites das classes de entradas, e elaborar testes que garantam o bom funcionamento do software. Para isso vamos nos basear em alguns conceitos, como: point, offpoint, inpoints e offpoints.

- Point: é o valor que determina a fronteira das classes, é o que aparece na condição, no exemplo da maioridade é exatamente o 18.

- OffPoint: é o valor mais proximo que invalida a condição. Ao levar para o exemplo da maioridade, é o 17. É importante observar que para condições de igualdade ou diferença, teremos offpoint para duas direções.

- offPoints: são todos os valores que invalidam a condição. 

- inpoints: são todos os valores que validam  a condição.



#### Criando testes para maioridade com JUnit e Boundary Test

o primeiro caso de testes que iremos fazer é o que toca no `point`, então iremos validar se de fato a pessao é de maior ou não.


```java
@DisplayName("deve ser de maior quando a idade seja 18")
@Test
void test(){
    boolean maiorDeIdade = maiorIdadeValidador.isMaiorIdade(18);

    assertTrue(maiorDeIdade, "com 18 anos já deveria ser de maior");
}
```

outro caso que iremos criar é quando toca no `offpoint`, então caso a idade seja 17 anos não deve ser maior de idade;

```java
@DisplayName("nao deve ser de maior quando a idade seja 17 anos")
@Test
void test1(){
    boolean maiorDeIdade = maiorIdadeValidador.isMaiorIdade(17);

    assertFalse(maiorDeIdade, "com 17 não deveria ser maior de idade");
}
```

para garantir que ambas as classes se comportam conforme o esperado, nos resta, testar os `in-points` e `out-points`. E se torna uma grande alternativa criar testes parametrizados.


então teremos os metodos: 

```java
@ParameterizedTest(name = "{index} => idade={0}")
@MethodSource("idadesComMaioridade")
@DisplayName("deve ser maior de idade")
void test2(int idade){
    boolean maiorDeIdade = maiorIdadeValidador.isMaiorIdade(idade);

    assertTrue(maiorDeIdade, format("com %d deveria ser maior de idade",idade));
}

private static Stream<Arguments> idadesComMaioridade() {
    return Stream.of(
            Arguments.of(19),
            Arguments.of(20),
            Arguments.of(36),
            Arguments.of(78),
            Arguments.of(99)
    );
}
```
```java
@ParameterizedTest(name = "{index} => idade={0}")
@MethodSource("idadesSemMaioridade")
@DisplayName("deve ser maior de idade")
void test3(int idade){
    boolean maiorDeIdade = maiorIdadeValidador.isMaiorIdade(idade);

    assertFalse(maiorDeIdade, format("com %d não deveria ser maior de idade",idade));
}

private static Stream<Arguments> idadesSemMaioridade() {
    return Stream.of(
            Arguments.of(17),
            Arguments.of(16),
            Arguments.of(5),
            Arguments.of(8),
            Arguments.of(2)
    );
}
```
