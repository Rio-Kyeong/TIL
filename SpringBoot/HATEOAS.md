# Implementing HATEOAS for RESTful Services
<pre>
<b>HATEOAS(Hypermedia As the Engine Of Application State)</b>
- 현재 리소스와 연관된(호출 가능한) 자원 상태 정보를 제공
- LINK에 사용 가능한 URL을 리소스로 전달하여 클라이언트가 참고하여 사용 할 수 있도록 한다.

<b>서버는 리소스를 보낼 때 리소스와 연관된 링크 정보를 담아 클라이언트에게 제공해야하며, 
클라이언트는 링크 정보를 바탕으로 리소스에 접근해야한다는 원칙이다.</b>

<b>RESTful API를 사용하는 클라이언트가 전적으로 서버와 동적인 상호작용이 가능하도록 하는데,
하나의 리소스에서 파생할 수 있는 여러가지 추가 작업의 정보들을 한번에 얻을 수 있다.</b>

장점
- 요청 URI가 변경되더라도 클라이언트에서 동적으로 생성된 URI를 사용함으로써,
  클라이언트가 URI 수정에 따른 코드를 변경하지 않아도 되는 편리함을 제공한다.
- URI 정보를 통해 들어오는 요청을 예측할 수 있게 된다.
- Resource가 포함된 URI를 보여주기 때문에, Resource에 대한 확신을 얻을 수 있다.
</pre>
## HATEOAS
### pom.xml
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```
### Controller
<pre>
이 예제에서는 개별 사용자 조회 시 전체 사용자 조회로 돌아갈 수 있는 all-users URL을 추가시켰다.
또, 예를 들어 전체 사용자 조회 시 삭제하기, 수정하기로 이동하기위한 링크작업들을 추가 시킬 수 있다.

<b>사용자에게 어떤 api정보를 추가로 사용할 수 있는지 알려줄 수 있다.</b>
</pre>
```java
import org.springframework.hateoas.EntityModel;
import org.springframework.hateoas.server.mvc.WebMvcLinkBuilder;

import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.linkTo;
import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.methodOn;

@RestController
public class UserController {

    //전체 사용자 조회
    @GetMapping(value = "/users")
    public List<User> retrieveAllUsers(){
        return service.findAll();
    }

    //개별 사용자 조회
    @GetMapping(value = "/users/{id}")
    public EntityModel<User> retrieveUser(@PathVariable int id){
        User user = service.findOne(id);

        if(user == null){
            throw new UserNotFoundException(String.format("ID[%s] not found",id));
        }

        // HATEOAS
        // "all-users", SERVER_PATH + "/users"
        EntityModel<User> model = EntityModel.of(user); //user 객체 값 지정
        
        //static method 사용하면 아래의 코드 처럼 길게 쓰지 않아도 됨 
        //WebMvcLinkBuilder linkTo = WebMvcLinkBuilder.linkTo(
        //WebMvcLinkBuilder.methodOn(this.getClass()).retrieveAllUsers());

        WebMvcLinkBuilder linkTo = linkTo(methodOn(this.getClass()).retrieveAllUsers());
        // retrieveAllUsers() 메소드와 "all-users" 이름 하이퍼미디어 작업
        model.add(linkTo.withRel("all-users"));

        return model;
    }
}
```
### 개별 사용자 조회결과(Body)
```json
// GET /users/1

{
    "id": 1,
    "name": "Kenneth",
    "joinDate": "2021-08-12T05:25:46.293+00:00",
    "password": "pass1",
    "ssn": "701010-1111111",
    "_links": {
        "all-users": {
            "href": "http://localhost/users"
        } //retrieveAllUsers()의 URL
    }
}
```
