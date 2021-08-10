# Versioning - Basic Approach with URIs
```
우리가 제공하려는 REST API는 자원의 정보뿐만 아니라 현재 API 버전이 어떤 것인지 같이 표시해 주는 게 좋다.

버전 관리라는 것은 단순히 사용자에게 보여지는 화면만을 제어하는 것이 아니라
REST API의 설계가 변경되거나, application의 구조가 바뀔 때에도 버전을 변경해서 사용해야한다.

또한 사용자에게도 어떠한 API의 버전을 사용하는지 적절히 명시해주어야한다.
```
```
Versioning

1) URI Versioning - 일반 브라우저에서 실행 가능(URI 변경)
    - Twitter

2) Request Parameter Versioning - 일반 브라우저에서 실행 가능(Params 추가)
    - Amazon

3) (Custom) Headers Versioning - 일반 브라우저에서 실행 불가(Headers 추가)
    - Microsoft

4) Media Type Versioning(a.k.a "content negotiation" or "accept header") - 일반 브라우저에서 실행 불가(파일 변환)
    - GitHub
```
```
Factors

 - 과도한 URI값은 지양한다
 - 잘못된 헤더 값 사용주의
 - 브라우저의 캐쉬기능으로 인해서 우리가 지정한 값이 반영되지 않을 수 있다(캐쉬 삭제를 한다)
 - 웹 브라우저에서 요청을 실행할 수 있어야 한다(URI, Parmas)
 - 개발하고 제공하는 REST API에는 적절한 도움 문서가 있어야 한다
```
## URI에 버전을 포함시켜 API 버전을 관리하는 방법
```
개별 사용자를 조회하는 HTTP GET METHOD 에서 조회 정보가 다른 V1버전과 V2버전(새로운 조회 정보 추가)을 만드려고 한다.
URI를 통해 사용자에게 버전을 표시해준다.
```
### 새로운 V2 Version VO
```java
// UserV2 (새로운 조회 정보 추가)

@Data 
@AllArgsConstructor 
@NoArgsConstructor 
@JsonFilter("UserInfoV2") 
public class UserV2 extends User { 
    private String grade; // 회원 등급 
}
```
### Contoroller
```java
import org.springframework.beans.BeanUtils;

@RestController
@RequestMapping("/admin")
public class AdminUserController {

    // 개별 사용자 조회
    // GET방식 /admin/users/1 -> /admin/v1/users/1
    // User 의 "id", "name", "password", "ssn" 를 조회하는 V1 version
    @GetMapping(value = "/v1/users/{id}")
    public MappingJacksonValue retrieveUserV1(@PathVariable int id){
        User user = service.findOne(id);

        if(user == null){//user id가 null 일 경우
            throw new UserNotFoundException(String.format("ID[%s] not found",id));
        }

        SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                .filterOutAllExcept("id", "name", "password", "ssn");

        FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfo", filter);

        MappingJacksonValue mapping = new MappingJacksonValue(user);
        mapping.setFilters(filters);

        return mapping;
    }

    // GET방식 /admin/users/1 -> /admin/v2/users/1
    // User 와 새롭게 추가된 UserV2의 "id", "name", "joinDate", "grade" 를 조회하는 V2 version
    @GetMapping(value = "/v2/users/{id}")
    public MappingJacksonValue retrieveUserV2(@PathVariable int id){
        User user = service.findOne(id);

        if(user == null){//user id가 null 일 경우
            throw new UserNotFoundException(String.format("ID[%s] not found",id));
        }

        // User -> UserV2
        UserV2 userV2 = new UserV2();// UserV2 객체생성

        // BeanUtils : springframework 에서 제공해주는 util Class, 객체를 쉽고 간결하게 복사할 수 있다.
        // BeanUtils.copyProperties(복사 할 원본 객체, 복사 객체);
        BeanUtils.copyProperties(user, userV2);
        userV2.setGrade("VIP");

        SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                .filterOutAllExcept("id", "name", "joinDate", "grade");

        FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfoV2", filter);

        MappingJacksonValue mapping = new MappingJacksonValue(userV2);
        mapping.setFilters(filters);

        return mapping;
    }
}
```
## Request Parameter를 이용한 버전 관리 방법
```java
// params 값이 추가되어야 하기 때문에 URI 맨 마지막에 슬러시(/)를 추가해준다.
@GetMapping(value = "/users/{id}/", params = "version=1")
public MappingJacksonValue retrieveUserV1(@PathVariable int id){ ... }

@GetMapping(value = "/users/{id}/", params = "version=2")
public MappingJacksonValue retrieveUserV2(@PathVariable int id){ ... }
```
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBoot/img/params.PNG">
</pre>
## header 값을 이용해 버전을 관리하는 방법
```java
//headers= "헤더 값"
@GetMapping(value = "/users/{id}", headers="X-API-VERSION=1")
public MappingJacksonValue retrieveUserV1(@PathVariable int id){ ... }

@GetMapping(value = "/users/{id}", headers="X-API-VERSION=2")
public MappingJacksonValue retrieveUserV2(@PathVariable int id){ ... }
```
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBoot/img/headers.PNG">
</pre>
## Mime Type을 이용해 버전을 관리하는 방법
```
- Multipurpose Internet Mail Extensions : 간단히 말해 파일 변환
- 인코딩 : 바이너리 파일에서 텍스트 파일로 변환
- 디코딩 : 텍스트 파일에서 바이너리 파일로 변환
```
```java
// produces = "제공 하고자하는 Mime Type"
@GetMapping(value = "/users/{id}", produces = "application/vnd.company.appv1+json")
public MappingJacksonValue retrieveUserV1(@PathVariable int id){ ... }

@GetMapping(value = "/users/{id}", produces = "application/vnd.company.appv2+json")
public MappingJacksonValue retrieveUserV2(@PathVariable int id){ ... }
```
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBoot/img/mime.PNG">
</pre>