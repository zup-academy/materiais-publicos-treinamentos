# Repository com Spring Data JPA

Existem diversas informações que precisam ser consultadas a partir de uma especificação. E independente de como essas informações estão armazenadas, seja em banco de dados relacionais, ou banco de dados para documentos e até mesmo arquivos de texto,  é importante que estejam agrupadas para facilitar o consumo de um cliente. Agrupar o acesso ao banco em uma abstração para ser usado de acordo com as regras do domínio é um padrão bem conhecido e chamado de Repository. Tem esse nome, porque acessamos o repositório de dados, independente de qual seja.

A criação de repositorios de dados podem ser necessários para abstrair logicas de processamento dos dados, e até mesmo escrita de JPQL ou SQL caso esteja utilizando um banco de dados relacional. Visando maximizar a experiência do desenvolvedor na definição de consultas para obter determinado Objeto ou Coleção de Objeto, apartir de especificações ou metodos para criação, remoção, atualização que foi criado o Spring Data. 

Spring Data JPA oferece através de interface um conjunto de métodos referêntes a persistencia e consulta que podem ser estendidos para classes mapeadas como entidade. Metodos como consulta por chave primaria, remoção por chave primaria, consultar todos objetos e salvar estão pronto para uso.


## Explorando Spring Data Jpa e seus Repositorios

A unica premissa necessária para utilização dos Repositorios do Spring Data é que Entidade esteja mapaeada corretamente, ou seja, exista uma refêrencia a tabela e a sua chave primaria.  Vamos ao mapeamento;

```java
@Entity
public class Imovel {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private Integer quantidadeQuartos;
    private Integer quantidadeBanheiros;
    private Integer areaConstruidaEmMetrosQuadrados;
    private Integer anoDeConstrucao;
    private Boolean possuiGaragem;

    public Imovel(Integer quantidadeQuartos, Integer quantidadeBanheiros, Integer areaConstruidaEmMetrosQuadrados, Integer anoDeConstrucao, Boolean possuiGaragem) {
        this.quantidadeQuartos = quantidadeQuartos;
        this.quantidadeBanheiros = quantidadeBanheiros;
        this.areaConstruidaEmMetrosQuadrados = areaConstruidaEmMetrosQuadrados;
        this.anoDeConstrucao = anoDeConstrucao;
        this.possuiGaragem = possuiGaragem;
    }

    /**
     * @deprecated construtor para uso excluisivo do Hibernate
     */
    @Deprecated
    public Imovel() {
    }

}
```
 Com a classe imovel mapeada para um Entidade, o proximo passo é a criação do repositorio de imoveis. Começaremos criando uma interface, por convensão iremos nomea-la como ImovelRepository e iremos estende-la a interface JpaRepository.  Nota-se que esta interface requere dois parametros o primeiro é a entidade que iremos referenciar e o segundo é o tipo de sua chave primaria. Veja a implementação abaixo:


```java
    public interface ImovelRepository extends JpaRepository<Imovel,Long> {
    }
```

Apenas com essa implementação podemos utilizar diversos metodos, entre eles estão:

1. save (salvar): Este metodo é capaz de salvar um Objeto do tipo Imovel.
2. findById (Consulte pela chave primaria): este metodo retorna ou não  um registro de Imovel com a seguinte chave privaira.
3. findAll (Consultar todos): este metodo é capaz de retorna uma Lista de todos Imoveis cadastrados.
4. deleteById (remova pela chave primaria): este metodo remove o imovel pertencente a seguinte chave primaria.

Veja um exemplo de uso.

```java
@Component
public class ImovelResource {
    private final ImovelRepository repository;

    public ImovelResource(ImovelRepository repository) {
        this.repository = repository;
    }

    public void salvar(Imovel imovel){
        repository.save(imovel);
    }

    public Imovel buscarPeloId(Long id){
        
        Optional<Imovel> possivelImovel = repository.findById(id);
        
        return possivelImovel.orElseThrow(()-> new RuntimeException("Não existe cadastro para este id."));
                
    }

    public List<Imovel> buscarTodos(){
        return repository.findAll();
    }

    public void removerPeloId(Long id){
        repository.deleteById(id);
    }
}
```


