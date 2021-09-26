# 경로 표현식
<pre>
<b>경로 표현식</b>
- .(점)을 찍어 객체 그래프를 탐색하는 것

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/PathExpression.PNG"/>
- <b>상태 필드(state field)</b> : 단순히 값을 저장히기 위한 필드(ex: m.username)
- <b>연관 필드(association field)</b> : 연관관계를 위한 필드
  • <b>단일 값 연관 필드</b> : @ManyToOne, @OneToOne, 엔티티로 넘어감(ex: m.team)
  • <b>컬렉션 값 연관 필드</b> : @OneToMany, @ManyToMany, 컬렉션으로 넘어감(ex: t.members)
</pre>
## 상태 필드 경로 탐색
<pre>
<b>상태 필드</b>
- Entity에서 바로 값을 가져온다.
- 경로 탐색의 끝, 더이상 탐색할 수 없다.
</pre>
```sql
-- JPQL
select m.username, m.age from Member m
-- SQL
select m.username, m.age from Member m
```
## 연관 필드 경로 탐색
<pre>
<b>연관 필드</b>
  • <b>단일 값 연관 필드</b>
    - 묵시적 내부 조인(inner join) 발생, 객체 그래프를 탐색할 수 있다.
  • <b>컬렉션 값 연관 필드</b>
    - 묵시적 내부 조인 발생, 컬렉션 자체를 가리키기 때문에 더이상 탐색할 수 없다.
    - From 절에서 명시적 조인을 통해 별칭을 얻으면 별칭을 통해 탐색 가능하다.
    - ex) select t.members from Team t <- 더이상 Members 탐색불가(묵시적 조인 발생)
    - ex) select m.username from Team t join t.members m <- 명시적 조인을 사용하여 그래프 탐색

<b>명시적 조인</b>
- join 키워드 직접 사용
- ex) select m from Member m join m.team t

<b>묵시적 조인</b>
- 경로 표현식에 의해 묵시적으로 SQL 조인 발생(내부 조인만 가능)
- ex) select m.team from Member m
</pre>
```sql
-- 단일 값 연관 경로 탐색(SQL 묵시적 조인 발생)
-- JPQL
select  m.team from Member m
-- SQL
select m.* 
from Member m 
inner join Team t on m.team.id = t.id
```
## 정리
<pre>
- 조인은 SQL 튜닝에 중요 포인트이다.
- 그런데 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어렵다.
- <b>그러므로 가급적 묵시적 조인 대신에 명시적 조인을 사용해야한다.</b>
</pre>