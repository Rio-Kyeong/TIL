# JPA Flow
## 데이터베이스 방언
<pre>
- <b>JPA는 특정 데이터베이스에 종속적이지 않다.</b>
- 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다름
  - 기변 문자 : MySQL은 VARCHAR, Oracle은 VACHAR2
  - 문자열을 자르는 함수 : SQL 표준은 SUBSTRING(), Oracle은 SUBSTR()
  - 페이징 : MySQL은 LIMIT, Oracle은 ROWNUM

<b>방언이란? SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능</b>
- 사용하는 데이터베이스에 맞게 방언을 설정해 주어야 한다.
- <b>Dialect</b>
</pre>
## JPA 설정하기
```xml
<!-- src/main/resources/META-INF/persistence.xml -->

<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
           xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <!-- JPA 는 DB 를 사용하기 때문에 접근정보를 넣어주어야 한다.-->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <!-- 방언 설정(하이버네이트는 40가지 이상의 데이터베이스 방언 지원) -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```
## JPA 구동방식
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/Persistence.PNG"/>
1. Persistence Class가 설정 정보 조회한다.
2. EntityManagerFactory를 생성한다.
3. 요청(Request)이 들어올 떄마다 EntityManager를 생성하고, 사용 후 버린다.

<b>Persistence</b> : EntityManagerFactory 인스턴스를 생성하는 정적(static) 메소드를 가지고 있다.
<b>EntityManagerFactory</b> : EntityManager 클래스의 팩토리 클래스이며, 이 클래스로 EntityManager 클래스의 인스턴스를 생성하고 관리할 수 있다.
<b>EntityManager</b> : 인터페이스이며, 객체에 대한 영속성 관리작업을 한다.

<b>주의</b>
- <b>엔티티 매니저 팩토리</b>는 하나만 생성하여 애플리케이션 전체에서 공유
- <b>엔티티 매니저</b>는 쓰레드간에 공유X (사용하고 버려야 한다)
- <b>JPA의 모든 데이터 변경은 트랜잭션 안에서 실행</b>
</pre>
## JPQL
<pre>
- JPA를 사용하면 엔티티 객체를 중심으로 개발
- 그러므로 검색을 할 때도 <b>테이블이 아닌 엔티티 객체를 대상으로 검색</b>
- JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
- <b>JPQL은 엔티티 객체</b>를 대상으로 쿼리
- <b>SQL은 데이터베이스 테이블</b>을 대상으로 쿼리

<b>JPQL은 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리이다.</b>
</pre>
## JpaMain
```java
// src/main/java/hellojpa/JpaMain
// Spring에 의존하지 않음

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import java.util.List;

public class JpaMain {

    public static void main(String[] args) {
        // Web Server 가 올라오는 시점에 한개만 생성된다.
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        // 요청이 들어올 때마다 생성하고, 사용 후 버린다.
        EntityManager em = emf.createEntityManager();
        // JPA 의 모든 데이터 변경은 트랜잭션 안에서 실행해야 한다.
        EntityTransaction tx = em.getTransaction();
        tx.begin(); // Transaction 시작

        try {
            // INSERT
            Member member = new Member();
            member.setId(2L);
            member.setName("HelloA");
            em.persist(member);

            // SELECT(단건 조회)
            Member findMember = em.find(Member.class, 1L);

            // SELECT(다중건 조회)
            // setFirstResult(int startPosition) : 조회 시작 위치
            // setMaxResults(int maxResult) : 조회할 데이터 수
            List<Member> result = em.createQuery("select m from Member as m", Member.class)
                    .setFirstResult(5)
                    .setMaxResults(8)
                    .getResultList();

            for (Member member : result) {
                System.out.println("member.getName = "+ member.getName());
            }

            // DELETE
            em.remove(findMember);

            // UPDATE(dirty checking)
            findMember.setName("HelloJPA");

            tx.commit(); // Transaction 저장
        } catch (Exception e){
            tx.rollback(); // Transaction 취소
        } finally {
            em.close(); // EntityManager 종료
        }
        emf.close(); // EntityManagerFactory 종료
    }
}
```
