# Status Code 제어
<pre>
클라이언트의 요청이 정상적으로 처리가 되면 200(성공)의 상태코드를 반환받게 된다.
(GET HTTP METHOD 를 이용하여 DB 에 존재하지 않는 데이터를 검색하여도 서버측의 오류가 아니기 때문에 200 OK 의 상태코드를 받는다) 

각각의 CRUD에 따라서 서버로부터 요청결과값에 적절한 상태코드를 반환시켜주는 것이 좋은 REST API 중 하나이다.

GET(조회) 
성공 : 200 OK 
조회할 자원이 존재하지 않을 때 : 404 Not Found

POST(생성) 
성공 : 201 Created
클라이언트의 요청이 허용되지 않는 메소드일 때(실패) : 405 Method Not Allowed

PUT(수정)
성공 : 201 Created
자원 수정 요청의 결과가 기존의 자원 내용과 동일하여 변경된 내용이 없을 때 : 204 No Content
수정하려는 자원이 없을 때 : 404 Not Found

DELETE(삭제) 
성공 : 204 No Content
자원이 존재하지 않을 때 : 404 Not Found

자세한 내용은 이곳을 <a href="https://sanghaklee.tistory.com/61">참조</a>
</pre>
## ResponseEntity를 사용한 컨트롤러단 상태코드 제어
<pre>
<b>ResponseEntity</b>는 <b>HtttHeaders headers</b>와 <b>T body</b>, <b>HttpStatus status</b>를 포함한 클래스이다.
<b>ResponseEntity</b>를 반환값으로 사용하여 응답내용과 상태코드를 제어해준다.
자세한 내용은 이곳을 <a href="https://blog.jiniworld.me/71">[참조]</a> <a href="https://linked2ev.github.io/gitlog/2019/12/29/springboot-5-ResponseEntity/">[참조]</a>
</pre>
### ServletUriComponentsBuilder
```
사용자로부터 요청(request)가 왔을 때 특정값을 포함한 uri를 전달해야 하는 상황이 발생할 수 있다.
이 때 ServletUriComponentsBuilder Class를 통해서 적잘한 URI를 만들고 요청한 사용자에게 특정값을 포함한 URI를 전달 할 수 있다.
(파일업로드 후 파일 다운로드 경로를 사용자에게 보내주고자 할 때 자주 사용된다)
```
### Controller
```
사용자를 추가하는 역할인 POST HTTP METHOD 는 200 OK 가 아닌 201 Created 상태코드를 전달 받는것이 좋다.
ServletUriComponentsBuilder Class 를 이용해서 Server 에서 반환시켜주려고 하는 데이터 값을 ResponseEntity 에 담아서 전달한다.
```
```java
/**
* 사용자 추가
* @param user
* @return
*/
@PostMapping(value = "/users")
public ResponseEntity<User> createUser(@RequestBody User user){
    User savedUser = service.save(user);

    URI uri = ServletUriComponentsBuilder.fromCurrentRequest() //현재 요청된 요청(request) URI값을 가져온다.
            .path("/{id}") //buildAndExpand를 통해 얻은 id 값이 들어온다.
            .buildAndExpand(savedUser.getId()) //가변 변수{id}에 새롭게 만들어진 savedUser.getId() 값을 설정시킨다.
            .toUri(); //URI 형태로 변경(URI 생성)

    return ResponseEntity.created(uri).build();
    //요청 후 Response Headers의 location 부분을 보면 controller에서 만든 uri가 전달된것을 확인 할 수 있다.
    //사용자를 추가하면서 200 OK가 아닌 201 Created 상태코드를 전달 받게 된다.
}
```
## 사용자 지정 Exception Handling
<pre>
직접 Exception Class를 정의하여 예외처리를 해준다.
<b>@ResponseStatus</b> : 원하는 상태코드를 반환시켜준다.
</pre>
### 직접 정의한 예외처리 클래스
```java
// HTTP Status Code
// 2XX -> OK
// 4XX -> Client
// 5XX -> Server
@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```
### Controller
```java
/**
* 개별 사용자 조회
* GET /users/1 or /users/10 -> String 으로 전달되지만 자동으로 Mapping 되어 int 로 변환된다.
* @param id
* @return
*/
@GetMapping(value = "/users/{id}")
public User retrieveUser(@PathVariable int id){
    User user = service.findOne(id);

    if(user == null){//user id가 null 일 경우 직접 정의한 UserNotFonundException 예외발생처리
        throw new UserNotFoundException(String.format("ID[%s] not found",id));
    }
    return user;
}
```

## AOP를 이용한 Exception Handling
<pre>
클래스에 <b>@ControllerAdvice</b> 와 <b>@RestController</b> 어노테이션을 추가함으로써 REST 응답을 리턴하게 된다.

<b>ControllerAdvice</b>를 사용하여 Exception 처리를 한 곳으로 모으는 경우,
<b>ResponseEntityExceptionHandler</b>를 상속받도록 하여 Spring MVC에서 기본으로 제공되는 Exception들의 처리를 간단하게 등록할 수 있다.
각 Exception 처리를 위한 메서드들은 모두 protected로 선언되어 있으며, 하위 클래스에서 필요에 따라 Override(재정의)할 수 있다.

<b>@ControllerAdvice</b> : 모든 @Controller 즉, 전역에서 발생할 수 있는 예외를 잡아 처리해주는 annotation
<b>@ExceptionHandler</b> : annotation 을 사용하여 예외를 처리할 클래스를 정의한다.
</pre>
### 예외처리를 위한 POJO Class
```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Date;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class ExceptionResponse {
    private Date timestamp; //예외발생 시간
    private String message; //예외발생 메시지
    private String details; //예외발생 상세정보
}
```
### AOP를 이용한 예외처리 클래스
```java
@RestController
@ControllerAdvice// 어떠한 컨트롤러 클래스가 실행된다 하더라도 이 클래스가 같이 실행된다.
public class CustomizedResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {

    /**
     * 모든 예외처리를 해주는 Method
     * @param ex
     * @param request
     * @return
     */
    @ExceptionHandler(Exception.class)// Exception 이 발생되면 이 메소드가 실행된다.
    public final ResponseEntity<ExceptionResponse> handleAllExceptions(Exception ex, WebRequest request){
        //WebRequest : URI(Uniform Resource Identifier)에 대한 요청을 만든다. 이 클래스는 abstract 클래스이다.
        //request.getDescription(false) : 반환 -> "uri=/users/12"
        //((ServletWebRequest)request).getRequest().getRequestURI().toString() : 반환 -> "/users/12"
        
        ExceptionResponse exceptionResponse =
            new ExceptionResponse(new Date(), ex.getMessage(), request.getDescription(false));
            
        return new ResponseEntity<>(exceptionResponse, HttpStatus.INTERNAL_SERVER_ERROR);
    }

    /**
     * UserNotFoundException 발생시 예외처리를 해주는 Method
     * @param ex
     * @param request
     * @return
     */
    @ExceptionHandler(UserNotFoundException.class)// UserNotFoundException 이 발생되면 이 메소드가 실행된다.
    public final ResponseEntity<ExceptionResponse> handleUserNotExceptions(Exception ex, WebRequest request){

        ExceptionResponse exceptionResponse =
            new ExceptionResponse(new Date(), ex.getMessage(), ((ServletWebRequest)request).getRequest().getRequestURI().toString());
            
        return new ResponseEntity<>(exceptionResponse, HttpStatus.NOT_FOUND);
    }
}
```
