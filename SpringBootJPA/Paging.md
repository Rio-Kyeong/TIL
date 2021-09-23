# Paging
<pre>
<b>페이징</b>
- <b>한정된 네트워크 자원을 효율적으로 활용하기 위해 특정한 정렬 기준에 따라 데이터를 분할하여 가져오는 것</b>

<b>JPA는 페이징을 다음 두 API로 추상화</b>
- <b>setFirstResult(int startPosition) : 조회 시작 위치(0부터 시작)</b>
- <b>setMaxResults(int maxResult) : 조회할 데이터 수</b>
</pre>
## 페이징 - JPQL
```java
// 페이징 쿼리
// Member Entity에서 2번째부터 10개의 나이를 내림차순으로 조회
List<Member> resultList = em.createQuery("select m from Member m order by m.age desc", Member.class)
        .setFirstResult(1) // 0부터 시작
        .setMaxResults(10)
        .getResultList();
```
```console
Hibernate: 
    /* select
        m 
    from
        Member m 
    order by
        m.age desc */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.TEAM_ID as team_id4_0_,
            member0_.username as username3_0_ 
        from
            Member member0_ 
        order by
            member0_.age desc limit ? offset ?
```
### `MySQL 방언`
```sql
SELECT
    M.ID AS ID,
    M.AGE AS AGE,
    M.TEAM_ID AS TEAM_ID,
    M.NAME AS NAME 
FROM
    MEMBER M 
ORDER BY
    M.AGE DESC LIMIT ?, ?
```
### `Oracle 방언`
```sql
SELECT * FROM
    ( SELECT ROW_.*, ROWNUM ROWNUM_
    FROM
        ( SELECT
            M.ID AS ID,
            M.AGE AS AGE,
            M.TEAM_ID AS TEAM_ID,
            M.NAME AS NAME
        FROM MEMBER M 
        ORDER BY M.AGE 
        ) ROW_ 
    WHERE ROWNUM <= ?
    ) 
WHERE ROWNUM_ > ?
```