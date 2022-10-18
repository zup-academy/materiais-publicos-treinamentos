# SRP - Single Responsibility Principle ou Princípio da Reponsabilidade Única

## O que é?

Como o próprio nome já diz, o Princípio da Reponsabilidade Única diz que cada módulo ou classe deve ter apenas uma
responsabilidade.
Robert "Uncle Bob" Martin
diz [neste artigo](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html) algo como: "A
classe precisa ter apenas um motivo para mudar".

Como esse princípio nos ajuda a criar um 'software' melhor? Vamos ver alguns dos seus benefícios:

- Teste — uma classe com uma responsabilidade terá muito menos casos de testes.
- Menor acoplamento — menos funcionalidades numa única classe terá menos dependências.
- Organização — classes menores e bem organizadas são mais fáceis de pesquisar do que as classes monolíticas.
- Coesão — classes coesas são mais fáceis de serem mantidas, reutilizadas e tendem a ter menos bugs (como diria Alberto
  Souza: "a única certeza que temos ao desenvolver um software é que ele tem bugs", mas nossa trabalho é fazer nosso
  melhor para evitá-los!)