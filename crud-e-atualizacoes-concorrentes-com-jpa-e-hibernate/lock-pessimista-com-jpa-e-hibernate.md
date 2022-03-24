# Inserindo um Bloqueio Pessimista (Pessimistic Locking)  com JPA/Hibernate


Ambientes de alta concorrência ofertam um grande risco a integridade dos dados, o risco de uma informação ser sobrescrita é algo que pode prejudicar o negocio. 

Sistemas de reserva como por exemplo uma biblioteca virtual, é comum que dois usuarios disputem a reserva de um determinado livro. Este livro é representado por um registro em uma tabela em um banco de dados. Se não existir alguma estratégia é possível que dois usuarios leiam o registro e verfiquem que o livro esta disponivel, e que ambos realizem a reserva de forma paralela, ou seja, uma reserva ira sobre escrever a outra.

Uma estratégia para lidar com cenario dito no paragrafo acima é no memento que a primeira transação realizar a leitura do livro, ela insere um bloqueio exclusivo explicito garântindo que apenas ela possa escrever neste registro. O ato de inserir um bloqueio exclusivo é o que chamamos de Bloqueio Pessimista.

## Bloqueio Pessimista

Os bloqueios de escrita garantem que outras transações iram aguardar o termino da transação que obtem o bloqueio para solicitarem permissão de escrita em determinada linha. 

Esta estrategia é usada na prevenção de conflitos, ou seja, garanto que ninguém mais escreve no registro até que eu decida o que fazer.

Para inserir bloqueios de escrita explicito em comandos sql, basta adicionar ao fim da consulta as cláusulas `for update`, e a nivél de codigo utilizando a API da JPA/Hibernate?

### Bloqueio Pessimista com JPA/Hibernate

A interface do EntityManager fornece métodos para consultas com chave primaria e criação de consultas personalizadas, independente do caso é possível informar um atributo que defina o Tipo de bloqueio que iremos inserir. O atributo que define o tipo de bloqueio é o `LockModeType`  que conta com diversas estratégias de bloqeio.

A estratégia de bloqueio pessimista é nomeada como `PESSIMISTIC_WRITE`. Em sua descrição é informado que esta estrategia garante um bloqueio esxclusivo para evitar que qualquer outra transação realize alterações no registro consultado.

### Aplicando o bloqueio pessimista

Para definir um bloqueio pessimista iremos utilizar a entidade OrdemManutencao que esta mapeada abaixo. 

```java
    @Entity
    public class OrdemManutencao {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
        @Column(nullable = false)
        private String titulo;
        @Column(nullable = false)
        private String descricao;
        @Column
        private String dono;
}
```

Para realizar a consulta por chave primaria com bloqueio pessimista utilizando o `EntityManager` fariamos da seguinte forma.

```java
public OrdemManutencao findById(Long id){
    return entityManager.find(OrdemManutencao.class,id,LockModeType.PESSIMISTIC_WRITE);
}
```
Observe o `SQL` gerado pelo Hibernate ao definir esta consulta.

```sql
    select
        ordemmanut0_.id as id1_0_0_,
        ordemmanut0_.descricao as descrica2_0_0_,
        ordemmanut0_.dono as dono3_0_0_,
        ordemmanut0_.titulo as titulo4_0_0_ 
    from
        ordem_manutencao ordemmanut0_ 
    where
        ordemmanut0_.id=? for update
```

### Aplicando Bloqueio Pessimista em consultas personalizadas

O processo para aplicar a estrátegia de bloqueio muda um pouco quando estamos definindo a consulta através do objeto `Query`. Veja a baixo como definir.

```java
public OrdemManutencao findByTitulo(String titulo){
     String consulta="select om from OrdemManutencao om where om.titulo=:titulo"

     
     return entityManager.createQuery(consulta,OrdemManutencao.class)
     .setParameter("titulo",titulo)
     .setLockMode(LockModeType.PESSIMISTIC_WRITE)
     .getSingleResult();
}
```

Observe o `SQL` gerado pelo Hibernate para esta consulta: 

```sql
select
        ordemmanut0_.id as id1_0_,
        ordemmanut0_.descricao as descrica2_0_,
        ordemmanut0_.dono as dono3_0_,
        ordemmanut0_.titulo as titulo4_0_ 
    from
        ordem_manutencao ordemmanut0_ 
    where
        ordemmanut0_.titulo=? for update
            of ordemmanut0_
    
```

### Como definir o tipo de bloqueio em uma consulta em um `Repository` do  Spring Data JPA?

Para definir o tipo bloqueio em consultas nos repositories da JPA, devemos utilizar a anotação `@Lock` que recebe como argumento um `LockModeType`, observe abaixo um exemplo para consulta de ordem de manutenção pelo titulo. 

```java
public interface OrdemManutencaoRepository extends JpaRepository<OrdemManutencao,Long>{
     @Lock(LockModeType.PESSIMISTIC_WRITE)
     Optional<OrdemManutencao> findByTitulo(String titulo);
}
```

Ao realizar a chamada deste metodo verifique o sql gerado pelo Hibernate.

```sql
    select
        ordemmanut0_.id as id1_0_,
        ordemmanut0_.descricao as descrica2_0_,
        ordemmanut0_.dono as dono3_0_,
        ordemmanut0_.titulo as titulo4_0_ 
    from
        ordem_manutencao ordemmanut0_ 
    where
        ordemmanut0_.titulo=? for update
            of ordemmanut0_
```

Definindo a consulta com Bloqueio Pessimista garantimos que apenas a transação que obteve a trava pode escrever neste registro, ou seja, apenas um funcionario pode atribuir para si esta ordem de manuntenção.

## Links para aprofundamento

- [Documentacao LockModeType](https://docs.oracle.com/javaee/7/api/javax/persistence/LockModeType.html)
- [Implementando Lock Pessimista Baeldung](https://www.baeldung.com/jpa-pessimistic-locking)
- [Documentação do @Lock](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Lock.html)