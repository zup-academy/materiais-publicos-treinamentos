#O que é Log e porque é importante?

Log é o processo de capturar dados/informações e registrá-las para análise.
Falando de maneira mais direcionada, quando estamos falando de logs em software é regitrar o que esta sendo feito em um determinado momento da execução e salvar esse dado para ser consultado posteriormente.

Muitas vezes não temos acesso às máquinas de produção que nossa aplicação esta executando, além disso é necessário que tenhamos registro do que ocorreu e como ocorreu quando algum fluxo de negócio executou com comportamento não esperado.
Os logs são amplamente utilizados especialmente para análise quando algum comportamento não esperado ocorre durante a execução do software, como um erro por exemplo.
Esse log pode ser utilizado para entender o que ocorreu no momento do erro.

Através dos logs, conseguimos registrar se as informações chegaram ou foram enviadas para um serviço da meneira correta, se um caminho que era esperado ser executado foi de fato executado, etc.
Os logs nem sempre indicarão erro, mas podem indicar informações que levarão a identificar uma lógica implementada de maneira erronea.

Imagine que os logs são direções para um lugar que desejamos encontrar, por isso é importante implementá-los de maneira efetiva, que nos forneça o que precisarmos quando for necessário.

Por esse motivo é muito importante entender sobre as boas práticas que estão relacionadas com logs.
Para que ao implementarmos tenhamos como extrair a quantidade máxima de benefícios possíveis.

É importante você ter em mente que logs devem ser pré requisito ao implementar features no seu sistema.
Não espere ocorrer um problema para que assim você coloque logs, você irá utilizar para buscar a fonte dos problemas quando for necessário.

Falar sobre as estratégia de logs em sua aplicação entre seu time, pod, área é fundamental, por isso procure saber como seu time trabalha nesse ponto e caso não saibam levante essa questão para ser discutida o quanto antes.
