# Introducão a Bloqueio Otimista

Ao lidar com ambiêntes concorrêntes não podemos ficar refém de uma unica estratégia. Inserir bloqueios exclusivos em todas ocasiões pode se tornar custoso, não é sempre que queremos que a quantidade de threads executadas seja reduzida.

O bloqueio pessimista é a estrategia apropriada para quando estamos lidando com prevenção de conflitos, ou seja, dado uma situação nós precavemos e evitamos atualizações concorrentes em um unico registro. O custo desta estrátegia é que podemos gerar a anomalia de atualização perdida.


## Anomalia de Atualização Perdida

Para entender a anomalia de atualização perdida, imagine que estamos trabalhando em um sistema de estoque, onde diversos funcionarios concorrem a um determinado insumo. 

Suponha que o insumo com id igual a um, é uma camiseta estampada e conta com apenas uma quantidade. A entidade pode ser representada pelo seguinte Json.

```json
    {
        "id":1,
        "nome":"camiseta estampada nike",
        "quantidade":1
    }
```

Agora imagine que o `vendedor 1` e `vendedor 2` estão atendendo clientes diferentes no estabelecimento e estes clientes se interessaram na camiseta. Então ambos os vendedores iniciam a transação de compra, e o `vendedor 1` finaliza a compra, atualizando a quantidade para zero. O `vendedor 2` não ciente que a quantidade foi alterada para zero, finaliza a compra, e o resultado é que o registro que representa a camiseta fica da seguinte forma. 

```json
    {
        "id":1,
        "nome":"camiseta estampada nike",
        "quantidade":-1
    }
```

A anomalia de atualização perdida foi dada quando o `vendedor 2` acha que a camiseta ainda esta disponivel, porém já não esta.

A solução para esta problematica pode ser feita através de um mecanismo de detecção de conflitos. Este mecanismo por si, tera que verificar se o estado da entidade no momento de atualização é o mesmo que no memomento de leitura. Esta estratégia é o que chamamos de Bloqueio Otimista.