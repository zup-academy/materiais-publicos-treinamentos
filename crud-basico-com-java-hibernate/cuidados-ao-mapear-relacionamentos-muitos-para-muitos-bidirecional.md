# Cuidados ao lidar com Relacionamentos para Muitos `(ToMany)` com JPA e Hibernate

Não é de hoje que sabemos que JPA e Hibernate proporcionam agilidade e produtividade atráves de suas APIS. Mapear relacionamentos nunca foi tão simples, porém, quando lidamos com relacionamentos para Muitos (`ToMany`) devemos nós atentar para manter a consistência das colleções, e para não executar consultas indesejadas. 

Neste artigo iremos nos atentar em manter a sincronização das colleções, em relacionamentos Muitos para Muitos.

## Mantendo a consistência de Relacionamentos Muitos para Muitos Bidirecional

Sabemos que um relacionamento é bidirecional quando o mapeamento aparece nas duas entidades que fazem parte da relação. Para descobrir qual entidade não detem a relação, procuramos pelo campo mappedBy na anotação `@ManyToMany`. Neste contexto o que acontece é que as alterações são realizadas na entidade que detem o relacionamento e são propagadas para a outra, entretanto em memoria as informações não são atualizadas, então cabe ao desenvolvedor atualizar a colleção do lado mais fraco da relação.

Abaixo será detalhado em codigo como delegar a responsabilidade de sincronização em memoria das ocorrências de um relacionamento bidirecional a  métodos para adicionar e remover unidades de uma colleção.


```java
@Entity
public class Aluno{
    @Id
    @GeneratedValue(strategy=IDENTITY)
    private Long id;

    private String nome;
    private String endereco;
    private LocalDate dataNascimento;
    private LocalDate dataMatricula;


    @JoinTable(name="aluno_disciplina",
        joinColumns = @JoinColumn(name = "aluno_id"),
        inverseJoinColumns = @JoinColumn(name = "disciplina_id")
    )
    @ManyToMany(cascade= {
        CascadeType.PERSIST,
        CascadeType.MERGE
    })
    private Set<Disciplina> disciplinas= new HashSet<>();

    public Aluno(String nome, String endereco,LocalDate dataNascimento,LocalDate dataMatricula){
        this.nome=nome;
        this.endereco=endereco;
        this.dataNascimento=dataNascimento;
        this.dataMatricula=dataMatricula;
    }

    /**
     * @deprecated construtor para uso exclusivo do hibernate
     */
    @Deprecated
    public Aluno(){

    }

    public void adicionar(Disciplina disciplina){
        this.disciplinas.add(disciplina);
        disciplina.adicionar(this);
    }

    public void remover(Disciplina disciplina){
        this.disciplinas.remove(disciplina);
        disciplina.remover(this);
    }
}
```

```java
@Entity
public class Disciplina{
    @Id
    @GeneratedValue(strategy=IDENTITY)
    private Long id;

    private String titulo;

    private Integer duracaoEmHoras;

    @Lob
    private String ementa;

    @ManyToMany(mappedBy="disciplinas")
    private Set<Aluno> alunos = new HashSet<>();

    private LocalDateTime criadoEm=LocalDateTime.now();

    public Disciplina(String titulo, Integer duracaoEmHoras, String ementa){
        this.titulo=titulo;
        this.duracaoEmHoras=duracaoEmHoras;
        this.ementa=ementa;
    }

    /**
     * @deprecated construtor para uso exclusivo do hibernate
     */
    @Deprecated
    public Disciplina(){

    }

    public void adicionar(Aluno aluno){
        this.alunos.add(aluno);
    }

    public void remover(Aluno aluno){
        this.alunos.remove(aluno);
    }
}
```