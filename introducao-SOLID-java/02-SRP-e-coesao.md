# SRP - Single Responsibility Principle ou Princípio da Reponsabilidade Única

## O que é?

Como o próprio nome já diz, o Princípio da Reponsabilidade Única diz que cada módulo ou classe deve ter apenas uma
responsabilidade, ela deve representar apenas um conceito de negócio.
Robert "Uncle Bob" Martin
diz [neste artigo](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html) algo como: "A
classe precisa ter apenas um motivo para mudar". Por exemplo, se eu tenho uma classe que calcula imposto e por algum
motivo a maneira que eu calculo imposto precisa ser alterada, vou alterar ali naquela classe. Mas se a maneira de enviar
emails do meu sistema mudar, eu não vou mexer nessa mesma classe, pois são responsabilidades distintas, então essa parte
do sistema deve se encontrar em outro ponto.

Como esse princípio nos ajuda a criar um 'software' melhor? Vamos ver alguns dos seus benefícios:

- Teste — uma classe com uma responsabilidade terá muito menos casos de testes.
- Menor acoplamento — menos funcionalidades numa única classe terá menos dependências.
- Organização — classes menores e bem organizadas são mais fáceis de pesquisar do que as classes monolíticas.
- Coesão — classes coesas são mais fáceis de serem mantidas, reutilizadas e tendem a ter menos bugs (como diria Alberto
  Souza: "a única certeza que temos ao desenvolver um software é que ele terá bugs", mas nosso trabalho é fazer o
  melhor para evitá-los!)

## Coesão

Se procurarmos no dicionário o significado de coesão, encontraremos algo como: coesão é a conexão, ligação, harmonia
entre os elementos de um todo.

Trazendo para o mundo do 'software', esse todo seria nossa classe, por exemplo, e a coesão seria a harmonia dos
elementos dessa classe, sendo esses elementos os atributos e métodos. Os atributos e os métodos devem estar em harmonia,
devem estar unidos e representar coisas em comum.

Entendo que temos duas maneiras de observar a coesão do nosso código:

- **Maneira lógica**: uma classe 100% coesa é aquela que cada método utiliza todos os atributos da classe.
  Claro que dessa forma seria muito difícil encontrar coesão em classes de domínio (entidades, por exemplo), onde
  costumamos ter métodos que manipulam apenas um ou outro atributo.
  Porém, dessa maneira conseguimos ter uma forma universal de olhar para o código e ter ao menos um bom sinal de saber
  se temos ou não coesão.
- **Maneira subjetiva**: outra forma importante é entender a responsabilidade de cada classe (ou de um domínio como
  todo), e esse é o momento que a coesão e o SRP se juntam. Mas como definir responsabilidades únicas?
  _Uncle Bob_ afirma que "Um módulo deve ser responsável por um, e apenas um, ator". O termo ator pode ser um
  _stakeholder_ ou um usuário, e o módulo pode ser a classe ou o conjunto de classes que representam aquele tema. Logo
  como dito acima, uma classe deve ter apenas um motivo para mudar.

É sempre bom lembrar que mesmo se uma classe for logicamente 100% coesa (todos os métodos utilizarem todos os atributos),
ela pode não cumprir o SRP por ter mais de uma responsabilidade.
E uma classe que tem apenas uma responsabilidade pode não ser 100% coesa, pois não utiliza todos os atributos nos
métodos (por exemplo: uma entidade do hibernate).
O objetivo aqui é encontrar um equilíbrio de maneira que esses dois conceitos trabalhem juntos.

No livro **Orientação a Objetos e SOLID para Ninjas**, _Maurício Aniche_ diz:
"Coesão é, com certeza, uma das palavras mais conhecidas por programadores que usam linguagens orientadas a objeto. Seu
significado também é bastante conhecido: **uma classe coesa é aquela que possui uma única responsabilidade**".

Veja o código a seguir:

```java
public class Usuario {

    private String nome;
    private String email;
    private String rua;
    private String numero;
    private String bairro;
    private String cidade;
    private String estado;
    private String CEP;

    private void alterarRua() {
        // lógica alteração
    }

    private void formatarCEP() {
        // lógica formatação cep
    }

    private void consultarCEP() {
        // consulta serviço externo Correios
    }

}
```

Essa classe está coesa?
Logo de cara podemos afirmar que **não**!
Primeiro indício da falta de coesão é a quantidade de atributos da classe e a tendência a aumentar, por exemplo,
se precisarmos colocar agora o país e um complemento.
Mas mais importante que a quantidade é perceber que alguns desses atributos não dizem respeito ao Usuário em si, e sim
ao Endereço do mesmo. Claro que o Usuário tem um Endereço, mas está muito detalhado, todas as informações do Endereço
soltas na classe Usuário.
Será que a formatação visual do CEP deveria estar nessa classe? Será que a consulta do CEP a um serviço externo também
está no lugar correto?

Então para resolver isso e deixar as nossas classes mais coesas, vamos extrair o Endereço para uma classe
separada então teremos:

```java
public class Usuario {

    private String nome;
    private String email;
    private Endereco endereco;

    private void alteraEndereco(Endereco endereco) {
        this.endereco.altera(endereco);
    }
}

public class Endereco {

    private String rua;
    private String numero;
    private String bairro;
    private String cidade;
    private String estado;
    private String CEP;

    public void altera(Endereco endereco) {
        // lógica
    }
}
```

Note que o método _consultarCEP()_ também foi removido, pois, essa consulta deve ser feita numa classe separada apenas
para consultar CEP's, mantendo assim uma única responsabilidade.

[//]: # (inserir mais um exemplo, com classe com regras de negócios, de assuntos diferentes)




