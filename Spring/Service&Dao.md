# Service&Dao(Data Access Object)
## 서비스 객체(bean) 구현
1. new 연산자를 이용한 service 객체 생성 및 참조(java에서만 쓰자.)
```java
MemberService service = new MemberService();
```
2. 스프링 설정파일(servlet-context.xml)을 이용한 서비스 객체 생성 및 의존 객체 자동 주입<br/>

스프링 설정파일(servlet-context.xml)
```xml
<!-- service bean 생성 -->
<beans:bean id="service" class="com.bs.lec17.member.service.MemberService"></beans:bean> 
```
Controller
```java
@Controller
public class MemberController {
    @Autowired
    MemberService service;
}
```
3. 어노테이션을 이용해서 서비스 객체 생성 및 의존 객체 자동 주입<br/>

Service
```java
//@Component
//@Repository
@Service
public class MemberService implements IMemberService {

}
```
* `@Component` 또는 `@Repository` 또는 `@Service` 어노테이션을 사용해서 객체(bean)를 생성할 수 있다.
* Service 객체 이므로 `@Service` 어노테이션을 이용하자.<br/>

Controller
```java
@Controller
public class MemberController {
    @Autowired
    MemberService service;
}
```
## DAO 객체(bean) 구현
1. 어노테이션을 이용해서 DAO 객체 생성 및 의존 객체 자동 주입(`Service와 동일`)<br/>

DAO
```java
@Repository
public class MemberDao implements IMemberDao {

}
```
* `@Component` 또는 `@Repository` 어노테이션을 사용해서 객체(bean)를 생성할 수 있다.<br/>

Service
```java
@Service
public class MemberService {
    @Autowired
    MemberDao dao;
}
```
## CharacterEncodingFilter를 통한 UTF-8 한글 인코딩 처리
web.xml
```xml
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>
        org.springframework.web.filter.CharacterEncodingFilter 
    </filter-class>
    <init-param>
    <param-name>encoding</param-name>
    <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
    <param-name>forceEncoding</param-name>
    <param-value>true</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
