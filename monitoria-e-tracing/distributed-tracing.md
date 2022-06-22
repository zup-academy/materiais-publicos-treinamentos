# Distributed Tracing

Muitas vezes é necessário fazermos um tracing de um fluxo de negócio. Ou seja, vamos procurar através do passo-a-passo das aplicações para compreendermos algum comportamento nestes fluxos. 
E fazer isso pode ser bem desafiador e complexo quando trabalhamos com arquitetura de microserviços.

O termo inglês "tracing", que significa rastreamento. Existem formas de realizar o tracing de acordo com o tipo de processamento, como:

### Sem concorrência

Utiliza-se de uma técnica básica pois cada processo será responsável por executar um fluxo completo de negócio.
Neste caso implementar os logs e realizar tracing é mais simples, sendo necessário identificar o processo e o servidor.

### Multithreading

Neste cenário, um processo pode utilizar mais de uma tarefa de maneira paralela ou concorrente, como por exemplo criando multiplas threads para executar no mesmo processo que corresponde a um fluxo de negócio.
Neste caso é mais complicado fazer o tracing do que foi executado , neste caso é importante informar no log a thread além do processo e do servidor.

### Processos Assincronos

Neste cenário existe a necessidade de mapeamento de partes do processo que podem executar de maneira assincrona para um fluxo de negócio.
Neste caso é importante identificar o processo como um todo para que ao registrar logs seja possível identificar as partes envolvidas no fluxo como um todo.

### Processos Distribuídos / Microserviços

Neste caso um fluxo de negócio envolve multiplos processos, que podem envolver fluxos assincronos .
Então precisamos identificar os logs de uma forma que possamos agregar as informações correlacionadas ao mesmo fluxo de negócio.

## O que é 

É o conjunto de técnicas, padrões e práticas com objetivo de para identificar logs/rastreamento de fluxos de negócio que estão em arquiteturas de sistemas distribuídos, principalmente focado em microserviços.
Imagine que cada log gerado para o fluxo de compra em um marketplace conterá informações que identificam que os logs fazem parte da mesma compra que estava sendo processada por multiplas chamadas a serviços distintos.


