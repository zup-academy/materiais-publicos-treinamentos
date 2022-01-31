## O que são Transactions?

Ao usar sistemas de banco de dados relacionais, notamos que as operações são pautas em dois estados, que são  COMMIT e ROLLBACK.
 - COMMIT: é enviar, significa que as operações feitas com as informações serão efetivadas no Banco de Dados (BD)
 - ROLLBACK: é reverter, signfica que as operações feitas com as informações não serão efetivadas, ou seja, a operação será cancelada.

 para exemplificar melhor, imagine que exista uma tabela Pessoa com as seguintes colunas: id, nome, cpf, email, rg e data de nascimento. E agora você precisa inserir o registro do Jose que tem os seguites dados:

- nome: Jose Fernando Alves
- cpf: 43797369034
- rg: 139154917
- email: jose.fernando@email.com
- data nascimento: 25/01/1992

provavelmente você ira no seu Sistema de Gerenciamento de Banco de Dados (SGBD) e fará um script com operação de inserção igual a mostrada abaixo.

```sql
insert into PESSOA (nome,cpf,rg,email,dataNascimento) values ('Jose Fernando Alves','43797369034','139154917','jose.fernando@email.com',DATE('1992-01-25'));
```

Caso nenhuma constraint seja quebrada ocorrerá o commit desta operação, ou seja, na tabela pessoa existira um registro com as informações de Jose. Caso alguma constraint seja quebrada ocorrerá o rollback da operação. Este comportamento é feito para manter a integridade dos dados, já que um BD  processa diversas operações em uma unidade de tempo. O mecânismo implmentado para garantir a integridade são as Transações (Transactions).

Uma Transação é uma unidade de processamento formado pelo agrupamento de uma ou mais operações, onde todas operações tem sucesso ou falham juntas. Todas operações são realizadas em um contexto transacional, que pode ser definido explicitamente ou não.

As transações são baseadas em quatro propriedades: Atomicidade (Atomicity), Consistência (Consistency), Isolamento (Isolation) e Durabilidade (Durability) que formam o acrônimo ACID.

O que cada uma das propriedades significam?

1.   Atomicidade: é a propriedade responsavel por agrupar as operações em uma unidade de processamento. Esta propriedade caracteriza tudo ou nada, ou todas operações tem sucesso e as alterações são commitadas, porém se uma falha todas as outras são revertidas no rollback.

2. Consistência: esta propriedade é responsavel por manter a consistência das alterações de uma transação, caso alguma regria seja violada, todas as operações devem ser revertidas garantindo que o estado dos dados estejam sempre adequados.

3. Isolamento: esta propriedade é responsavel por garantir que transações concorrentes não gerem conflitos e não comprometam a integridade dos dados.

4. Durabilidade: é a propriedade que garante que todas as alterações de transações sejam permanentes.


