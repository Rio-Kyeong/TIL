# RestController
<pre>
<b>@RestController</b>는 @Controller에 @ResponseBody가 결합된 어노테이션이다.

Spring MVC <b>@Controller</b>와 RESTful 컨트롤러인 <b>@RestController</b>의 차이점은 HTTP Response Body가 생성되는 방식이다.

<b>@Controller 는 View Page를 반환</b>하지만, <b>@RestController는 객체(VO,DTO)를 반환하기만 하면, 
객체데이터는 application/json 형식의 HTTP ResponseBody에 직접 작성</b>되게 된다.
</pre>
<pre>
View를 가지 않는 REST Data(JSON/XML)를 반환한다.

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBoot/img/RestController.PNG"/>
</pre>
```java 
@RestController
public class HelloWorldController {

    // @GetMapping : GET 요청을 하는 API의 어노테이션이며 데이터를 가져올 때 사용한다.
    // /hello-world (endpoint)
    // @RequestMapping(method=RequestMethod.GET, path ="/hello-world")
    @GetMapping(value ="/hello-world")
    public String helloWorld(){
        return "Hello World";
    }

    // HelloWorldBean Bean(VO)을 반환한다.        
    // JSON 형식으로 출력 "message": "Hello World"
    @GetMapping(value ="/hello-world-bean")
    public HelloWorldBean helloWorldBean(){
        return new HelloWorldBean("Hello World");
    }

    // Path Variable
    @GetMapping(value ="/hello-world-bean/path-variable/{name}")
    public HelloWorldBean helloWorldBean(@PathVariable String name){
        return new HelloWorldBean(String.format("Hello World, %s", name));
    }
}
```
## Lombok을 이용해 객체(VO,DTO) 만들기
```
Lombok는 자바에서 작성해야 하는 boilerplate code(getter/setter, constructor, toString)를 
어노테이션을 통해서 자동으로 생성해주는 라이브러리입니다. 

@NonNull : 메서드나 생성자 인자에 @NunNull 어노테이션을 추가하면 Lombok가 null 체크 구문을 생성
@Getter, @Setter : 클래스 필드에 대한 getter와 setter 메서드를 생성
@ToString : 클래스의 toString 메서드를 자동으로 생성
@EqualsAndHashCode : equals()와 hashCode()를 자동 생성
@NoArgsConstructor : 인자 없는 생성자 자동 생성
@AllArgsConstructor : 모든 필드를 인자로 받는 생성자 생성
@RequiredArgsContructor : RequiredArgsContructor(staticName=“of") : static factory 메서드를 생성
@Data : @Data 어노테이션은 아래 모든 어노테이션이 적용되는 어노테이션
 - @ToString
 - @EqualsAndHashCode
 - @Getter, @Setter
 - @RequiredArgsConstructor
```
```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data //@ToString, @EqualsAndHashCode, @Getter, @Setter, @RequiredArgsConstructor
@AllArgsConstructor //모든 필드를 인자로 받는 생성자
@NoArgsConstructor //인자 없는 생성자
public class HelloWorldBean {
    
    private String message;
    
    // @AllArgsConstructor 어노테이션을 통해서 자동생성(둘 다 선언 시 error)
    // public HelloWorldBean(String message){
    //    this.message = message;
    // }  
}
```
<pre>
Lombok을 이용함으로써 메서드를 정의하지 않아도 어노테이션에 해당하는 메서드가 만들어진 것을 볼 수 있다.

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBoot/img/Structure.PNG"/>
</pre>
## Path Variable(@PathVariable)
```
@PathVariable 파라메터를 사용하면 아래와 같이 URI의 일부를 변수로 전달할 수 있다.

- 사용법
    @RequestMapping 어노테이션 값으로 {템플릿변수} 를 사용하고
    @PathVariable 어노테이션을 이용해서 {템플릿 변수} 와 동일한 이름을 갖는 파라미터를 추가하면 된다.

- 주의점
    null이나 공백값이 들어가는 parameter라면 적용하지 말것
    @PathVariable 사용하여 값을 넘겨받을때 값에 .가 포함되어 있으면 .포함하여 그뒤가 잘려서 들어온다는 것
```
```java
// Path Variable
@GetMapping(value ="/hello-world-bean/path-variable/{name}")
public HelloWorldBean helloWorldBean(@PathVariable("name") String name){
    return new HelloWorldBean(String.format("Hello World, %s", name));
}
```
