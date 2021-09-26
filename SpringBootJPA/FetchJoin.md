# 페치 조인(fetch join)
<pre>
<b>페치 조인</b>
- SQL 조인 종류가 아닌 JPQL에서 <b>조인 성능 최적화</b>를 위해 제공하는 기능이다.
- 연관관계가 있는 엔티티나 컬렉션을 프록시가 아닌 즉시 로딩할 수 있다.
- 연관된 엔티티나 컬렉션을 <b>SQL 한 번에 함께 조회</b>할 수 있다.
- join fetch 명령어 사용
- 페치 조인 ::= [LEFT [OUTER] | INNER] JOIN FETCH 조인경로
</pre>
## 엔티티 페치 조인
<pre>
- 연관된 엔티티(N:1 관계)도 함께 한 번에 조인
- ex) 회원(Member)을 조회하면서 연괸된 팀(Team) 엔티티도 함께 조회(SQL 한 번에)
- SQL을 보면 회원 뿐만 아니라 팀(T.*)도 함께 SELECT
- <b>페치 조인으로 프록시가 아닌 진짜 데이터를 조회해서 한 번에 가져온다</b>(즉시 로딩)
</pre>
```sql
-- JPQL
select m from Member m join fetch m.team

-- SQL
SELECT M.*, T.* FROM MEMBER M INNER JOIN TEAM T ON M.TEAM_ID = T.ID
```
## 컬렉션 페치 조인
<pre>
- 연관된 컬렉션(1:N 관계)도 함께 한 번에 조인
- ex) 팀(Team)을 조회하면서 연괸된 회원(Member) 컬렉션도 함께 조회(SQL 한 번에)

<b>주의</b>
- 1:N 관계를 가지고 있는 컬렉션(JCF)를 조인하면 다(N)를 기준으로 행(ROW)이 생성되므로 중복 데이터가 발생한다.
- 팀(Team) 엔티티의 회원 컬렉션(Members) 조회하기
  <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/JCFFetchJoin.PNG"/>
  -> 다(N)를 기준으로 조인이 되어 팀A가 2개 나온다.
  -> 결과 :
     teamname = 팀A, team = Team@0x100
       username = 회원1
       username = 회원2
     teamname = 팀A, team = Team@0x100
       username = 회원1
       username = 회원2
     teamname = 팀B, team = Team@0x200
       username = 회원3

<b>해결</b>
- 중복제거를 위해 <b>DISTINCT를 사용</b>한다.
- JPQL의 DISTINCT는 <b>SQL에 distinct를 추가</b>하고, 더해서 <b>같은 엔티티가 조회되면 
  애플리케이션단에서 JPA가 같은 식별자를 가진 엔티티를 제거해준다.</b>
- 결과는 distinct으로 인해서 정상적으로 중복 데이터 없이 조회되지만, <b>실제로 DB에서는 중복제거가 안된다.</b>
</pre>
```sql
-- JPQL
select t from Team m join fetch t.members where t.name = 'TeamA'

-- SQL
SELECT T.*, M.* FROM TEAM T INNER JOIN MEMBER M ON T.ID = M.TEAM_ID WHERE T.NAME = 'TeamA'
```
## 페치 조인과 일반 조인의 차이점
<pre>
<b>일반 조인</b>
- JPQL은 결과를 반환할 때 연관관계를 고려하지 않는다.
- 그러므로 일반 조인 실행 시 연관된 엔티티를 함께 조회하지 않는다.

<b>페치 조인</b>
- 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회(즉시 로딩)
- <b>페치 조인은 객체 그래프를 SQL 한 번에 조회하는 개념</b>
</pre>
## 페치 조인의 특징과 한계
<pre>
<b>1. 페치 조인 대상에는 별칭(as)을 줄 수 없다.</b>
- 기본적으로 객체그래프를 탐색 시 모든 데이터를 가져오는것을 원칙으로 한다.
  (Team Entity의 Member Collection 조회 시 모든 회원(Member) 데이터를 가져온다는 것을 가정하에 설계되어 있다)
