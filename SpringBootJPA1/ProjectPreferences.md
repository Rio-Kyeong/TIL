# Project Preferences
## Produce Project
<pre>
스프링 부트 스타터(<a href="https://start.spring.io/">https://start.spring.io/</a>)를 이용하여 프로젝트 생성하기

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA1/img/Templates.PNG"/>
</pre>
## Build Tool
<pre>
<b>빌드 관리 도구(Build Tool)</b>는 각 라이브러리들을 번거롭게 모두 다운받을 필요없이,
빌드도구 설정파일에 필요한 라이브러리 종류와 버전들, 종속성 정보를 명시하여
필요한 라이브러리들을 설정파일을 통해 자동으로 다운로드 해주고 이를 간편히 관리해주는 도구이다.

우리가 프로젝트에서 작성한 java 코드와 프로젝트 내에 필요한 각종 xml, properties, jar 파일들을
JVM이나 WAS가 인식할 수 있도록 패키징 해주는 빌드 과정, <b>빌드 자동화 도구</b>라고 할 수 있다.

<b>Maven</b>
- Maven은 <b>Java용 프로젝트 관리도구로 Apache의 Ant 대안</b>으로 만들어졌다.
- 필드 중인 프로젝트, 빌드 순서, 다양한 외부 라이브러리 종속성 관계를 pom.xml파일에 명시한다.
- Maven은 외부저장소에서 필요한 라이브러리와 플러그인들을 다운로드 한다음, 로컬시스템의 캐시에 모두 저장한다.
- Maven : <a href="https://mvnrepository.com/">https://mvnrepository.com/</a>

<b>Gradle</b>
- Gradle은 <b>Groovy를 이용한 빌드 자동화 시스템</b>이다.
- Apache Maven과 Apache Ant에서 볼 수 있는 개념들은 사용하는 대안으로써 나온 프로젝트 빌드 관리 툴이다(완전한 오픈소스)
- <b>Groovy 언어 기반의 (DSL)Domain-specific-language를 사용</b>한다(설정파일을 xml파일을 사용하는 Maven보다 코드가 훨씬 간결하다)
- Gradle은 프로젝트의 어느부분이 업데이트되었는지 알기 때문에, 빌드에 점진적으로 추가할 수 있다.
  -> 업데이트가 이미 반영된 빌드의 부분은 즉 더이상 재실행되지 않는다(<b>빌드 시간이 훨씬 단축될 수 있다</b>)
</pre>
## 타임리프(thymeleaf)
<pre>
타임리프는 흔히 <b>View Template(뷰 템플릿)</b>이라고 부른다.
뷰 템플릿은 컨트롤러가 전달하는 데이터를 이용하여 동적으로 화면을 구성할 수 있게 해준다.

<b>Thymeleaf와 jsp의 차이점</b>

- <b>Thymeleaf</b>는 (HTML, XML, JavaScript, CSS 및 일반 텍스트)를 처리 할 수 있는 웹 및 독립형 환경에서 사용할 수 있는 Java 템플릿 엔진이다.
- html파일을 가져와서 파싱해서 분석후 정해진 위치에 데이터를 치환해서 웹 페이지를 생성한다.
  (코드자체가 html 같은 웹형태이기에 was없이도 그냥 브라우저에서 직접 띄어볼 수 가 있다)
- Thymeleaf는 자바코드를 사용할 수 없고, jsp에서 처럼 커스텀 태그와 같은 기능도 없다.

- <b>JSP</b>는 서블릿으로 변환되어 실행이 되어진다.
- JSP 내에서 자바 코드를 사용할 수도 있다(유지보수가 힘들어지기 때문에 자바 코드를 넣어서 잘 사용하지 않음)
  
</pre>
### Gradle(build.gradle)
```gradle
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	}
```
### Controller
```java
@Controller
public class HelloController {

    @GetMapping(value = "/hello")
    public String hello(Model model){
        model.addAttribute("data","hello!!!");
        return "hello";
    }

}
```
### View
```html
<!-- 파일 : hello.html(동적인 페이지) -->
<!-- 경로 : main/resources/templates -->
<!-- URL : http://localhost/hello -->

<!DOCTYPE HTML>
<!-- html 문서의 언어는 한글(ko)이며 타임리프를 사용한다 -->
<html lang="ko" xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<p th:text="'안녕! ' + ${data}" >안녕하세요. 손님</p>
</body>
</html>
```
```html
<!-- 파일 : index.html(정적인 페이지) -->
<!-- 경로 : main/resources/static -->
<!-- URL : http://localhost/ -->

<!DOCTYPE HTML>
<!-- html 문서의 언어는 한글(ko)이며 타임리프를 사용한다 -->
<html lang="ko" xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
Hello
<a href="/hello">hello</a>
</body>
</html>
```
## H2Database
<pre>
H2DB는 개발이나 테스트 용도로 가볍고 편리한 자바 기반의 오픈소스 관계형 데이터 베이스 관리 시스템(DBMS)이다.
H2DB는 <b>서버(Server)</b> 모드와 임베디드(Embedded) 모드의 인메모리 DB 기능을 지원한다(본 강의에서는 <b>서버 모드</b>를 사용한다)

Version 1.4.200(Windows): <a href="https://h2database.com/h2-setup-2019-10-14.exe">https://h2database.com/h2-setup-2019-10-14.exe</a>

<b>서버모드 사용하기</b>
  1. H2Database 다운로드
  2. H2/bin/h2.bat 실행
  3. URL에서 앞에 ip를 localhost로만 딱 바꿔준다(key는 건들지 않는다)
     예:) http://localhost:8082/?key=xxxxxx...
  4. H2 데이터베이스 화면의 접속 URL에 jdbc:h2:~/test, jdbc:h2:~/jpashop 등 파일명을 적어주고 연결을 한다(최초 1회)
     <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA1/img/server.PNG"/>
  5. C:/Users/home(본인 홈 폴더)에 test.mv.db 또는 jpashop.mv.db 같은 파일이 생성되었는지 확인한다.
  6. H2 데이터베이스 화면의 접속 URL에 jdbc:h2:tcp://localhost/~/test, jdbc:h2:tcp://localhost/~/jpashop와 같이 설정 내용에 맞게 입력해서 접속해준다.
  7. H2 접속 오류 시 <a href="https://docs.google.com/document/d/1j0jcJ9EoXMGzwAA2H0b9TOvRtpwlxI5Dtn3sRtuXQas/edit#">참고</a>
