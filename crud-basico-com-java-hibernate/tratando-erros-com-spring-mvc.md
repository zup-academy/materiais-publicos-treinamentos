# Tratamento de Exceção na camada Web (Controller)

Quando construimos APIs esperamos que parceiros internos ou externos venham a se integrar com elas, para isso precisamos que nossas APIs gerem o maior valor a estes, tantos nos cenarios de sucesso quanto aos de erro. Ter um mecanismo de tratamento de erro bem definido faz com que nossos parceiros gastem menor tempo identificando o que precisa ser corrigido garantindo que o processo de consumo seja mais assertivo.

Um bom modelo para exibição de erro é dado pelo Campo que foi validado e a mensagem referente a qual validação falhou. Observe um exemplo abaixo.

```json
    {
        "mensagem":[
            "Campo nome não deve ser preenchido em branco.",
            "Campo email deve estar bem formatado.",
            "Campo peso deve ser maior que zero.",
            "Campo peso deve ser no minimo 30."
        ]
    }
```

## Criando no Exception Handler com Spring

O primeiro passo para definir um tratador de execessões é definir a estrutura que serão apresentadas as mensagens, para isso criaremos uma classes que agrupe diversas Strings. 

```java
public class ErroPadronizado {
    private final Collection<String> mensagens= new ArrayList<>();

    public void adicionarErro(String campo, String mensagem){

        String mensagemDeErro=String.format("Campo %s %$s.",campo,mensagem);

        mensagens.add(mensagemDeErro);
    }

    public void adicionarErro(FieldError fieldError){

        String mensagemDeErro=String.format("Campo %s %$s.",fieldError.getField(),fieldError.getDefaultMessage());

        mensagens.add(mensagemDeErro);
    }

    public Collection<String> getMensagens() {
        return mensagens;
    }
}

```

Agora precisamos definir uma classe que ficará responsavel por captar as exceptions lançadas nas classes anotadas com @RestController ou @Controller. 

```java
@RestControllerAdvice
public class NossoControllerAdvice {
    
    
}
```

- @RestControllerAdvice é responsavel por tornar a classe um Bean especializado em executar varreduras nos Controllers e capturar as excessões que forem lançadas. Para que aconteça um tratamento especificado é necessário que seja escrito um metodo  e seja anotado pela anotação @ExceptionHandler, e que esta anotação receba o tipo da exceção que sera tratada. 

### Tratando excessões da Bean Validation

Quando uma validação da Bean Validation falha é lançada uma MethodArgumentNotValidException, precisamos captura-la para realizar o tratamento da mesma. Veja abaixo como é definido o codigo.

```java
@RestControllerAdvice
public class NossoControllerAdvice {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErroPadronizado> methodArgumentNotValid(MethodArgumentNotValidException ex){
        ErroPadronizado erros= new ErroPadronizado();

        BindingResult bindingResult = ex.getBindingResult();

        List<FieldError> fieldErrors = bindingResult.getFieldErrors();

        for (FieldError fieldError : fieldErrors) {

            erros.adicionarErro(fieldError);
        }

        return ResponseEntity.badRequest().body(erros);
    }

}
```

- @ExceptionHandler: é responavel por indicar qual excessão desejamos capturar e tratar.


Perceba que é solicitado a classe BindingResult todos os FieldErros, ou seja, todos os erros que são disparados a nivel de campos são retornados a uma lista de fieldErros onde cada um é adicionado a nossa lista de mensagem personalizada. Onde no final é devolvido junto a um HTTP Status 400. 

Utilizando a API de Stream conseguimos escrever este método um pouco mais legivel, veja abaixo.


```java
@RestControllerAdvice
public class NossoControllerAdvice {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErroPadronizado> methodArgumentNotValid(MethodArgumentNotValidException ex) {
        ErroPadronizado erros = new ErroPadronizado();

        BindingResult bindingResult = ex.getBindingResult();

        bindingResult.getFieldErrors()
                .forEach(erros::adicionarErro);


        return ResponseEntity.badRequest().body(erros);
    }

}
```

## Links para aprofundamento

- [Definindo Status HTTP para Exceções personalizadas](https://www.baeldung.com/spring-response-status)
- [Tratamento de execeções com Spring](https://www.baeldung.com/exception-handling-for-rest-with-spring)
- [Guia para Error Handler com Spring Boot](https://reflectoring.io/spring-boot-exception-handling/)
