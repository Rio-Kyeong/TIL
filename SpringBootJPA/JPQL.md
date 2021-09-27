# 객체지향 쿼리 언어
<pre>
<b>JPA의 문제</b>
- JPA를 사용하면 <b>엔티티 객체를 중심으로 개발</b>해야 한다(Object-Oriented Programming)
- 그러므로 검색을 할 때도 <b>테이블이 아닌 엔티티 객체를 대상으로 검색</b>을 해야 한다.
- 그런데 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능하다.
- 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요하다.

<b>JPA는 다양한 쿼리 방법을 지원</b>
- <b>JPQL</b>
- JPA Criteria
- <b>QueryDSL</b>
- 네이티브 SQL
- JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용

    <b>JPQL</b>
    - JPA는 <b>SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어</b> 제공
    - SQL과 문법 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
    - <b>JPQL은 엔티티 객체를 대상으로 검색하는 객체 지향 쿼리</b>
    - SQL을 추상화해서 특정 데이터베이스 SQL에 의존X
    - SQL은 데이터베이스 테이블을 대상으로 쿼리

    <b>JPA Criteria</b>
    - 문자가 아닌 자바코드로 JPQL을 작성할 수 있음
    - JPQL 빌더 역할
    - JPA 공식 기능
    - <b>단점 : 너무 복잡하고 실용성이 없다.</b>
    - Criteria 대신 QueryDSL 사용 권장

    <b>QueryDSL</b>
    - 문제가 아닌 자바코드로 JPQL을 작성할 수 있음
    - JPQL 빌더 역할
    - 컴파일 시점에 문법 오류를 찾을 수 있음
    - <b>동적쿼리 작성 편리함</b>
    - <b>단순하고 쉬움(직관적)</b>
    - 실무 사용 권장

    <b>네이티브 SQL</b>
    - JPA가 제공하는 SQL을 직접 사용하는 기능
    - JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능
    - 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트

    <b>JDBC 직접 사용, SpringJdbcTemplate 등</b>
    - JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, MyBatis등을 함께 사용 가능
    - 단 영속성 컨텍스트를 적절한 시점에 강제로 플러시 필요
      (flush는 commit 또는 query가 날라가는 시점에 자동으로 발생하는데 JDBC를 사용시 직접 수동으로 플러시를 해야한다)
</pre>
## JPQL(Java Persistence Query Language)
<pre>
<b>JPQL</b>
- <b>SQL을 추상화한 객체 지향 쿼리 언어</b>
- <b>엔티티 객체를 대상으로 쿼리</b>
- JPQL은 SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.
- JPQL은 결국 SQL로 변환된다.

<b>JPQL 문법</b>
- select m from <b>Member</b> as m where <b>m.age</b> > 18
- 엔티티와 속성은 대소문자 구분O(Member, age)
- JPQL 키워드는 대소문자 구분X(SELECT, FROM, where)
- 엔티티 이름 사용, 테이블 이름이 아님
- <b>별칭은 필수(m) (as는 생략가능)</b>

<b>집합과 정렬</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/set.PNG"/>

<b>TypeQuery, Query</b>
- <b>TypeQuery : 반환 타입이 명확할 때 사용</b>
  EX) TypedQuery< Member > query = em.createQuery("SELECT m FROM Member m", Member.class);
- <b>Query : 반환 타입이 명확하지 않을 때 사용</b>
  EX) Query query = em.createQuery("SELECT m.username, m.age from Member m");

<b>결과 조회</b>
- <b>query.getResultList() : 결과가 하나 이상일 때, 리스트 반환</b>
  • 결과가 없으면 빈 리스트 반환
- <b>query.getSingleResult() : 결과가 정확히 하나, 단일 객체 반환</b>
  • 결과가 없으면: javax.persistence.NoResultException
  • 둘 이상이면: javax.persistence.NonUniqueResultException
</pre>
### `엔티티 직접 사용`
<pre>
<b>기본 키 값</b>
- JPQL에서 엔티티를 직접 사용하면 SQL에서 <b>해당 엔티티의 기본 키 값을 사용</b>한다.
- Where절 또는 From절에서 m(Member) 엔티티를 그대로 사용하면 SQL은 해당 엔티티의 기본 키 값 사용

