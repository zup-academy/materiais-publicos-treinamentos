# Entendendo o protocolo HTTP

Sempre que acessamos a um site, digitamos seu endereço na barra do navegador e, em seguida, é apresentado o conteúdo que buscamos, caso exista. Na web, esse é o fluxo padrão. Não importo o serviço que vamos acessar sempre fazemos um pedido (requisição) para um outro computador (servidor web) que nos gera uma resposta. Esse fluxo básico funciona através de uma regra de comunicação entre as máquinas chamada de HTTP.

## Entendendo o HTTP

O HTTP é um protocolo - isto é, uma regra de comunicação - utilizado na internet. É pelo utilizando o HTTP que os navegadores conseguem fazer requisições para os servidores executarem algo e devolverem uma resposta. Portanto, o HTTP é um protocolo do tipo Cliente/Servidor.

Com esse protocolo, conseguimos definir como realizaremos a troca de dados pela web. Será via JSON? Receberemos um HTML? Sera texto puro? Todos esses dados são trafegados via HTTP pelo protocolo TCP (apesar que qualquer protocolo confiável poderia ser utilizado para transporte).

A imagem abaixo ilustra o modelo de Requisição/Resposta, também chamado de Request/Response:

![Requisição e resposta](./imagens/request-response.png)

### Um pouco sobre a URL

Já vimos como o HTTP funciona no alto nível, mas agora vamos destrinchar o protocolo e entender como ele trafega os dados e a responsabilidade de cada parte mais a fundo.

Sempre que acessamos um site, por exemplo `www.zup.com.br`, digitamos o domínio, isto é, onde o recurso que queremos acessar está localizado. Quando passamos apenas o domínio, o servidor devolve para a gente o recurso localizado na raíz (root) ou `/`. 

Uma pessoa pode querer acessar a página de carreiras na Zup e para isso, adicionaria `/carreiras` no domínio ficando: `www.zup.com.br/carreiras`. Note que dessa vez o recurso acessado é `/carreiras` que se localiza dentro do domínio `www.zup.com.br`. 

É comum que antes do domínio, apareça o protocolo que usamos. Por estarmos falando do HTTP (ou HTTPS) a URL final ficaria assim: `http://www.zup.com.br/carreiras`. 

>
> A URL nada mais é do que uma forma padronizada de escrever e de localizar recursos através da rede.
>
> Esse padrão define que o formato da url é: 
>
> \<protocolo\>://\<domínio\>:\<porta\>/\<recurso\>
>


### Obtendo dados de um recurso

Quando acessamos a página de carreiras da Zup, o que queremos é **obter** os dados da página para que o navegador mostre os elementos na tela. Ou seja, pedimos para o servidor esses dados e este nos devolve o pedido. 

Se interceptarmos uma requisção (`request`) notaremos que ela se parecerá com a requisição a seguir

```
 GET /carreiras HTTP/1.1
 Host: www.zup.com.br
 Accept: text/html
```

Isso que estamos vendo é o cabeçalho de uma requisição. Nele temos algumas informações sobre:

1. Qual o método (também chamado Verbo) que será utilizado na requisição, o recurso acessado e qual a versão do protocolo será utilizado
1. Qual o host onde a requisição será performada
1. Qual o tipo de dado o cliente espera receber

Neste cabeçalho podemos adicionar outras informações como dados de autenticação, linguas aceitas, qual o tipo do cliente HTTP (Navegador, celular, outro servidor), entre diversas outras informações. 

Outra informação importante é o método (ou verbo) que foi utilizado para o recurso. Com o HTTP, podemos pedir para os servidores executarem determinados comportamentos para um mesmo recurso. 

Vamos supor que queremos listar todos os livros de nossa biblioteca particular. Para isso, pederiamos ter um recurso `/livros` e pedir para o HTTP obter (`GET`) esses dados para a gente.

Já se quiséssemos criar um livro, poderíamos postar (`POST`) essas informações para o servidor e ele se encarregaria de salvá-las. 

O HTTP define alguns métodos no seu protocolo. Os mais utilizados no dia a dia são o `GET`, `POST`, `PUT`, `PATCH` e `DELETE`. Cada um desses métodos têm uma semântica específica.


