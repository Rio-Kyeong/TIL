# 다양한 연관관계 매핑
<pre>
<b>목표</b>
- 객체의 참조와 테이블의 외래 키를 매핑하는 법을 알 수 있다.
- <b>해당 글에서는 일대다(1:N), 일대일(1:1), 다대다(N:M)의 단방향과 양방향 연관관계를 소개한다.</b>

<b>용어</b>
- 방향(Direction) : <b>단방향</b>(한 쪽만 참조), <b>양방향</b>(양쪽 모두 서로 참조)
  (<b>방향은 객체관계에만 존재하고, 테이블은 항상 양방향이다</b>)
- 다중성(Multiplicity) : 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:M)
- 연관관계의 주인(Owner) : 객체 양방향 연관관계는 관리 주인을 정해야 한다.
</pre>
## 일대다(1:N)
<pre>
<b>일대다 단방향</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/1N.PNG"/>
- 일대다 단방향은 일대다(1:N)에서 <b>일(1)이 연관관계의 주인</b>
- 테이블 일대다 관계는 항상 <b>다(N) 쪽에 외래 키가 있음</b>
- 그러므로 객체와 테이블의 차이 때문에 <b>반대편 테이블의 외래 키를 관리하는 특이한 구조</b>
- @JoinColumn을 꼭 사용해야 함. 그렇지 않으면 <b>조인 테이블 전략</b>을 사용함(중간에 테이블을 하나 추가함)

<b>일대다 단방향 단점</b>
- 엔티티가 관리하는 외래 키가 다른 테이블에 있음.
- 본인 테이블에 FK가 있으면 insert 쿼리 한 번으로 끝나지만, 서로 다른 테이블에 있으므로 별도의 update sql을 추가로 실행한다.
- <b>일대다(1:N) 단방향 매핑보다는 다대일(N:1) 양방향 매핑을 사용하자</b>

<b>일대다 양방향</b>
- 이런 매핑은 공식적으로 존재X
- @JoinColumn(insertable=false, updatable=false)
- <b>읽기 전용 필드</b>를 사용해서 양방향 처럼 사용하는 방법
- <b>다대일(N:1) 양뱡향 매핑을 사용하자</b>

<b>요약</b>
- <b>일대다(1:N) 매핑보다는 다대일(N:1) 양뱡향 매핑을 사용하자</b>
</pre>
### `일대다(1:N) 단방향`
```java
@Entity
public class Team {
    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<>();
}
```
### `일대다(1:N) 양방향`
```java
@Entity
public class Team {
    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<>();
}
```
```java
@Entity
public class Member {
    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;
}
```
## 일대일(1:1)
<pre>
<b>일대일 단방향</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/11.PNG"/>
- 하나의 멤버는 하나의 락커를 가질 수 있다(<b>일대일(1:1)</b> 관계는 그 반대도 <b>일대일</b>)
- <b>외래 키에 데이터베이스 유니크 제약조건(UNI) 추가되어야 일대일 관계가 된다.</b>
- 테이블에서는 FK(외래 키)가 어느 쪽에 있든 조인하여서 조회할 수 있다.
- 그러므로 주 테이블이나 대상 테이블 중에 외래 키를 어디에 둘지 선택할 수 있다.
- <b>대상 테이블에 외래 키가 있을 때 일대일 단방향 매핑을 JPA에서 지원하지 않는다.</b>

<b>일대일 양방향</b>
1. 주 테이블에 외래 키 양방향 매핑
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/MainTable11.PNG"/>

2. 대상 테이블에 외래 키 양방향 매핑
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/TargetTable11.PNG"/>
- 대상 테이블에 외래 키가 있을 때 일대일 단방향 매핑을 JPA에서 지원하지 않기 때문에 일대일 앙방향 매핑을 해야한다.
  * LOCKER 테이블(대상 테이블)에 외래키가 있으므로 Member.locker를 외래키와 단방향 매핑을 할 수 없다.
  * 그러므로 양방향 매핑 후 Locker.member 와 LOCKER 테이블(대상 테이블)의 외래키를 대신 매핑시킨다.