- 그런데 만약에 where절에 조건을 주어서 객체그래프로 탐색한 데이터중에 일부분의
  데이터만 가져온다면 나머지 데이터가 지워지거나 이상하게 동작할 수 있다.
  <b>그러므로 별칭을 통한 일부분의 데이터만 가져오지 않도록 한다.</b>

<b>2. 둘 이상의 컬렉션은 페치 조인 할 수 없다.</b>

<b>3. 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다.</b>
- 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능
- 하이버네이트는 경고 로그(WARN)를 남기고 메모리에서 페이징(매우 위험)

<b>문제</b>
- 일다대에서 일(1)을 기준으로 페이징을 하는 것이 목적이다. 그런데 데이터는 다(N)를 기준으로 row가 생성된다.
- 이 경우 하이버네이트는 경고 로그를 남기고 모든 DB 데이터를 읽어서 메모리에서 페이징을 시도한다. 최악의 경우 장애로 이어질 수 있다.

<b>해결방법1</b>
1. 먼저 <b>ToOne</b>(OneToOne, ManyToOne) 관계를 모두 페치조인 한다.
   (ToOne 관계는 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다)
2. 컬렉션은 지연 로딩으로 조회한다.
3. 지연 로딩 성능 최적화를 위해 <b>hibernate.default_batch_fetch_size</b> , <b>@BatchSize</b> 를 적용한다.
    * <b>hibernate.default_batch_fetch_size</b> : 글로벌 설정
    * <b>@BatchSize</b> : 개별 최적화
    * <b>이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size 만큼 IN 쿼리로 조회한다.</b>
    
<b>해결방법2</b>
1. 방향을 뒤집는다.
   EX) SELECT t FROM Team t JOIN FETCH t.members -> SELECT m FROM Member m JOIN FETCH m.team

<b>장점</b>
- <b>쿼리 호출 수가 1 + N -> 1 + 1으로 최적화 된다.</b>
- 조인보다 DB 데이터 전송량이 최적화 된다.(이 방법은 각각 조회하므로 전송해야할 중복 데이터가 없다.)
- 페이 조인 방식과 비교해서 쿼리 호출 수가 약간 증가하지만, DB 데이터 전송량이 감소한다.
- 컬렉션 페치 조인은 페이징이 불가능 하지만 이 방법은 페이징이 가능하다.

<b>결론</b>
- ToOne 관계는 페치 조인해도 페이징에 영향을 주지 않는다.
- <b>따라서 ToOne 관계는 페치조인으로 쿼리 수를 줄이고 해결하고, 나머지는 hibernate.default_batch_fetch_size 로 최적화 하자.</b>

<b>참고</b>
- default_batch_fetch_size 의 크기는 적당한 사이즈를 골라야 하는데, 100~1000 사이를 선택하는 것을 권장한다.
  이 전략은 SQL IN 절을 사용하는데, 데이터베이스에 따라 IN 절 파라미터를 1000으로 제한하기도 한다.

<b>querydsl</b>
- 가장 깔끔한 방법은 querydsl 을 이용해서 처리하는 것이다.
- 위에 고민들을 할필요가 없음. 동적 쿼리면 동적쿼리, 페이징이면 페이징 처리가 가능하다.
</pre>
## 페치 조인 - 정리
<pre>
- 페치 조인으로 연관관계가 있는 엔티티나 컬렉션의 데이터를 프록시가 아닌 즉시 로딩할 수 있다.
- <b>페치 조인은 객체 그래프를 유지할 때 사용하면 효과적</b>
  (객체 그래프를 탐색해야될 때 효과적이다 EX) m.team.teamName)
- 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 하면,
  페치 조인 보다는 일반 조인을 사용하고 필요한 데이터들만 조회하여 DTO로 반환하는 것이 효과적이다.
</pre>