O método `GET` é o que utilizamos para buscas. A semântica dele diz que não devemos alterar estado no servidor quando fazemos uma requisição com esse método. Logo é um método idempotente.

O método `POST` é utilizado para adicionar dados. Por exemplo, se quisermos salvar um novo livro na nossa coleção, podemos utilizar o método `POST`. 

Para remover, utilizamos o método `DELETE` e para atualizar os dados no servidor, podemos utilizar o `PUT` ou o `PATCH`. A diferênça aqui se dá que enquando a semântica do `PUT` diz que deveríamos atualizar todo o recurso, ou seja, todos os dados do livro enquanto que com o `PATCH` atualizamos apenas uma parte do recurso.

Vimos que no cabeçalho de uma requisição tem os metadados relativos a request, mas onde ficariam os dados de um livro que seria salvo, por exemplo? 

Para isso, é usado o corpo (body) da requisição. Toda requisição HTTP é composta por um cabeçalho e, opcionalmente, um corpo com os dados que serão enviados do cliente para o servidor.

Por exemplo, uma requisição para salvar um livro seria algo como:

```
POST /livros HTTP/1.1
Content-Type: application/json
Authorization: abcd123y873841.abc1637481748794.90934i
Host: localhost:8080
Content-Length: 77

{
	"autores": [1, 2, 9],
	"titulo": "Era uma vez...",
	"isbn": "1230908962"
}
```

No corpo da requisição é informado o JSON contento os dados que serão enviados ao servidor. Podemos também utilizar outros tipos de formato de texto. O HTTP trafega os textos em formato aberto.

Sabemos que o HTTP é um protocolo Cliente/Servidor, portanto para cada requisição, o servidor pode nos responder de formas diferentes.

### Olhando para a resposta

Quando queremos listar os livros de nossa biblioteca, fazemos um `GET` para algo como `/livros` e recebemos a listagem com todos os exemplares, mas o que recebemos quando criamos um novo livro? 

A operação para criar um livro (`POST`) apenas diz para o servidor que é para inserir um novo recurso, como o servidor pode nos informar que esse recurso foi criado? É nesse cenário que surge o código de status, ou **status code**

A resposta do HTTP, assim como a request, contém um header e pode conter um corpo. Nesse header, é comum que os servidores web sempre retornem um status para sumarizar o resultado da aplicação. 

Esses códigos vêm em faixas onde o primeiro número diz a qual grupo ele pertence. Por exemplo

- **1xx**: Respostas de informação
- **2xx**: Respostas de sucesso
- **3xx**: Redirecionamentos
- **4xx**: Erros do cliente
- **5xx**: Respostas do servidor

Voltando a listagem, podemos receber um simples status **200** que significa *OK* junto a lista de livros. 

No caso da criação, o status mais adequado seria o **201** que significa *Recurso Criado* ou *created*.

Uma exemplo de resposta para o `POST` de criação seria assim:

```
HTTP/1.1 201 created 
Date: Fri, 28 Jan 2022 19:49:58 GMT
Location: /livros/99
```

Nessa resposta vemos o cabeçalho da requisição com a versão do protocolo HTTP, seu status e mensagem. Além disso foi informado a data que ocorreu a resposta e a localização (`Location`) do recurso criado.

A resposta poderia ter um body seguindo as mesmas especificações da request. 

Um ponto que devemos nos atentar ao HTTP é que seus dados são trafegados em forma de texto aberto. Ou seja, qualquer um que interceptar uma request poderia olhar o body e os headers para ter acesso a informação.

> Mas se os dados são trafegados em formato aberto, o que acontece com os dados sensíveis?

O HTTP é um protocolo extensível, ou seja, pode ser adicionano novas funcionalidades a ele. Para resolver essa parte de segurança, foi criado o protocolo HTTPS, ou Hyper Text Transfer Protocol Secure.

Com o HTTPS todo dado no body e no header no HTTP é criptografado, com exceção da URL já que ela é necessária para roteamento.

## Para saber mais

- [Visão geral sobre o HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Overview)
- [Status HTTP](https://httpstatusdogs.com/)
- [Como o HTTPS funciona](https://howhttps.works/pt-br/)
