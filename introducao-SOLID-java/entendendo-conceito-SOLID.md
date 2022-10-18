# S.O.L.I.D

## O que é SOLID?

Como pessoa desenvolvedora de software, é comum tentar implementar as melhores soluções para nossos clientes,
considerando os requisitos, restrições e demais instruções dadas.
Mas algo igualmente comum são os constantes pedidos de alteração do projeto, independente da fase que esse projeto se
encontra.
Não é raro também, trabalhar num código instável e ainda assim as mudanças serem necessárias a todo o momento, com
regras de negócio cada vez mais complexas. Sendo necessário assim, nos atentarmos para a organização do código e para o
fato de que _bugs_ sempre estarão lá (talvez só não foram descobertos ainda), mas que temos que nos esforçar ao máximo
para produzir a menor quantidade de _bugs_ possível.

Além desses casos, temos também as boas práticas da **programação orientada a objetos**, além da busca por termos sempre
o melhor _design_ para se adequar a arquitetura que utilizamos.
Embora não haja nenhuma bala de prata para cada situação, os times comumente recorrem ao SOLID para evitar esses
problemas e manter o código da melhor maneira possível.

**Mas o que é o SOLID?**

SOLID é um conjunto de princípios e boas práticas para a programação orientada a objetos, pensado para melhorar o
_design do software_, tornando o código mais fácil para se manter, escalar e testar.
Esse termo surgiu pela sugestão de _Michael Feathers_ algum tempo após _Robert "Uncle Bob" Martin_ introduzir os cinco
princípios da programação orientada a objetos no artigo _Principles Of OOD_.

A palavra SOLID é um acrônimo que significa:

- **S**ingle Responsibility Principle (Princípio da responsabilidade única)
- **O**pen-Closed Principle (Princípio Aberto-Fechado)
- **L**iskov Substitution Principle (Princípio da substituição de Liskov)
- **I**nterface Segregation Principle (Princípio da Segregação da Interface)
- **D**ependency Inversion Principle (Princípio da inversão da dependência)

Na sua publicação, o _Uncle Bob_ lista quatro “cheiros ruins” que as boas práticas devem ser capazes de evitar:

1. **Rigidez**: mudanças simples em um único módulo de um projeto resultam em alterações de várias classes de outros
   módulos,
   consumindo uma grande quantidade de tempo;
2. **Fragilidade**: alterações a um único ponto do código podem causar inúmeros efeitos secundários;
3. **Imobilidade**: o código dentro do projeto não pode ser reutilizado com facilidade, pois possui muitas dependências;
4. **Viscosidade**: existem dois tipos de viscosidade:
    - **_Design_**: fazer alterações que preservam o _design de software_ são consideravelmente mais difíceis do que
      fazer
      gambiarras;
    - **Ambiente**: um ambiente de build muito lento faz com que os desenvolvedores prefiram soluções que requerem menos
      recursos do sistema (por exemplo: tempo de compilação), mas essas soluções podem quebrar o design.

Ele também menciona ser responsabilidade dos desenvolvedores preparar a arquitetura para lidar com as mudanças de
requisitos e que isso deve ser feito com uma boa gestão de dependência.



