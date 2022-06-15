## Ecommerce e marketplace de todo tipo de produto desse planeta

O Mercado Zup é uma local online onde é vendido de tudo. Livros, computadores, eletrodomésticos etc. 

Além de ter os produtos ofertados pelo próprio site, existe também a possibilidade de pessoas colocarem seus produtos lá dentro para que tudo seja vendido. Essa é uma via mais de marketplace. 

Para construir tudo isso, foram utilizadas diversas tecnologias, pensando em linguagens de programação e também em stacks construídas em cima dessas linguagens. Abaixo temos alguns exemplos de linguagens que foram utilizadas e uma ideia de quais tipos de problemas mais genéricos que elas foram lembradas para endereçar:

1. Java para a construção da grande maioria das apis de negócio
1. Javascript para a construção de frontends 
1. Javascript para Backend for Front Ends(BFF). Aqui temos também casos onde foi preferido usar um GraphQL para expor os serviços
1. Python para processamento de modelos de aprendizado de máquina(machine learning). 
1. Python para limpeza e análise de dados

Pensando nas stacks utilizadas em cima dessas tecnologias, temos algumas combinações:

1. Spring Boot + Java para apis
1. Micronaut + Java para apis
1. NextJS para construção de frontends com React
1. NestJS para construção de BFFs
1. Python + scikit-learn para aplicação de modelos de aprendizado de máquina em cima dos dados(conferir com geraldo)
1. Google colab para criação de análises (conferir com Geraldo)

A empresa conta com mais de 4000 pessoas na engenharia e existem vários desafios que são constantes dentro desse ecossistema:

1. Quando vamos criar novos projetos como podemos reaproveitar conhecimento que já existe para não perder nesta fase?
1. Quando precisamos acessar recursos internos, como podemos adiantar o lado e não perder tempo descobrindo como faz isso?
1. Quando fazemos uma coisa legal, como podemos distribuir esse conhecimento de uma forma mais palpável para as pessoas? 
1. Se temos algum padrão de código que é utilizado no contexto, como podemos maximizar a chance de ter esse padrão sendo reaproveitado pelas próprias pessoas do contexto e até sendo reaproveitado por outras equipes?

## Exemplos de desafios do nosso dia a dia de engenharia

A galera da stackspot fez um aprofundamento sobre os desafios de distribuição de conhecimento e também no que é entendido que pode acelerar a produção, algumas situações indesejadas foram separadas. 

1. Um novo projeto vai começar, independente da stack, e ninguém sabe quais são as versões dos artefatos que normalmente são usadas. Todo projeto tem versão diferente de framework de orm, lib de comunicação http, lib de validação, framework mvc etc. 
1. Tem projeto que começa precisando apenas se comunicar com um banco de dados relacional. Só que também tem projeto que já começa sabendo que, além de falar com o banco de dados relacional, também vai precisar mandar uma mensagem para nosso kafka. Vamos deixar algumas situações que foram investigadas. 
    1. Projeto quer utilizar uma stack básica já utilizada em outros lugares
    1. Projeto começa precisando falar com um postgresql
    1. Projeto começa precisando mandar mensagem para nosso kafka. 
    1. Projeto começa precisando receber mensagem de uma ou mais filas do kafka
    1. Projeto começa precisando mandar e consumir informação do cache. No caso utilizamos o redis. 
    1. Projeto começa sabendo que vai precisar falar com uma api que existe internamente no nosso ecossistema. 
    1. Projeto já começa sabendo que vai seguir a arquitetura de software sugerida pela Clean Architecture
    1. Projeto já começa sabendo que precisa habilitar autenticar e autorização usando o que é recomendado internamente. 

Além disso, existem as situações que acontecem dentro dos projetos que estão em andamento.

1. Quero adicionar um novo endpoint post dentro da minha api que recebe alguns parâmetros, tem validação e precisa ser implementado seguindo a sugestão arquitetural do projeto. 
1. Quero adicionar um novo endpoint get dentro da minha api que recebe alguns parâmetros, tem validação e precisa ser implementado seguindo a sugestão arquitetural do projeot. 
1. Quero adicionar um novo código que manda mensagem para uma fila do kafka que já existe
1. Quero adicionar um novo listener para uma fila do kafka que já existe
1. Agora eu preciso consumir uma api interna que eu sei que existe, mas não conheço o contrato e nem sei para quem perguntar direito. Isso daqui machua demais o nosso o time. 
1. Quero consumir algum serviço da aws que eu sei que existe lib, mas nunca usei. 
1. Quero consumir algum serviço da aws que eu sei que existe lib, já usei e agora tenho que copiar o meu código mais uma vez. 


## Exemplos de desafios para clientes que consomem nossas apis

Nosso produto é um sucesso e muitas vezes as pessoas querem exibir os produtos em seus próprios sites. 

Isso acontece principalmente com todo mundo que integra nosso programa de afiliados. Para uma parte dessas pessoas que não tem habilidades de programação, simplesmente oferecemos um snippet de código que ela precisa colocar no seu site. Algo como o google analytics faz. 

Só que existe um outro grupo de pessoas que são programadoras e elas preferem customizar mais o jeito que os produtos vão aparecer. Este grupo prefere usar api's diretas. Essas pessoas reclamam recorrentemente do esforço de ler a nossa documentação, entender qual o código precisa ser escrito, escrever, testar etc. Se isso fosse mais fácil, seria bem melhor. 

Todos os nossos endpoints públicos precisam ser acessados com uma chave de identificação definida no header da requisição, além disso também é necessário entender os parâmetros que precisam ser passados e os retornos. Alguns exemplos:

1. Temos uma api que lista todos os nossos produtos 
1. Temos uma api que exibe o detalhe de um dos produtos
1. Temos uma api que lista os produtos mais vendidos
1. Temos uma api que espera uma query de filtro para que possa retornar só o que faz sentido. 
1. Aceitamos que a afiliada configure um webhook de nova venda usando seu código de afiliado(a) para que ela possa fazer o que quiser com essa nova informação. 

Tudo isso tem documentação e até exemplos de consumo com Java, Javascript e Python. Mesmo assim as pessoas ainda reclamam de ter que ler tudo, copiar o código, as dependências etc. 

