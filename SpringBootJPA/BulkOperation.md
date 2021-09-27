# 벌크 연산
<pre>
<b>벌크 연산</b>
- 여러 건(대량의 데이터)을 한 번에 수정하거나 삭제하는 JPQL
  (쿼리 한 번으로 여러 테이블 로우 변경(엔티티))
- 많은 데이터 변경을 JPA 변경 감지 기능(Drity Checking)으로 실행하려면 너무 많은 SQL이 실행된다.
- executeUpdate()의 결과는 영향받은 엔티티 수 반환(int)
- <b>UPDATE, DELETE 지원</b>
- <b>INSERT(insert into .. select)  -> Insert Subquery 하이버네이트 지원</b>

<b>벌크 연산 주의</b>
- <b>벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리</b>
- 데이터베이스는 데이터 변경이 일어나지만 이미 영속성 컨텍스트에 있는 데이터는 변경이 일어나지 않는다.
- 데이터 정합성이 맞지 않을 수 있다.

<b>해결 방법</b>
- 벌크 연산을 먼저 수행
- <b>벌크 연산 수행 후 영속성 컨텍스트 초기화</b>(em.clear)
</pre>
## 벌크 연산 - JPQL
```java
// 재고가 10개 미만인 모든 상품의 가격을 10% 상승
String qlString = "update Product p " +
                  "set p.price = p.price * 1.1 " + 
                  "where p.stockAmount < :stockAmount";

// Query 수행으로 자동으로 플러시가 된다.
// 자동 플러시가 되는 시점 : Query 수행, 강제 플러시 선언, commit
int resultCount = em.createQuery(qlString)
        .setParameter("stockAmount", 10)
        .executeUpdate();

em.clear();
```
## Spring Data JPA - @Modifying
<pre>
- Spring Data JPA 에서는 벌크 연산 수행 후 영속성 컨텍스트 초기화(em.clear)를 위해 <a href="https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.modifying-queries">@Modifying</a>을 사용한다.
</pre>
```java
@Modifying
@Query("update User u set u.firstname = ?1 where u.lastname = ?2")
int setFixedFirstnameFor(String firstname, String lastname);
```