<b>외래 키 값</b>
- 외래키도 마찬가지로 JPQL에서 연관관계인 외래 키 엔티티를 직접 사용하면 SQL에서 <b>해당 엔티티의 외래 키 값을 사용</b>한다.
</pre>
```sql
-- JPQL
select count(m.id) from Member m -- 엔티티의 아이디를 사용
select count(m) from Member m -- 엔티티를 직접 사용 

select count(m.team.id) from Member m -- 엔티티의 아이디를 사용
select count(m.team) from Member m -- 엔티티를 직접 사용 

-- SQL(JPQL 둘다 같은 다음 SQL 실행)
select count(m.id) as cnt from Member m

-- SQL(JPQL 둘다 같은 다음 SQL 실행)
select count(m.team_id) as cnt from Member m
```
### `파라미터 바인딩 - 이름 기준`
<pre>
- <b>query.setParameter("바인딩 변수", 찾을 값)</b>
- 콜론(:)을 사용하여 데이터가 추가될 곳을 지정해준다.
- 위치 기준 파라미터 바인딩도 있으나 권장하지 않는다.
</pre>
```java
// :username이 member인 데이터를 조회
Member singleResult = em.createQuery("select m from Member m where m.username = :username", Member.class)
        .setParameter("username", "member")
        .getSingleResult();
```
### `프로젝션`
<pre>
<b>프로젝션</b>
- SELECT 절에 조회할 대상을 지정하는 것
- 프로젝션 대상 : 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자등 기본 데이터 타입)
  (관계형 데이터베이스에서는 스칼라 타입만 선택할 수 있지만 JPQL에서는 여러 타입으로 가능)
</pre>
```java
// SELECT m FROM Member m -> 엔티티(Member) 프로젝션
List<Member> result = em.createQuery("select m from Member m", Member.class)
        .getResultList();

// SELECT m.team FROM Member m -> 엔티티(Team) 프로젝션
// JPQL은 SQL과 비슷하게 작성하여 최대한 가독성 있게 만들어야 한다.
// Team.class를 참초하는 join query가 나가므로 join을 명시하여 JPQL을 작성한다.
List<Team> result = em.createQuery("select t from Member m join m.team t", Team.class)
        .getResultList();

// SELECT m.address FROM Member m -> 임베디드 타입(Address) 프로젝션
// 임베디드 타입(Address.class)는 Order Class 안에 소속되어 있다.
// 그러므로 Order에서 찾아와야 한다.
List<Address> result = em.createQuery("select o.address from Order o", Address.class)
        .getResultList();

// 하나의 스칼라 타입 값 조회
// SELECT m.username FROM Member m -> 스칼라 타입 프로젝션
List<String> result = em.createQuery("select m.username from Member m", String.class)
        .getResultList();

// 여러 스칼라 타입 값 조회
// SELECT m.username, m.age FROM Member m -> String, Integer 어떤 타입을 써야할까?
// 1. Query 타입으로 조회
// 해당 List의 타입은 Object로 되어있으므로 캐스팅해서 Object[]로 사용하자.
List result = em.createQuery("select m.username, m.age from Member m")
        .getResultList();

// 2. Object[] 타입으로 조회
List<Object[]> result = em.createQuery("select m.username, m.age from Member m")
        .getResultList();

// 3. new 명령어로 조회
// - 단순 값을 DTO로 바로 조회
// - 패키지 명을 포함한 전체 클래스 명을 입력(jpql 패키지의 MemberDTO)
// - DTO에 순서와 타입이 일치하는 생성자 필요
List<MemberDTO> result = em.createQuery("select new jpql.MemberDTO(m.username, m.age) from Member m", MemberDTO.class)
        .getResultList();

// DISTINCT로 중복 제거
// 중복된 m.username의 값이 제거되어서 나온다.
List<String> result = em.createQuery("select distinct m.username from Member m", String.class)
        .getResultList();
```