<table>
<th>외래키 위치</th><th>설명</th><th>장점</th><th>단점</th>
<tr>
    <td>주 테이블</br>(많이 접근하는 테이블)</td><td>객체지향 개발자들이 선호하고</br>JPA 매핑이 편리하다.</td><td>주 테이블만 조회해도 대상 테이블에</br>데이터가 있는지 확인이 가능</td><td>값이 없으면 외래 키에 NULL을 허용해야 한다.</br>(DB입장에서는 치명적)</td>
</tr>
<tr>
    <td>대상 테이블</td><td>전통적인 DBA가 선호하는 방식</br>NULL을 허용해야 하는 문제도 없다.</td><td>주 테이블과 대상 테이블을</br>일대일에서 일대다 관계로 변경할 때</br>테이블 구조를 유지 가능</td><td>양방향 매핑을 해야 한다.</br>지연 로딩으로 설정해도 항상 즉시 로딩 된다.</td>
</tr>
</table>

<b>요약</b>
- <b>주 테이블(많이 접근하는 테이블)에 외래키를 두고 단방향 매핑을 하는것이 제일 좋다.</b>
  <b>(주 테이블에 외래키를 둘 수 없는 상황이라면 양방향 매핑을 한다)</b>
</pre>
### `주 테이블에 외래 키 일대일(1:1) 단방향`
```java
@Entity
public class Member {
   @OneToOne
   @JoinColumn(name = "locker_id")
   private Locker locker;
}
```
### `주 테이블에 외래 키 일대일(1:1) 양방향`
```java
@Entity
public class Member {
   @OneToOne
   @JoinColumn(name = "locker_id")
   private Locker locker;
}
```
```java
@Entity
public class Locker {
    // 읽기 전용
    @OneToOne(mappedBy = "locker")
    private Member member;
}
```
## 다대다(N:M)
<pre>
<b>다대다(N:M)</b>
- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없음
- 그러므로 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어내야한다.
- <b>객체는 컬렉션을 사용해서 객체 2개로 다대다 관계를 만들 수 있다.</b>

<b>다대다 매핑의 한계</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/NM.PNG"/>
- 회원(Member)는 여러 상품(Product)을 가질 수 있으며, 상품 또한 여러 회원이 가질 수 있다.
- <b>@ManyToMany</b> 사용, <b>@JoinTable</b>로 연결 테이블 지정
- <b>편리해 보이지만 실무에서 사용하지 않는다.</b>
- 연결 테이블이 단순히 연결만 하고 끝나지 않는다.
- 주문시간, 수량 같은 데이터가 들어올 수 있다.

<b>다대다 한계 극복</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/NM2.PNG"/>
- 연결 테이블용 엔티티를 추가한다(연결 테이블을 엔티티로 승격)
- <b>@ManyToMany -> @OneToMany, @ManyToOne</b>
- 위에 다대다 매핑의 한계 첨부 그림에서는 MemberProduct의 MEMBER_ID, PRODUCT_ID를 묶어서 PK로 썻지만
  실제로는 아래 처럼 <b>독립적으로 generated되는 id를 사용하는 것을 권장</b>한다.
- 독립적으로 사용함으로써 ID가 두개의 테이블에 종속되지 않고 더 유연하게 개발할 수 있다.

<b>요약</b>
- <b>다대다(N:M) 매핑은 문제가 많기 때문에 왠만하면 실무에서는 사용하지 않는다.</b>
- 테이블의 N:M 관계는 중간 테이블을 이용해서 1:N, N:1 관계로 풀어낸다.
</pre>
### `다대다(N:M) 한계 극복`
```java
@Entity
public class Member{
  @OneToMany(mappedBy = "member")
  private List<MemberProduct> MemberProducts1 = new ArrayList<>();
}
```
```java
@Entity
public class Product{
  @OneToMany(mappedBy = "product")
  private List<MemberProduct> MemberProducts2 = new ArrayList<>();
}
```
```java
@Entity
public class MemberProduct{
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
​
  @ManyToOne
  @JoinColumn(name = "member_id")
  private Member member;
​
  @ManyToOne
  @JoinColumn(name = "product_id")
  private Product product;
}
```