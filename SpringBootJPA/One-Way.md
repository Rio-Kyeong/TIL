# 연관관계 매핑
<pre>
<b>목표</b>
- 객체의 참조와 테이블의 외래 키를 매핑하는 법을 알 수 있다.

<b>용어</b>
- 방향(Direction) : <b>단방향</b>(한 쪽만 참조), <b>양방향</b>(양쪽 모두 서로 참조)
  (<b>방향은 객체관계에만 존재하고, 테이블은 항상 양방향이다</b>)
- 다중성(Multiplicity) : 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:M)
- 연관관계의 주인(Owner) : 객체 양방향 연관관계는 관리 주인을 정해야 한다.
</pre>
## 단방향 연관관계
<pre>
<b>객체 연관관계와 테이블 연관관계의 가장 큰 차이점</b>
참조를 통한 연관관계는 언제나 단방향이다. 객체간에 연관관계를 양방향으로 만들고 싶으면 반대쪽에도 필드를 추가해서 참조를 보관해야 한다.
결국 연관관계를 하나 더 만들어야 한다. 하지만 이것은 엄밀히 말하면 <b>양방향 연관관계가 아니라 서로 다른 단방향 관계가 2개인 것</b>이다.
반면 테이블은 외래 키 하나로 양방향으로 조인할 수 있다.

- <b>객체는 참조(주소)로 연관관계를 맺는다.</b>
- 테이블은 외래 키로 연관관계를 맺는다.
- <b>참조를 사용하는 객체의 연관관계는 단방향이다.</b>
- 외래키를 사용하는 테이블의 연관관계는 양방향이다.
</pre>
```java
// 단방향 관계(A -> B)
class A{
    B b;
}
class B{}
```
```java
// 양방향 관계(A <-> B)
class A{
    B b;
}
class B{
    A a;
}
```
### `객체 지향 모델링`
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/one-way.PNG"/>
- 회원(Member)과 팀(Team)이 있다.
- 회원은 하나의 팀에만 소속될 수 있다.
- 회원과 팀은 다대일 관계(N:1)이다.

<b>@JoinColumn</b>
- 외래키를 매핑할 때 사용한다.

<table>
<th>속성</th><th>기능</th><th>기본값</th>
<tr>
    <td>name</td><td>매핑할 외래 키 이름</td><td>필드명 + _ + 참조하는 테이블의 기본키 컬럼명</td>
</tr>
<tr>
    <td>referencedColumnName</td><td>외래키가 참조하는 대상 테이블의 컬럼명</td><td>참조하는 테이블의 기본키 컬럼명</td>
</tr>
</table>

<b>@ManyToOne</b>
- 다대일(N:1) 관계에서 사용한다.

<table>
<th>속성</th><th>기능</th><th>기본값</th>
<tr>
    <td>optional</td><td>false로 설정하면 연관된 엔티티가 항상 있어야 한다.</td><td>true</td>
</tr>
<tr>
    <td>fetch</td><td>글로벌 페치전략</td><td>@ManyToOne = fetchType.EAGER</br>@OneToMany = fetchType.LAZY</td>
</tr>
<tr>
    <td>cascade</td><td>연속성 전이 기능</td><td></td>
</tr>
</table>
</pre>
```java
@Entity 
@Getter @Setter 
public class Member { 
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID") 
    private String id; 
    
    private String username; 
    private int age;
    
    // 연관 관계 맵핑(단방향)
    @ManyToOne 
    @JoinColumn(name="TEAM_ID") 
    private Team team;
}
```
```java
@Entity 
@Getter @Setter 
public class Team { 
    @Id @GeneratedValue
    @Column(name = "TEAM_ID") 
    private String id; 
    
    private String name; 
}
```
### `연관관계 저장`
```java
//팀 저장
Team team = new Team();
team.setName("TeamA");
em.persist(team);

//회원 저장
Member member = new Member();
member.setName("member1");
member.setTeam(team); //단방향 연관관계 설정, 참조 저장
em.persist(member);
```
### `참조로 연관관계 조회 - 객체 그래프 탐색`
```java
//조회
Member findMember = em.find(Member.class, member.getId());

//참조를 사용해서 연관관계 조회
Team findTeam = findMember.getTeam();
```
### `연관관계 수정`
```java
// 새로운 팀B
Team teamB = new Team();
teamB.setName("TeamB");
em.persist(teamB);

// 회원1에 새로운 팀B 설정
member.setTeam(teamB);
```
