# Java Persistence API
<pre>
<b>JPA란?</b>
- Java Persistence API
- 자바진영의 <b>ORM</b> 기술 표준

<b>ORM이란?</b>
- Object-relational mapping(객체 관계 매핑)
- <b>객체는 객체대로 설계</b>하고 <b>관계형 데이터베이스는 관계형 데이터베이스대로 설계</b>를 하면
  중간의 차이들은 <b>ORM 프레임워크가 중간에서 매핑</b>을 통해서 해결 해준다.
- 대중적인 언어에는 대부분 ORM 기술이 존재
</pre>
## JPA는 애플리케이션과 JDBC 사이에서 동작
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/JPA(JDBC).PNG"/>
- <b>개발자가 직접 JDBC API를 쓰는 것이 아닌 JPA에게 명령을 하면 JPA는 JDBC를 사용해서 DB에 SQL을 호출하고 결과를 반환 받는다.</b>
</pre>
## JPA는 표준 명세
<pre>
- <b>JPA는 인터페이스의 모음</b>
- JPA 2.1 표준 명세를 구현한 3가지 구현체
- 하이버네이트(Hibernate), EclipseLink, DataNucleus
</pre>
## JPA를 왜 사용해야 하는가?
<pre>
<b>SQL 중심적인 개발에서 객체 중심으로 개발</b>


<b>생산성 - JPA와 CRUD</b>
• 저장: jpa.persist(member)
• 조회: Member member = jpa.find(memberId)
• 수정(dirty checking): member.setName(“변경할 이름”)
• 삭제: jpa.remove(member)


<b>유지보수</b>
- 기존 : 필드 변경시 필드와 관련된 모든 SQL을 수정해야 한다.
- JPA : 필드만 추가하면 됨, SQL은 JPA가 처리한다.


<b>패러다임의 불일치 해결</b>
<b>1.JPA와 상속</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/JPA_EXTEND.PNG"/>

    <b>저장</b>
    <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/PERSIST.PNG"/>
    - jpa.persist() 를 이용하면 JPA가 자동으로 INSERT 쿼리를 생성 해준다.
    - ALBUM과 상속관계인 ITEM 객체에도 INSERT 쿼리도 자동으로 생성하여 저장 시켜준다.

    <b>조회</b>
    <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/FIND.PNG"/>
    - jpa.find(식별자) 를 이용하면 JPA가 자동으로 SELECT 쿼리를 생성 해준다.
    - ALBUM과 상속관계인 ITEM을 JOIN 하여 조회 시켜준다.

<b>2.JPA와 연관관계, 객체 그래프 탐색</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/OBJECT_GRAPH.PNG"/>
- JPA를 통해서 가져온 객체는 객체 그래프를 자유롭게 탐색할 수 있다.
- Member 에서 getTeam()을 통해서 Team 객체를 가져올 수 있다.

<b>3.JPA와 비교하기</b>
- 동일한 트랜잭션에서 조회(find)한 엔티티는 같음을 보장한다.


<b>성능 최적화 기능</b>
<b>1.1차 캐시와 동일성(identity) 보장</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/CACHE.PNG"/>
- <b>같은 트랜잭션 안에서는 같은 엔티티를 반환</b> - 약간의 조회 성능 향상
- 동일한 PK(식별자) 값으로 조회(find)하면 동일한 객체를 반환해준다 - SQL 1번만 실행

<b>2.트랜잭션을 지원하는 쓰기 지연(transactional write-behind)</b>

    <b>INSERT</b>
    <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/transactional_write-behind1.PNG"/>
    - 트랜잭션을 커밋할 때까지 INSERT SQL을 모음
    - JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송

    <b>UPDATE</b>
    <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/transactional_write-behind2.PNG"/>
    - UPDATE, DELETE로 인한 로우(ROW)락 시간 최소화
    - 트랜잭션 커밋 시 UPDATE, DELETE SQL 실행하고, 바로 커밋


<b>3.지연 로딩(Lazy Loading)</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/LAZY.PNG"/>
- <b>지연 로딩(LAZY)</b>: 객체가 실제 사용될 때 로딩
  (조회할 때 Member 객체만 가져오고 Team 객체는 필요할 때 따로 로딩한다)
- <b>즉시 로딩(EAGER)</b>: JOIN SQL로 한번에 연관된 객체까지 미리 조회
  (조회할 때 Member 객체와 연관된 Team 객체를 한번에 로딩한다)


<b>데이터 접근 추상화와 벤더 독립성</b>

<b>표준</b>
</pre>
