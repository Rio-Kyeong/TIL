# Configuring Auto Generation of Swagger Documentation
<pre>
<b>Spring REST API Documentation</b> - 개발자 도움말 페이지 생성

Swagger의 주된 목적은 RESTful API를 문서화시키고 관리하는 것이다.
API 문서를 일반 Document로 작성하면 API 변경 시마다 문서를 수정해야 하는 불편함이 있는데,
Swagger 같은 Framework를 이용하면 이를 자동화할 수 있다. (Spring boot 사용 시)

자세한 내용은 이 곳을 <a href="https://otrodevym.tistory.com/entry/spring-boot-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-4-Swagger-%EC%84%A4%EC%A0%95-%EB%B0%8F-%EC%82%AC%EC%9A%A9-%EB%B0%A9%EB%B2%95">참조</a>
</pre>
## Swagger
### pom.xml
```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```
### Configuration Class 생성
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {
    // Swagger Documentation Customizing
    // static final로 생성하는 이유 : 한 번 생성이되면 변경이 되지않는 정보들이기 때문에

    // 사용자 정보를 나타내는 컨택트 객체
    private static final Contact DEFAULT_CONTACT =  new Contact("Kenneth Lee",
            "http://www.joneconsulting.co.kr", "edwon@joneconsulting.co.kr");

    // API 정보
    private static final ApiInfo DEFAULT_API_INFO = new ApiInfo("Awesome API Title",
            "My User management REST API service", "1.0", "urn:tos",
            DEFAULT_CONTACT, "Apache 2.0",
            "http://www.apache.org/licenses/LICENSE-2.0", new ArrayList<>());

    // 문서 타입 지정
    // asList() : 배열형태의 데이터 값을 리스트로 바꿔주는 메서드
    private static final Set<String> DEFAULT_PRODUCES_AND_CONSUMES = new HashSet<>(
            Arrays.asList("application/json", "application/xml")); 

    // REST API의 경우는 보통 version 별로 관리한다.
    // Docket을 버전 별로 여러개 만들어서 테스트도 가능하다.
    @Bean
    public Docket api(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(DEFAULT_API_INFO)
                .produces(DEFAULT_PRODUCES_AND_CONSUMES)
                .consumes(DEFAULT_PRODUCES_AND_CONSUMES);
    }
}
```
### Postman 조회 
<pre>
// http://localhost/v2/api-docs

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBoot/img/api-docs.PNG"/>
</pre>
### User(Domain)
<pre>
Swagger가 적용 될 Model에 대한 정보 추가하기

@ApiModel : Model에 대한 설명 추가(모델명)
@ApiModelProperty : Model 속성에 대한 설명 추가
</pre>
```java
@ApiModel(description = "사용자 상세 정보를 위한 도메인 객체")
public class User {
    private Integer id;

    @Size(min = 2, message = "이름은 2글자 이상 입력해 주세요.")
    @ApiModelProperty(notes = "사용자 이름을 입력해 주세요.")
    private String name;
    @Past
    @ApiModelProperty(notes = "사용자 등록일을 입력해 주세요.")
    private Date joinDate;

    
    @ApiModelProperty(notes = "사용자 비밀번호를 입력해 주세요.")
    private String password;
    
    @ApiModelProperty(notes = "사용자 주민번호를 입력해 주세요.")
    private String ssn;
}
```
### Postman 조회
```json
// http://localhost/v2/api-docs

"User": {
    "type": "object",
    "properties": {
        "id": {
            "type": "integer",
            "format": "int32"
        },
        "joinDate": {
            "type": "string",
            "format": "date-time",
            "description": "사용자 등록일을 입력해 주세요."
        },
        "name": {
            "type": "string",
            "description": "사용자 이름을 입력해 주세요.",
            "minLength": 2,
            "maxLength": 2147483647
        },
        "password": {
            "type": "string",
            "description": "사용자 비밀번호를 입력해 주세요."
        },
        "ssn": {
            "type": "string",
            "description": "사용자 주민번호를 입력해 주세요."
        }
    },
    "title": "User",
    "description": "사용자 상세 정보를 위한 도메인 객체"
},
```
### Controller
```
Swagger가 적용 될 Controller에 대한 정보 추가하기

@Api : Controller에 대한 설명 추가(컨트롤러명)
@ApiOperation : 메서드에 대한 설명 추가
@ApiParam : 파라미터에 대한 설명 추가
```
```java
@RestController
@Api("User Controller API V1")
public class UserController {

    //개별 사용자 조회
    @ApiOperation(value = "회원 목록", notes = "개별 회원 목록을 반환한다.")
    @GetMapping(value = "/users/{id}")
    public EntityModel<User> retrieveUser(@ApiParam(value = "회원의 아이디", required = true) @PathVariable int id){
        User user = service.findOne(id);

        if(user == null){//user id가 null 일 경우
            throw new UserNotFoundException(String.format("ID[%s] not found",id));
        }

        // HATEOAS
        // "all-users", SERVER_PATH + "/users"
        EntityModel<User> model = EntityModel.of(user); //user 객체 값 지정
        WebMvcLinkBuilder linkTo = linkTo(methodOn(this.getClass()).retrieveAllUsers());
        // retrieveAllUsers() 메소드와 "all-users" 이름 하이퍼미디어 작업
        model.add(linkTo.withRel("all-users"));

        return model;
    }
}
```
