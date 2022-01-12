# Sobre Swift

>Esse material teórico foi traduzido e editado da fonte original disponível em https://www.swift.org/about/.

Swift é uma linguagem de programação de propósito geral construída usando uma abordagem moderna para segurança, desempenho e padrões de design de software.

O objetivo do projeto Swift é criar a melhor linguagem disponível para usos que vão desde a programação de sistemas como aplicativos móveis e de desktop, até serviços em nuvem. Mais importante ainda, o Swift foi projetado para tornar a escrita e manutenção de programas mais fácil para o desenvolvedor. Para atingir esse objetivo, a maneira mais óbvia de escrever código Swift também deve ser:

**Seguro**. A maneira mais óbvia de escrever código também deve ser de maneira segura. O comportamento indefinido é inimigo da segurança e os erros do desenvolvedor devem ser detectados antes que o software esteja em produção. Optar pela segurança às vezes significa que o Swift parecerá rigoroso, mas acredita-se que a clareza economiza tempo no longo prazo.

**Rápido**. O Swift pretende ser um substituto para linguagens baseadas em C (C, C ++ e Objective-C). Como tal, o Swift deve ser comparável a essas linguagens em desempenho para a maioria das tarefas. O desempenho também deve ser previsível e consistente, não apenas rápido em execuções curtas que exigem trabalho posterior.

**Expressivo**. O Swift se beneficia de décadas de avanços na ciência da computação para oferecer uma sintaxe que é divertida de usar, com recursos modernos que os desenvolvedores esperam. Mas o projeto Swift não é algo concluído. Os mantenedores estarão sempre monitorando os avanços para propor melhorias para a linguagem e abraçar o que funciona, evoluindo continuamente para tornar a linguagem ainda melhor.

## Recursos da linguagem

O Swift inclui recursos que tornam o código mais fácil de ler e escrever, enquanto fornece ao desenvolvedor o controle necessário. O Swift oferece suporte a tipos inferidos para tornar o código mais limpo e menos sujeito a erros, e os módulos eliminam cabeçalhos e fornecem namespaces. A memória é gerenciada automaticamente e você não precisa se preocupar com um ponto-e-vírgula. O Swift também pega emprestado de outras linguagens, por exemplo, os parâmetros nomeados trazidos do Objective-C são expressos em uma sintaxe limpa que torna as APIs em Swift fáceis de ler e manter.

Os recursos da Swift são projetados para trabalhar juntos criando uma linguagem poderosa. Alguns recursos adicionais do Swift incluem:

- Closures unificadas com ponteiros de função
- Tuplas e múltiplos valores de retorno
- Generics
- Iteração rápida e concisa em ranges ou coleções
- Structs que oferecem suporte a métodos, extensões e protocolos
- Padrões de programação funcional, por exemplo, mapping e filtering
- Poderoso tratamento de erros integrado
- Fluxo de controle avançado com palavras-chave como `do`, `guard`, `defer` e `repeat`

## Segurança

O Swift foi projetado desde o início para ser mais seguro do que as linguagens baseadas em C e elimina um grande conjunto de estruturas de código inseguro. As variáveis são sempre inicializadas antes do uso, arrays e inteiros são verificados quanto a estouro e a memória é gerenciada automaticamente. A sintaxe é ajustada para facilitar expressar sua intenção - por exemplo, palavras-chave simples de três caracteres definem uma variável (var) ou constante (let).

Outro recurso de segurança é que, por padrão, os objetos Swift nunca podem ser nulos, e tentar fazer ou usar um objeto nulo resulta em um erro em tempo de compilação. Isso torna a escrita de código muito mais limpa e segura e evita uma causa comum de _crashes_ em tempo de execução. No entanto, há casos em que `nil` é apropriado e, para essas situações, o Swift possui recurso para _optional types_. Um opcional pode conter nulo, mas a sintaxe do Swift força você a lidar com ele com segurança usando `?` para indicar ao compilador que você entende o comportamento e o tratará com segurança.