# Overview of Connecting RESTful Service to JPA
## JPA(Java Persistence API)
<pre>
자바 ORM 기술에 대한 API 표준 명세를 의미한다.
<b>JPA는 자바 어플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스</b>이다.
(인터페이스이기 때문에 메서드의 선언부만 존재)

JAP를 사용하기 위해서는 JPA를 구현한 ORM 프레임워크를 사용해야 한다.
EntityManager를 통해 CRUD 처리를 한다.
</pre>
## Spring Data JPA
```
Spring Module중의 하나로, JAP를 추상화한 Repository 인터페이스를 제공한다.
```
## ORM(Object Reletionship Mapping)
<pre>
<b>객체와 DB의 테이블이 매핑을 이루는 것</b>을 의미한다(JAVA에 국한된 기술은 아니다)
ORM을 사용하면 SQL Query가 아닌 직관적인 코드(메소드)로 데이터를 조작할 수 있다.

예를 들어, User 테이블의 데이터를 출력하기 위해서는 SELECT*FROM user;라는 query를 실행해야하지만,
ORM을 사용하면 user.findAll() 이라는 메소드 호출(user : User테이블과 매핑된 객체)로 데이터 조회가 가능하다.

query를 직접 작성하지 않고 메서드 호출만으로 query가 수행되다 보니, ORM을 사용하면 <b>생산성이 매우 높아진다.</b>
그러나 <b>query가 복잡해지면 ORM으로 표현하는데 한계가 있고, 성능이 raw query에 비해 느리다는 단점</b>이 있다.
그래서 한 프로젝트 내에서 Mybatis와 JPA를 같이 사용하기도 한다.
</pre>
## JPA의 탄생 배경
<pre>
Mybatis에서는 테이블 마다 비슷한 CRUD SQL을 반복적으로 사용했었다.
DAO 개발이 매우 반복되는 작업이라 매우 귀찮고 번거롭다.
또한 테이블에 컬럼이 하나 추가된다면 이와 관련된 모든 DAO의 SQL문을 수정해야 한다.
<b>즉, DAO와 테이블은 강한 의존성을 갖고 있다.</b>

객체 모델링보다 데이터 중심 모델링(테이블 설계)를 우선시 했으며,
객체지향의 장점(추상화, 캡슐화, 정보은닉, 상속, 다형성)을 사용하지않고
단순히 데이터 전달 목적(VO, DTO)에만 집중했기 때문에 전혀 객체 지향적이지 않았다.

즉, JDBC API를 사용했을 때에는 유사한 CRUD SQL의 반복 작업과
객체를 단순히 데이터 전달 목적으로 사용할 뿐, 객체 지향적이지 못하다는 문제가 있다.

-> 그래서 <b>객체와 테이블을 매핑시켜주는 ORM이 주목</b> 받기 시작했고,
   자바 진영에서는 JPA라는 표준 스펙이 정의되었다.
</pre>
## Hibernate(ORM framework)
<pre>
<b>JPA의 구현체, 인터페이스를 직접 구현한 라이브러리</b>
EJB(Enterprise JavaBeans) 에 사용되었던 Entity Beans 이용을 대체할 목적으로 개발되었다.

하이버네이트 API는 자바 패키지를 통해 제공된다.
- <a href="http://docs.jboss.org/hibernate/stable/core/javadocs/index.html?overview-summary.html">org.hibernate</a>
- <a href="http://docs.jboss.org/hibernate/stable/core/javadocs/org/hibernate/SessionFactory.html">org.hibernate.SessionFactory</a>
- <a href="http://docs.jboss.org/hibernate/stable/core/javadocs/org/hibernate/Session.html">org.hibernate.Session</a>

<b>장점</b>

<b>생산성</b>
    그런데 SQL을 직접 사용하지 않는다고 해서 SQL을 몰라도 된다는 것은 아니다.
    Hibernate가 수행한 쿼리를 콘솔로 출력하도록 설정을 할 수 있는데, 쿼리를 보면서 의도한 대로 쿼리가 짜여졌는지, 
    성능은 어떠한지에 대한 모니터링이 필요하기 때문에 SQL을 잘 알아야 한다.

    Hibernate는 SQL를 직접 사용하지 않고, 메서드 호출만으로 쿼리가 수행된다.
    즉, SQL 반복 작업을 하지 않으므로 생산성이 매우 높아진다.

<b>유지보수</b>
    테이블 컬럼이 경되었을 경우

    Mybatis에서는 관련 DAO의 파라미터, 결과, SQL 등을 모두 확인하여 수정해야 하지만
    JPA를 사용하면 JPA가 이런 일들을 대신해주기 때문에 유지보수 측면에서 좋다.

<b>특정 벤더에 종속적이지 않음(비종속성)</b>
    즉, 설정 파일에서 JPA에게 어떤 DB를 사용하고 있는지 알려주기만 하면 얼마든지 DB를 바꿀 수가 있다.

    여러 DB 벤더( MySQL, Oracle 등.. )마다 SQL 사용이 조금씩 다르기 때문에 애플리케이션 개발 시 처음 선택한 DB를
    나중에 바꾸는 것은 매우 어렵다.
    그런데 JPA는 추상화된 데이터 접근 계층을 제공하기 때문에 특정 벤더에 종속적이지 않다.


<b>단점</b>

<b>성능</b>
    메서드 호출로 쿼리를 실행한다는 것은 내부적으로 많은 동작이 있다는 것을 의미하므로,
    직접 SQL을 호출하는 것보다 성능이 떨어질 수 있다.

    실제로 초기의 ORM은 쿼리가 제대로 수행되지 않았고, 성능도 좋지 못했다고 한다.
    그러나 지금은 많이 발전하여, 좋은 성능을 보여주고 있고 계속 발전하고 있다.

<b>세밀함</b>
    메서드 호출로 SQL을 실행하기 때문에 세밀함이 떨어진다.
    또한 객체간의 매핑( Entity Mapping )이 잘못되거나 JPA를 잘못 사용하여 의도하지 않은 동작을 할 수 있다.

    복잡한 통계 분석 쿼리를 메서드 호출로 처리하는 것은 힘든 일이다.

    이것을 보완하기 위해 JPA에서는 SQL과 유사한 기술인 JPQL을 지원한다.
    물론 SQL 자체 쿼리를 작성할 수 있도록 지원도 하고 있다.

<b>러닝커브</b>
    JPA를 잘 사용하기 위해서는 알아야 할 것이 많다.
    그래서 이러한 복잡성을 해결하고자 최근에는 Spring Data JDBC가 주목을 받고 있다( 2018-09-21 첫 1.0.0 RELEASE )
</pre>
## 정리
<pre>
<b>JPA가 SQL을 직접 작성하지 않는다고 해서 JDBC API를 사용하지 않는 것은 아니다.</b>

Hibernate가 지원하는 메소드 내부에서는 JDBC API가 동작하고 있으며, 단지 개발자가 직접 SQL문을 작성하지 않을 뿐이다.
그래서 JPA가 수행하는 쿼리가 내 의도대로 실행이 된건지 모니터링을 할 줄 알아야 한다.
비즈니스가 복잡하고, 안정성을 중요시하는 서비스일 경우에는 JPA보다 SQL을 작성하는 것이 더 안전하다.
</pre>

참고사이트 : [JAP와 ORM, Hibernate](https://prinha.tistory.com/entry/Spring-Boot-RESTful-Service-%EA%B0%95%EC%9D%98-%EC%A0%95%EB%A6%AC-12-JPAJava-Persistence-API%EC%99%80-ORM-Hibernate?category=904497)
