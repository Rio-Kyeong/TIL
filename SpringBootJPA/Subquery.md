# Subquery
<pre>
<b>Subquery</b>
- 하나의 SQL문 안에 포함되어 있는 또 다른 SQL문(메인쿼리가 서브쿼리를 포함하는 종속적인 관계)

<b>insert subquery</b>
- 조회된 여러 행의 레코드를 한 번에 추가할 때(복수행)
- 정산작업 수행에 많이 사용

<b>update subquery</b>
- 다른 컬럼의 정보를 가지고 레코드를 변경할 때
- 단수행만 가능(in을 사용하여 다수행 가능)

<b>delete Subquery</b>
- 다른 컬럼의 정보를 가지고 레코드를 삭제 할 때
- 단수행만 가능(in을 사용하여 다수행 가능)

<b>select subquery</b>
- 다른 컬럼의 값으로 조회를 수행 해야 할 때
- 단수행, 복수행 서브쿼리 모두 가능

<b>select subquery의 위치에 따른 명칭</b>
- SELECT문에 있는 subquery : 스칼라 서브 쿼리 - 다른 테이블 컬럼의 값을 보여줄 때
- FROM문에 있는 subquery : 인라인 뷰를 사용한 검색결과 재 조회
- WHERE문에 있는 subquery : 중첩 서브 쿼리 - 다른 컬럼의 값으로 조건을 만들 때
</pre>
## JPA 서브 쿼리
<pre>
- JPA자체로는 WHERE, HAVING절에서만 서브 쿼리가 사용이 가능하다.
- 하이버네이트가 지원을 해줘 SELECT절까지 서브 쿼리를 쓸 수 있다.

