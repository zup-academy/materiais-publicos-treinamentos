# OCP - Open-Closed Principle ou Princípio do Aberto-Fechado

## O que é?

O Princípio do Aberto-Fechado diz que as classes devem ser abertas para extensão e fechadas para modificação. Isso
significa que é possível adicionar novos comportamentos a uma classe sem modificá-la, mas sim criando novas classes que
estendem ou implementam essa classe original.

Segundo Bertrand Meyer, criador do conceito, "Um objeto ou componente deve ser capaz de adicionar novos comportamentos
sem ser modificado". Isso significa que devemos evitar a necessidade de mudar o código existente para adicionar novos
recursos.

## Benefícios

- Flexibilidade: ao seguir esse princípio, é possível adicionar novos comportamentos sem precisar modificar o código
  existente, tornando o sistema mais flexível.
- Manutenabilidade: ao evitar modificações no código existente, o sistema se torna mais fácil de ser mantido e evita
  erros.
- Reutilização: ao seguir esse princípio, é possível reutilizar classes existentes sem precisar modificá-las, o que
  aumenta a eficiência do código.

[//]: # (Os exemplos precisam ser melhor trabalhados)
## Exemplos de códigos:

## Utilizando Interface

### Exemplo 1: Adicionando novos tipos de pagamento

#### Antes:

```java
class CarrinhoDeCompras {
    public void finalizarCompra(double valor) {
        // Lógica para finalizar compra
    }
}
``` 

Neste exemplo, temos uma classe `CarrinhoDeCompras` que tem um método para finalizar compras. Se, no futuro, precisarmos
adicionar novos tipos de pagamento (por exemplo, pagamento via cartão de crédito ou pagamento via boleto), precisaríamos
modificar essa classe, o que violaria o princípio do Aberto-Fechado.

#### Depois:

```java
interface Pagamento {
    void pagar(double valor);
}
```

```java
class PagamentoBoleto implements Pagamento {
    public void pagar(double valor) {
        // Lógica para pagar via boleto
    }
}
```

```java
class PagamentoCartaoCredito implements Pagamento {
    public void pagar(double valor) {
        // Lógica para pagar via cartão de crédito
    }
}
```

```java
class CarrinhoDeCompras {
    private Pagamento pagamento;

    public CarrinhoDeCompras (Pagamento pagamento) {
        this.pagamento = pagamento;
    }

    public void finalizarCompra(double valor) {
        pagamento.pagar(valor);
    }
}
```

Neste exemplo, criamos uma interface `Pagamento` que define um método para pagar, e duas classes `PagamentoBoleto` e
`PagamentoCartaoCredito` que implementam essa interface. A classe `CarrinhoDeCompras` tem uma propriedade pagamento do tipo
`Pagamento` e um método para setar o pagamento. Dessa forma, podemos adicionar novos tipos de pagamento sem precisar
modificar a classe `CarrinhoDeCompras`, o que segue o princípio do Aberto-Fechado. Isso torna o código mais flexível e
fácil de ser mantido.

## Utilizando Herança

### Exemplo 1: Adicionando novos tipos de veículos

#### Antes:

```java
class ControleDeFrota {
    public void adicionarVeiculo(Veiculo veiculo) {
        // Lógica para adicionar veículo
    }
}
```
```java
class Veiculo{
    // atributos e lógicas de veículo
}
```

Neste exemplo, temos uma classe `ControleDeFrota` que tem um método para adicionar veículos. Se, no futuro, precisarmos
adicionar novos tipos de veículos (por exemplo, caminhão ou motocicleta), precisaríamos modificar essa classe, o que
violaria o princípio do Aberto-Fechado.

#### Depois:

```java
abstract class Veiculo {
    // atributos e métodos comuns a todos os veículos
}
```
```java
class Carro extends Veiculo {
// atributos e métodos específicos de carro
}
```
```java
class Caminhao extends Veiculo {
    // atributos e métodos específicos de caminhão
}
```
```java
class Motocicleta extends Veiculo {
// atributos e métodos específicos de motocicleta
}
```

```java
class ControleDeFrota {
    public void adicionarVeiculo(Veiculo veiculo) {
        // Lógica para adicionar veículo
    }
}
```

Neste exemplo, criamos uma classe abstrata `Veiculo` que define os atributos e métodos comuns a todos os veículos e as
classes `Carro`, `Caminhao` e `Motocicleta` que estendem essa classe abstrata e definem os atributos e métodos específicos de
cada tipo de veículo. A classe `ControleDeFrota` tem um método para adicionar veículos e este método aceita como parâmetro
qualquer objeto que seja uma subclasse de `Veiculo`. Dessa forma, podemos adicionar novos tipos de veículos sem precisar
modificar a classe `ControleDeFrota`, o que segue o princípio do Aberto-Fechado. Isso torna o código mais flexível e fácil
de ser mantido.

### Exemplo 2: Adicionando novos tipos de veículos

#### Antes:

```java
class Veiculo {
    public void mover() {
        // Lógica para mover veículo
    }
}
``` 

Neste exemplo, temos uma classe `Veiculo` que tem um método para mover o veículo. Se, no futuro, precisarmos adicionar
novos tipos de veículos (por exemplo, carro ou bicicleta), precisaríamos modificar essa classe, o que violaria o
princípio do Aberto-Fechado.

#### Depois:

```java
class Veiculo {
    public void mover() {
        // Lógica para mover veículo
    }
}
```

```java
class Carro extends Veiculo {
    public void mover() {
        // Lógica específica para mover carro
    }
}
```

```java
class Bicicleta extends Veiculo {
    public void mover() {
        // Lógica específica para mover bicicleta
    }
}
```

Neste exemplo, temos a classe Veiculo que define um método para mover o veículo e as classes Carro e Bicicleta que
estendem essa classe e sobrescrevem o método mover para adicionar lógica específica para cada tipo de veículo. Assim,
podemos adicionar novos tipos de veículos sem precisar modificar a classe Veiculo, o que segue o princípio do
Aberto-Fechado. Isso torna o código mais flexível e fácil de ser mantido.

## Utilizando Composição

### Exemplo 1: Adicionando novos tipos de envio de e-mails

#### Antes:

```java
class EnviadorDeEmails {
    public void enviarEmail(String destinatario, String assunto, String mensagem) {
        // Lógica para enviar email
    }
}
```

Neste exemplo, temos uma classe EnviadorDeEmails que tem um método para enviar e-mails. Se, no futuro, precisarmos
adicionar novos tipos de envio de e-mails (por exemplo, envio via SMTP ou envio via API), precisaríamos modificar essa
classe, o que violaria o princípio do Aberto-Fechado.

#### Depois:

```java
interface EnvioDeEmails {
    void enviar(String destinatario, String assunto, String mensagem);
}
```

```java
class EnvioViaSmtp implements EnvioDeEmails {
    public void enviar(String destinatario, String assunto, String mensagem) {
// Lógica para enviar email via SMTP
    }
}
```

```java
class EnvioViaApi implements EnvioDeEmails {
    public void enviar(String destinatario, String assunto, String mensagem) {
// Lógica para enviar email via API
    }
}
```

```java
class EnviadorDeEmails {
    private EnvioDeEmails envioDeEmails;

    public void setEnvioDeEmails(EnvioDeEmails envioDeEmails) {
        this.envioDeEmails = envioDeEmails;
    }

    public void enviarEmail(String destinatario, String assunto, String mensagem) {
        envioDeEmails.enviar(destinatario, assunto, mensagem);
    }
}
```

Neste exemplo, criamos uma interface `EnvioDeEmails` que define um método para enviar e-mails e classes
como `EnvioViaSmtp` e `EnvioViaApi` que implementam essa interface. A classe `EnviadorDeEmails` tem uma
propriedade `envioDeEmails` do tipo `EnvioDeEmails` e um método para setar o tipo de envio de e-mails. Dessa forma,
podemos adicionar novos tipos de envio de e-mails sem precisar modificar a classe `EnviadorDeEmails`, o que segue o
princípio do Aberto-Fechado. Isso torna o código mais flexível e fácil de ser mantido.

### Exemplo 2: Adicionando novos tipos de pagamento

#### Antes:

```java
class ProcessadorDePagamentos {
    public void processarPagamento(Pagamento pagamento) {
        // Lógica para processar pagamento
    }
}
```

Neste exemplo, temos uma classe ProcessadorDePagamentos que tem um método para processar pagamentos. Se, no futuro,
precisarmos adicionar novos tipos de pagamento (por exemplo, cartão de crédito ou boleto), precisaríamos modificar essa
classe, o que violaria o princípio do Aberto-Fechado.

#### Depois:

```java
interface Pagamento {
    void processar();
}
```

```java
class PagamentoComCartao implements Pagamento {
    public void processar() {
        // Lógica para processar pagamento com cartão de crédito
    }
}
```

```java
class PagamentoComBoleto implements Pagamento {
    public void processar() {
// Lógica para processar pagamento com boleto
    }
}
```

```java
class ProcessadorDePagamentos {
    public void processarPagamento(Pagamento pagamento) {
        pagamento.processar();
    }
}
```

Neste exemplo, criamos uma interface `Pagamento` que define um método para processar pagamentos e classes
como `PagamentoComCartao` e `PagamentoComBoleto` que implementam essa interface. A classe `ProcessadorDePagamentos` tem
um método para processar pagamentos e este método aceita como parâmetro qualquer objeto que implemente a
interface `Pagamento`. Dessa forma, podemos adicionar novos tipos de pagamento sem precisar modificar a
classe `ProcessadorDePagamentos`, o que segue o princípio do Aberto-Fechado. Isso torna o código mais flexível e fácil
de ser mantido.

Esses foram alguns exemplos de como podemos utilizar interface, herança e composição para seguir o princípio do
Aberto-Fechado (OCP) e garantir que nossas classes sejam mais flexíveis e fáceis de serem mantidas. Lembre-se de que não
existe uma única maneira de seguir um princípio de design e que cada caso deve ser avaliado individualmente para
escolher a melhor abordagem.

## Como implementar o OCP

Existem algumas maneiras de seguir o princípio do Aberto-Fechado em seu código:

- Utilizando herança: uma forma de seguir o OCP é criando subclasses que estendem uma classe base. Dessa forma, é
  possível
  adicionar novos comportamentos sem precisar modificar a classe base.

- Utilizando interfaces: outra forma de seguir o OCP é criando interfaces que definem comportamentos específicos, e
  classes que as implementam. Dessa forma, é possível adicionar novos comportamentos sem precisar modificar as classes
  existentes.

- Utilizando padrões de projeto: alguns padrões de projeto, como o Strategy, o Template Method e o Decorator, seguem o
  princípio do Aberto-Fechado e podem ser utilizados para adicionar novos comportamentos sem precisar modificar o código
  existente.

## Quando não seguir o OCP

É importante lembrar que não é sempre necessário seguir o princípio do Aberto-Fechado. Às vezes, é melhor modificar o
código existente do que adicionar novas classes ou interfaces. É importante avaliar qual a melhor opção para cada caso
específico. Além disso, existem casos em que o OCP não é aplicável, como por exemplo, em alguns casos de classes
auxiliares ou classes de infraestrutura.

## Conclusão

O princípio do Aberto-Fechado é uma boa prática para desenvolvimento de software, pois permite adicionar novos
comportamentos sem precisar modificar o código existente. Isso torna o sistema mais flexível, fácil de ser mantido e
reutilizável. No entanto, é importante avaliar quando seguir esse princípio e quando não seguir.