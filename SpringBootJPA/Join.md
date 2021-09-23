# Join
<pre>
<b>조인</b>
- 둘 이상의 테이블을 연결해서 데이터를 검색하는 방법

<b>조인 종류</b>
- 내부 조인(INNER JOIN) : 교집합, 공통 레코드만 조회
- 외부 조인(OUTER JOIN) : 한쪽 테이블에만 레코드가 존재하더라도 조회
- 세타 조인(THETA JOIN) : 두 릴레이션 속성 값을 비교하여 조건에 만족하는 튜플만 반환(관계 없어도 됨)
</pre>
## 조인 - JPQL
```java
// 내부 조인 : SELECT m FROM Member m [INNER] JOIN m.team t
// INNER 생략가능
List<Member> resultList = em.createQuery("select m from Member m inner join m.team t", Member.class)
        .getResultList();

// 외부 조인 : SELECT m FROM Member m LEFT [OUTER] JOIN m.team t
// LEFT OUTER JOIN : 왼쪽 테이블을 기준으로 조인
// OUTER 생략가능
List<Member> resultList = em.createQuery("select m from Member m left outer join m.team t", Member.class)
        .getResultList();

// 세타 조인 : select count(m) from Member m, Team t where m.username = t.name
// 예) 연관관계가 없는 Member와 Team Entity에서 회원 이름과 팀 이름이 같은 회원(Member)조회
List<Member> resultList = em.createQuery("select m from Member m, Team t where m.username = t.name", Member.class)
        .getResultList();
```
## 조인 - ON 절
<pre>
<b>ON절을 활용한 조인(JPA 2.1부터 지원)</b>
- 조인 대상 필터링(조인에 조건 추가)
- 연관관계 없는 엔티티 외부 조인(하이버네이트 5.1부터)
</pre>
### `조인 대상 필터링`
```java
// 예) 회원과 팀을 조인하면서, 팀 이름이 teamA인 팀만 조인
List<Member> resultList = em.createQuery("select m, t from Member m left join m.team t on t.name = 'teamA'", Member.class)
        .getResultList();
```
```sql
SELECT m.*, t.* 
FROM Member m LEFT JOIN Team t
ON m.TEAM_ID=t.id and t.name='teamA'
```
### `연관관계 없는 엔티티 외부 조인`
```java
// 예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인
// 서로 관계가 없어도 됨
List<Member> resultList = em.createQuery("select m, t from Member m left join Team t on m.username = t.name", Member.class)
        .getResultList();
```
```sql
SELECT m.*, t.* 
FROM Member m LEFT JOIN Team t
ON m.username = t.name
```

