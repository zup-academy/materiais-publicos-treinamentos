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
  Souza: "a única certeza que temos ao desenvolver um software é que ele tem bugs", mas nossa trabalho é fazer nosso
  melhor para evitá-los!)

## Coesão

