# Implementing Validations for RESTful Services
```
JDK에 포함된 Validations API와 Hibernate Library에 포함된 Hibernate Validations을 사용하여 검증한다.
```
## pom.xml
```xml
<!-- validation-api는 생략 가능 -->
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>

<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.7.Final</version>
</dependency>
```
## HTTP POST Method를 통해서 사용자가 입력하는 값(CREATE VALUE)을 검증
### Validation 2.0 Annotation
```
@Valid : 대상 객체의 Validation 조건을 검사

@AssertFalse : 값이 거짓인지?
@AssertTrue : 값이 참인지?
@DecimalMax(value=최댓값,inclusive=지정값 포함 여부) : 지정 값 이하의 실수인지?
@DecimalMin(value=최솟값,inclusive=지정값 포함 여부) : 지정 값 이상의 실수인지?
@Digits(integer=허용 가능한 정수 자릿수,fraction=허용 가능한 소수점 이하 자릿수) : 정수 여부
@Max : 지정 값 이상인지?
@Min : 지정 값 이하인지?
@NotNull : Null이 아닌지?
@Null : Null인지?
@Pattern(regexp=정규표현식) : 정규표현식을 만족하는지?
@Future : 대상 날짜가 현재보다 미래일 경우에만 통과 가능
@Past : 대상 날짜가 현재보다 과거일 경우에만 통과 가능
@Size(min=,max=,message=) : 문자열 또는 배열등의 길이 만족 여부
```
### POJO Class
```java
import javax.validation.constraints.Past;
import javax.validation.constraints.Size;

@Data
@AllArgsConstructor
public class User {
    private Integer id;

    //최소 2글자, 검증에 실패 시 전달되는 메세지
    @Size(min = 2, message = "이름은 2글자 이상 입력해 주세요.")
    private String name;
    //대상 날짜가 현재보다 과거일 경우에만 통과 가능
    @Past
    private Date joinDate;
}
```
### Controller
<pre>
User 객체에 <b>@Valid Annotation</b>을 선언을 하여 사용자(Client)가 추가할 사용자의 정보를 입력하면 대상 객체(User)의 Validation 조건을 검사
검증에 실패할 경우 HTTP Status Code는 <b>400 Bad Request(Client의 잘 못된 요청)</b>가 발생한다.
</pre>
```java
/**
* 사용자 정보추가
* @param user
* @return
*/
@PostMapping(value = "/users")
public ResponseEntity<User> createUser(@Valid @RequestBody User user){
    User savedUser = service.save(user);

    URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(savedUser.getId())
            .toUri();

    return ResponseEntity.created(location).build();
}
```
## MethodArgumentNotValidException
```
사용자의 입력값에 문제(Validation Fail)가 생겨서 MethodArgumentNotValidException 가 발생할 때 사용하는 예외 메소드(handleMethodArgumentNotValid) 재정의
```
### AOP를 이용한 Exception Handling
```java
@RestController
@ControllerAdvice// 어떠한 컨트롤러 클래스가 실행된다 하더라도 이 클래스가 같이 실행된다.
public class CustomizedResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {

    @Override //상속받은 ResponseEntityExceptionHandler Class의 handleMethodArgumentNotValid Method를 재정의하여 사용
    protected ResponseEntity<Object> handleMethodArgumentNotValid
            (MethodArgumentNotValidException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {
            //ex : 발생한 예외의 객체, headers : Http 헤더 값, status : Http 상태, WebRequest : URI 에 대한 요청

        ExceptionResponse exceptionResponse =
                new ExceptionResponse(new Date(), "Validation Failed", ex.getBindingResult().toString());

        return new ResponseEntity<>(exceptionResponse, HttpStatus.BAD_REQUEST);
    }
}
```
### JOSN 출력
```json 
// 검증 실패
{
    "timestamp": "2021-08-09T05:56:43.645+00:00",
    "message": "Validation Failed",
    "details": "org.springframework.validation.BeanPropertyBindingResult: 1 errors\n
        Field error in object 'user' on field 'name': rejected value [N]; 
        codes [Size.user.name,Size.name,Size.java.lang.String,Size]; 
        arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [user.name,name]; 
        arguments []; 
        default message [name],2147483647,2]; 
        default message [이름은 2글자 이상 입력해 주세요.]"
}
```