<b>JPA 서브 쿼리의 한계</b>
- 하지만 <b>FROM 절에서는 서브 쿼리가 불가능</b>하므로 정말 성능이 중요하다면 네이티브 SQL로 작성하자.
- FROM 절은 JOIN으로 해결할 수 있는 가능성이 높으니 <b>JOIN 절을 사용</b>하던지 쿼리 두 번 날리는 것도 하나의 방법이다.
</pre>
## 서브쿼리 - JPQL
```java
em.persist(new Member("A", 10));
em.persist(new Member("B", 20));
em.persist(new Member("C", 30));
em.persist(new Member("D", 40));

// 나이가 평균보다 많은 회원 검색
List<Member> resultList = 
        em.createQuery("select m from Member m where m.age > (select avg(subM.age) from Member subM)", Member.class)
        .getResultList();

resultList.forEach(System.out::println);
```
```console
Hibernate:
    select
        member0_.id as id1_0_,
        member0_.age as age2_0_,
        member0_.TEAM_ID as team_id4_0_,
        member0_.username as username3_0_ 
    from
        Member member0_ 
    where
        member0_.age>(
            select
                avg(cast(member1_.age as double)) 
            from
                Member member1_
        )

Member{id=3, username='C', age=30}
Member{id=4, username='D', age=40}
```
<pre>
- JPQL도 서브 쿼리를 지원한다.
- 위와 같이 일반적인 서브 쿼리 작성과 똑같다.
- SQL을 확인해보면 서브 쿼리가 제대로 동작하는 것을 알 수 있다.
</pre>
## 서브 쿼리 지원 함수
### `EXISTS`
<pre>
<b>[NOT] EXISTS (subquery)</b> : 서브쿼리에 결과가 하나라도 존재하면 참
</pre>
```java
Team teamA = new Team("teamA");
em.persist(teamA);
Team teamB = new Team("teamB");
em.persist(teamB);

em.persist(new Member("A", 10, teamA));
em.persist(new Member("B", 20, teamA));
em.persist(new Member("C", 30, teamB));
em.persist(new Member("D", 40, teamB));

// 팀A 소속인 회원
// t.name이 temaA인 결과가 존재하므로 멤버가 출력된다.
List<Member> resultList = 
        em.createQuery("select m from Member m where exists (select t from m.team t where t.name = 'teamA')", Member.class)
        .getResultList();

resultList.forEach(System.out::println);
```
```console
Hibernate:
    select
        member0_.id as id1_0_,
        member0_.age as age2_0_,
        member0_.TEAM_ID as team_id4_0_,
        member0_.username as username3_0_ 
    from
        Member member0_ 
    where
        exists (
            select
                team1_.id 
            from
                Team team1_ 
            where
                member0_.TEAM_ID=team1_.id 
                and team1_.name='teamA'
        )

Member{id=3, username='A', age=10}
Member{id=4, username='B', age=20}
```
### `ALL, ANY`
<pre>
<b>{ALL | ANY | SOME} (subquery)</b>
- ALL 모두 만족하면 참
- ANY, SOME : 같은 의미, 조건을 하나라도 만족하면 참
</pre>
```java
//ALL
em.persist(new Product("Book", 10));

em.persist(new Order(20));
em.persist(new Order(11));
em.persist(new Order(5));
em.persist(new Order(10));

//ANY
Team teamA = new Team("teamA");
em.persist(teamA);
Team teamB = new Team("teamB");
em.persist(teamB);

em.persist(new Member("A", 10, teamA));
em.persist(new Member("B", 20, null));
em.persist(new Member("C", 30, teamB));
em.persist(new Member("D", 40, null));

// 전체 상품 각각의 재고보다 주문량이 많은 주문들
// 상품(Book)의 재고 10권보다 많은 20권과 11권의 2개의 주문건이 출력
List<Order> allList = em.createQuery("select o from Order o where o.orderAmount > ALL (select p.stockAmount from Product p)", Order.class)
        .getResultList();

System.out.println("================ ALL ================");
allList.forEach(System.out::println);


// 어떤 팀이든 팀에 소속된 회원
// 팀에 소속되어있는 A회원과 C회원이 출력
List<Member> anyList = em.createQuery("select m from Member m where m.team = ANY (select t from Team t)", Member.class)
        .getResultList();

System.out.println("================ ANY ================");
anyList.forEach(System.out::println);
```
```console
Hibernate: 
    select
        order0_.id as id1_1_,
        order0_.city as city2_1_,
        order0_.street as street3_1_,
        order0_.zipcode as zipcode4_1_,
        order0_.orderAmount as orderamo5_1_,
        order0_.PRODUCT_ID as product_6_1_ 
    from
        ORDERS order0_ 
    where
        order0_.orderAmount>all (
            select
                product1_.stockAmount 
            from
                Product product1_
        )

================ ALL ================
Order{id=2, orderAmount=20, address=null}
Order{id=3, orderAmount=11, address=null}


Hibernate:
    select
        member0_.id as id1_0_,
        member0_.age as age2_0_,
        member0_.TEAM_ID as team_id4_0_,
        member0_.username as username3_0_ 
    from
        Member member0_ 
    where
        member0_.TEAM_ID=any (
            select
                team1_.id 
            from
                Team team1_
        )

================ ANY ================
Member{id=8, username='A', age=10}
Member{id=10, username='C', age=30}
```
### `IN`
<pre>
<b>[NOT] IN (subquery)</b> : 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참
</pre>
```java
em.persist(new User("User1", 20));
em.persist(new User("User2", 15));

em.persist(new Member("A", 10));
em.persist(new Member("D", 15));
em.persist(new Member("B", 20));
em.persist(new Member("C", 30));

// 유저의 나이와 같은 나이를 가진 멤버
// User age의 20, 15와 같은 Member D, B가 출력
List<Member> inQuery = em.createQuery
        ("select m from Member m" +
                " where m.age in (select u.age From User u)", Member.class)
        .getResultList();

System.out.println("================ inQuery ================");
inQuery.forEach(System.out::println);
```
```console
Hibernate: 
        select
            생략...
        from
            Member member0_ 
        where
            member0_.age in (
                select
                    user1_.age 
                from
                    User user1_
            )

================ inQuery ================
Member(id=4, name=D, age=15)
Member(id=5, name=B, age=20)
```