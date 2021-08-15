# SpringBoot
<pre>
스프링 부트는 2014년부터 개발된 스프링의 하위 프로젝트 중 하나이다.

1. <b>단독 실행(stand-alone)이 가능</b>한 수준의 스프링 어플리케이션 개발 가능
2. <b>내장된(Embed) 톰켓, Jetty, UnderTow 등의 서버를 이용해서 별도의 서버설치 없이</b> 실행이 가능
3. <b>자동화(Automatically)된 설정방식</b>을 제공. 코드 생산성이 뛰어나다.
4. 기존 스프링 환경에서 설정했던 <b>XML 없이 단순 자바수준의 설정방식</b>을 제공
5. <b>maven, gradle의 환경을 제공</b>해준다.
6. <b>noSQL, AWS등의 환경도 자동으로 제공</b>된다.

기존 스프링에서는 여러가지 설정을 할려면 xml이 필요하다.
아래 경우는 일반적인 스프링 환경에서 web.xml servlet-context.xml 등 여러가지 설정을 하는 경우이다.

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBoot/img/SpringXML.PNG"/>

하지만 스프링부트의 경우 이러한 xml 설정이 필요가 없이 자바코드로 설정하여 쉽게 관리를 할 수 있다.
</pre>
## 기본 프로젝트 구조
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBoot/img/SpringBoot.PNG"/>

* <b>/src/main/java</b> : java source directory(MVC 관련 패키지 정의)
  - MVC관련 패키지들은 반드시 @SpringBootApplication 패키지(루트 패키지)의 하위 패키지에 있어야 정상적으로 Component Scan이 가능
* <b>/src/main/resources</b> : sql쿼리문이 작성되는 mapper.xml파일, static파일, application.properties가 있다.
  - <b>static</b> : html, css, js, img 등 정적 resource directory
  - <b>application.properties</b> : application 및 spring의 설정 등에서 사용할 여러 가지 property를 정의한 file
* <b>Application.java</b> : application을 시작할 수 있는 main method가 존재하는 spring 구성 Main Class
* <b>templates</b> : Spring Boot에서 사용 가능한 여러가지 View Template 위치
* <b>src/main</b> : jsp 등의 resource directory
</pre>
## JSP 사용하기
<pre>

spring boot 에서 모든 설정(servlet-context, root-context 등)은 application.properties 에 작성된다.
외부에서 읽어들일 필요가 없으므로 web.xml도 필요가 없고 webapp에는 결국 화면에 보여줄 View 파일만 존재한다.

<b>JSP 사용을 위해 /src/main/webapp/WEB-INF/views/ 폴더를 만들고 여기에 jsp 파일을 만들어주어야 한다.</b>
</pre>
### application.properties
```properties
# View Resolver setting
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```
### pom.xml
```xml
<!-- https://mvnrepository.com/artifact/javax.servlet/jstl -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.tomcat.embed/tomcat-embed-jasper -->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
</dependency>
```
