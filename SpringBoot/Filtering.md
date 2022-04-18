# 응답 데이터 제어를 위한 Filtering
```
중요한 데이터(ex 비밀번호, 주민번호)를 클라이언트(사용자)에게 바로 노출 시 보안상의 문제가 생길 수 있기 때문에 Filtering을 해주어야 한다.
```
## Jackson Annotation - Filtering
### @JsonIgnore
`필드 레벨`에 적용되며, 해당 데이터는 'Ignore'되어서 응답값이 보이지 않는다.
```java
//UserDomain.java
import com.fasterxml.jackson.annotation.JsonIgnore;

@Data 
@AllArgsConstructor 
public class User { 
    private Integer id; 
    
    @Size(min=2, message = "Name은 2글자 이상 입력해주세요.") 
    private String name; 
    @Past 
    private Date joinDate; 

    // 중요한 데이터(응답 데이터에 password와 ssn은 보이지 않는다.)
    @JsonIgnore 
    private String password; 
    @JsonIgnore 
    private String ssn; 
}
```
### @JsonIgnoreProperties
`클래스 레벨`에 적용되며, 해당 데이터는 'Ignore'되어서 응답값이 보이지 않는다.
```java
//UserDomain.java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@Data 
@AllArgsConstructor 
// 중요한 데이터(응답 데이터에 password는 보이지 않으며, 배열 형식으로 여러 개를 선언할 수 있다)
@JsonIgnoreProperties(value = {"password"})
public class User { 
    private Integer id; 
    
    @Size(min=2, message = "Name은 2글자 이상 입력해주세요.") 
    private String name; 
    @Past 
    private Date joinDate; 

    private String password; 
    private String ssn; 
}
```
## 프로그래밍으로 제어하는 Filtering
### @JsonFilter
`클래스 레벨`에 적용되며, 어노테이션으로 JSON 변환시 사용할 필터를 명시
```java
//UserDomain.java
import com.fasterxml.jackson.annotation.JsonFilter;

@Data
@AllArgsConstructor

// Controller, Service 클래스에서 사용한다.
// Filter ID를 문자열로 지정하며, 이 어노테이션 사용시에는 무조건 FilterProvider 와 해당 ID를 처리하는 필터를 제공해야한다.
@JsonFilter("UserInfo")
public class User {
    private Integer id;

    @Size(min = 2, message = "이름은 2글자 이상 입력해 주세요.")
    private String name;
    @Past
    private Date joinDate;

    //@JsonIgnore
    private String password;
    //@JsonIgnore
    private String ssn;
}
```
### Controller
`@JsonFilter()` 이용하여 admin페이지에서 개별/전체 사용자 조회
```java
@RestController
@RequestMapping("/admin")
public class AdminUserController {

    // 개별 사용자 조회
    @GetMapping(value = "/users/{id}") // /admin/users/{id}
    public MappingJacksonValue retrieveUser(@PathVariable int id){
        User user = service.findOne(id);

        if(user == null){
            throw new UserNotFoundException(String.format("ID[%s] not found",id));
        }

        // SimpleBeanPropertyFilter : 지정된 필드들만 JSON 변환하고, 알 수 없는 필드는 무시한다.
        SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                .filterOutAllExcept("id", "name", "password", "ssn"); //포함 시키자 하는 필드변수 선언

        // SimpleBeanPropertyFilter를 사용할 수 있는 필터형태로 변경 
        // SimpleFilterProvider().addFilter("Bean filter명", filter); 
        FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfo", filter); 

        // SimpleBeanPropertyFilter를 반환할 수 있는 MappingJacksonValue를 반환 값으로 선언
        MappingJacksonValue mapping = new MappingJacksonValue(user);
        mapping.setFilters(filters);

        return mapping;
    }

    // 전체 사용자 조회
    @GetMapping(value = "/users") // /admin/users
    public MappingJacksonValue retrieveAllUsers(){
        List<User> users = service.findAll();

        SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                .filterOutAllExcept("id", "name", "joinDate", "password"); //포함 시키자 하는 필드변수 선언

        FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfo", filter);

        MappingJacksonValue mapping = new MappingJacksonValue(users);
        mapping.setFilters(filters);

        return mapping;
    }
}
```

## 응답 데이터 형식 변환 - XML format
### pom.xml에 maven 라이브러리 추가
```xml
<dependency> 
    <groupId>com.fasterxml.jackson.dataformat</groupId> 
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.10.2</version>
</dependency>
```
