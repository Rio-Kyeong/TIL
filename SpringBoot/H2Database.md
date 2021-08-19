# Creating some test data
## h2
```
H2DB는 자바 기반의 오픈소스 관계형 데이터 베이스 관리 시스템(DBMS)이다.
H2DB는 서버(Server) 모드와 임베디드(Embedded) 모드의 인메모리 DB 기능을 지원한다.

물론 디스크 기반 테이블을 또한 생성할 수 있으며, 브라우저 기반의 콘솔모드를 이용할 수 있다.
별도의 설치과정이 없고 용량도 2MB(압축버전) 이하로 매우 저용량 이다.
DB자체가 매우 가볍기 때문에 매우 가볍고 빠르며, JDBC API 또한 지원하고 있다.
SQL 문법은 다른 DBMS들과 마찬가지로 표준 SQL의 대부분이 지원됩다.

이러한 장점들 때문에 어플리케이션 개발 단계의 테스트 DB로서 많이 이용됩니다.
```
### pom.xml
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```
### application.yml
```yml
spring:
    datasource:
        url: jdbc:h2:mem:testdb 
    jpa:
        show-sql: true
        hibernate:
            ddl-auto: create
        generate-ddl: true
        # Hibernate 초기화를 통해 생성된 스키마에다가 데이터를 채우기를 위해서 data.sql를 먼저 실행을 시킨다.
        defer-datasource-initialization: true
    h2:
        console:
            enabled: true
```
### h2 
<pre>
// http://localhost/h2-console
// mem : memory DB / 현재 어플리케이션이 기동되는 동안에만 유지되는 데이터베이스
// 어플리케이션이 종료되도 유지되는 데이터베이스를 원한다면 TCP방식이 지원되는 드라이버 방식으로 바꿔주기

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBoot/img/H2.PNG"/>
</pre>
