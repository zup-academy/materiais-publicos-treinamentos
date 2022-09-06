# Utilizando Avro

## O que é ?
Avro é um formato de dados. A Apache chama de um sistema de serealização de dados.
Além disso, é uma especificação para formato de dados mantida pela Apache.
Com muitas utilidades, sua principal e mais utilizada com Apache Kafka é a Schema Resolution.

### Schemas
O Avro depende de esquemas. Quando os dados Avro são lidos, o esquema usado ao gravá-los está sempre presente. 
Isso permite que cada dado seja escrito sem sobrecarga por valor, tornando a serialização rápida e pequena.
Isso também facilita o uso com linguagens de script dinâmicas, pois os dados, juntamente com seu esquema, são totalmente autodescritivos.
Os esquemas Avro são definidos em JSON. Isso facilita a implementação em linguagens que já possuem bibliotecas JSON.
Schemas são compostos de tipos primitivos (null, boolean, int, long, float, double, bytes, e String) e tipos complexos (record, enum, array, map, etc). Os tipos complexos devem ser mapeados antes da serialização.

### Tipos de Dados

Atualmente quando utilizamos Kafka o mais comum é a utilização de 3 tipos de dados, Json, Protobuf e Avro.
Vamos entender as vantagens e desvantagens de cada um:

#### JSON

**Prós:**
* Podemos criar estruturas complexas (matrizes ou elementos aninhados);
* É o formato mais aceita na Internet atualmente;
* Interoperabilidade com quase todas as linguagens (até Cobol);
* Legível por humanos;

**Contras:**
* Não tem suporte nativo a schemas;
* Pode se tornar enorme devido a repetição chaves ao criar objetos complexos;
* Falta de suporte a documentação e metadados;

#### Protobuf

**Prós:**
* Rápido devido trabalhar com binários;
* Interoperabilidade com quase todas as linguagens;
* Suporte para streaming (tanto parâmetro de método quanto como resposta);
* Popularmente utilizada para integração sistêmica;
* Possui suporte a geração de classes a partir de um “contrato” (arquivo .proto);
* Facilita a evolução dos contratos;

**Contras:**
* Curva de aprendizado;
* Não legível por humanos (ferramentas auxiliares ajudam o consumo de APIs, por exemplo);

#### AVRO

**Prós:**
* Forte apoio da da comunidade de Data e da Confluente para o uso em conjunto do Schema Registry;
* Utiliza a estrutura JSON para definir os tipos de dados;
* Permite a criação de objetos complexos;
* Multi-schema e documentação embarcada;
* Oferecer suporte a tipos primitivos, lógicos e objetos(records)
* É flexível e restrito em certo ponto, o Avro não permite que dados sem schema sejam objeto válidos;
* Curva de aprendizagem baixa

**Contra**
* O plugin utilizado na Java para converter os schemas Avro em Classes Java nem sempre utiliza classes adequadas

## Por que usar?

Com o uso desse tipo de dado o volume de bytes trafegado pela rede será bem menor, se compararmos com JSON a diferença será grande.
Usar Avro é uma forma poderosa de trabalhar com evolução dos esquemas sem dores-de-cabeça e utilizando Schema Registry torna o trabalho muito mais fácil.


## Como usar?

Vamos implementar a dependência no projeto :

```xml
<dependency>
  <groupId>org.apache.avro</groupId>
  <artifactId>avro</artifactId>
  <version>1.11.1</version>
</dependency>
```

Além disso vamos adicionar o Plugin, que irá gerar as classes Java através do schema:

```xml
<plugin>
  <groupId>org.apache.avro</groupId>
  <artifactId>avro-maven-plugin</artifactId>
  <version>1.11.1</version>
  <executions>
    <execution>
      <phase>generate-sources</phase>
      <goals>
        <goal>schema</goal>
      </goals>
      <configuration>
        <sourceDirectory>${project.basedir}/src/main/avro/</sourceDirectory>
        <outputDirectory>${project.basedir}/src/main/java/</outputDirectory>
      </configuration>
    </execution>
  </executions>
</plugin>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <configuration>
    <source>1.8</source>
    <target>1.8</target>
  </configuration>
</plugin>
```

## Schema Avro

Criamos um arquivo onde estará nosso Schema Avro, esse arquivo deve atender alguns parametros da especificação e deve ter sua extensão como .avsc:

```json
{"namespace": "example.avro",
 "type": "record",
 "name": "User",
 "fields": [
     {"name": "name", "type": "string"},
     {"name": "favorite_number",  "type": ["int", "null"]},
     {"name": "favorite_color", "type": ["string", "null"]}
 ]
}
```
No exemplo acima seguimos as seguintes regras:

- A declaração esta em json
- Alguns campos podem ser tipos primitivos que podem ser utilizados os tipos abaixos:
  - null : nenhum valor 
  - boolean : um valor binário 
  - int : inteiro com sinal de 32 bits 
  - long : inteiro com sinal de 64 bits 
  - float : número de ponto flutuante IEEE 754 de precisão simples (32 bits)
  - double : número de ponto flutuante IEEE 754 de precisão dupla (64 bits)
  - bytes : sequência de bytes sem sinal de 8 bits 
  - string : sequência de caracteres unicode
- Alguns campos podem ser tipos complexos , sendo os tipos listados abaixo:
  - records , enums , arrays , maps , unions e fixed
    - Records:
      - name : uma string JSON fornecendo o nome do registro (obrigatório).
      - namespace , uma string JSON que qualifica o nome (opcional);
      - doc : uma string JSON que fornece documentação ao usuário deste esquema (opcional).
      - aliases : uma matriz JSON de strings, fornecendo nomes alternativos para este registro (opcional).
      - fields : uma matriz JSON, listando campos (obrigatório). Cada campo é um objeto JSON com os seguintes atributos:
      - name : uma string JSON fornecendo o nome do campo (obrigatório) e 
      - doc : uma string JSON que descreve este campo para usuários (opcional).
      - type : um esquema , conforme definido acima
      - default : Um valor padrão para este campo, usado apenas ao ler instâncias que não possuem o campo para fins de evolução do esquema. A presença de um valor padrão não torna o campo opcional no momento da codificação. Os valores permitidos dependem do tipo de esquema do campo, conforme tabela abaixo. Os valores padrão para campos de união correspondem ao primeiro esquema na união. Os valores padrão para bytes e campos fixos são strings JSON, em que os pontos de código Unicode de 0 a 255 são mapeados para valores de bytes de 8 bits não assinados de 0 a 255. O Avro codifica um campo mesmo que seu valor seja igual ao padrão.

        | avro type       | 	json type       |  example  |
        |------------------|----------------|-----------|
        | null            | null             |null       |
        | boolean         | boolean          |true|
        | int,long | integer          |1|
        | float,double | number           |1.1|
        | bytes | string           |"\u00FF"|
        | string | string           |"foo"|
        | record | object |{"a": 1}|
        | enum | string |"FOO"|
        | array | array |[1]|
        | map | object |{"a": 1}|
        | fixed | string |"\u00ff"|

      - order : especifica como este campo afeta a ordenação deste registro (opcional). Os valores válidos são “ascendente” (o padrão), “descendente” ou “ignorar”. Para obter mais detalhes sobre como isso é usado, consulte a seção de ordem de classificação abaixo. 
      - aliases : uma matriz JSON de strings, fornecendo nomes alternativos para este campo (opcional).

    - ENUM
      - name : uma string JSON fornecendo o nome da enumeração (obrigatório). 
      - namespace , uma string JSON que qualifica o nome (opcional); 
      - aliases : uma matriz JSON de strings, fornecendo nomes alternativos para este enum (opcional). 
      - doc : uma string JSON que fornece documentação ao usuário deste esquema (opcional). 
      - símbolos : uma matriz JSON, listando símbolos, como strings JSON (obrigatório). Todos os símbolos em uma enumeração devem ser exclusivos; duplicatas são proibidas. Cada símbolo deve corresponder à expressão regular [A-Za-z_][A-Za-z0-9_]* (o mesmo requisito para nomes ). 
      - default : um valor padrão para essa enumeração, usado durante a resolução quando o leitor encontra um símbolo do gravador que não está definido no esquema do leitor (opcional). O valor fornecido aqui deve ser uma string JSON que seja membro da matriz de símbolos. Consulte a documentação sobre resolução de esquema para saber como isso é usado.

    - Array
      - items : o esquema dos itens do array.
    - Map
      - values : o esquema dos valores do mapa.
		
Todos os detalhes do que é aceito ou não esta na [especificação do Avro.](https://avro.apache.org/docs/1.11.1/specification/)		
		
		
		


## Referências
https://avro.apache.org/docs/1.11.1/
https://medium.com/@st4rl0rd/apache-kafka-trabalhando-com-convers%C3%B5es-do-avro-schema-2aa11b382af8