</pre>
### Gradle(build.gradle)
```gradle
dependencies {
	runtimeOnly 'com.h2database:h2:1.4.200'
	}
```
### application.yml
```yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/jpashop
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      # 애플리케이션 실행 시점에 테이블을 drop 하고, 다시 생성한다(초기화)
      ddl-auto: create
    properties:
      hibernate:
        # show_sql : System.out 에 하이버네이트 실행 SQL을 남긴다.
        # show_sql: true
        format_sql: true

logging:
  level:
    # org.hibernate.SQL : logger를 통해 하이버네이트 실행 SQL을 남긴다.
    org.hibernate.SQL: debug
    # org.hibernate.type : SQL 실행 파라미터를 로그로 남긴다.
    org.hibernate.type: trace
```
## Query Parameter log
<pre>
- <b>쿼리 파라미터 로그 남기기</b>(쿼리 파라미터는 물음표(?)로 남기 때문에 볼 수 없다)
  1. 로그에 다음을 추가하기 <b>org.hibernate.type</b> : SQL 실행 파라미터를 로그로 남긴다.
  2. 외부 라이브러리 사용(<b>implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.6'</b>)
</pre>
### Console
```sql
insert 
    into
        member
        (username, id) 
    values
        (?, ?)

-- 외부 라이브러리 추가 시
insert into member (username, id) values (?, ?)
insert into member (username, id) values ('memberA', 1);
```