# 연관관계 매핑
<pre>
<b>목표</b>
- 객체의 참조와 테이블의 외래 키를 매핑하는 법을 알 수 있다.
- <b>해당 글에서는 다대일(N:1)의 단방향과 양방향 연관관계를 소개한다.</b>

<b>용어</b>
- 방향(Direction) : <b>단방향</b>(한 쪽만 참조), <b>양방향</b>(양쪽 모두 서로 참조)
  (<b>방향은 객체관계에만 존재하고, 테이블은 항상 양방향이다</b>)
- 다중성(Multiplicity) : 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:M)
- 연관관계의 주인(Owner) : 객체 양방향 연관관계는 관리 주인을 정해야 한다.
</pre>
## 단방향 연관관계
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
- 속성에 mappedBy가 없다(무조건 연관관계의 주인이 된다는 뜻)
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
<tr>
    <td>targetEntity</td><td>연관된 엔티티의 타입 정보를 설정한다.</br>(이 기능은 거의 사용하지 않는다)</td><td></td>
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
## 객체 연관관계와 테이블 연관관계의 가장 큰 차이점
<pre>
참조를 통한 연관관계는 언제나 단방향이다. 객체간에 연관관계를 양방향으로 만들고 싶으면 반대쪽에도 필드를 추가해서 참조를 보관해야 한다.
결국 연관관계를 하나 더 만들어야 한다. 하지만 이것은 엄밀히 말하면 <b>양방향 연관관계가 아니라 서로 다른 단방향 관계가 2개인 것</b>이다.
반면 테이블은 외래 키 하나로 양방향으로 조인할 수 있다.

<b>객체</b>
- 객체는 참조(주소)로 연관관계를 맺는다.
- 참조를 사용하는 객체의 연관관계는 단방향이다.
  (객체 양방향 관계는 A->B, B->A 처럼 <b>참조가 2군데</b>)
- 객체 양방향 관계는 참조가 2군데 있음. 둘중 테이블의 외래 키를 관리할 곳을 지정해야 한다.
  * <b>연관관계의 주인</b> : 외래 키를 관리하는 참조
  * <b>주인의 반대편</b> : 외래 키에 영향을 주지 않음, 단순 조회만 가능

<b>테이블</b>
- 테이블은 <b>외래 키 하나</b>로 두 테이블이 연관관계를 맺는다.
- 외래 키 하나로 양쪽 조인이 가능하며, 방향이라는 개념이 없다.
  (A <b>JOIN</b> B , B <b>JOIN</b> A 둘다 가능하며 결과값도 같다)
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
## 양방향 연관관계
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/many-way.PNG"/>
- 양방향 연관관계는 <b>서로 다른 단방향 관계가 2개</b>인 것이다.
  - 객체 연관관계 = 2개
    - 회원 -> 팀 연관관계 1개(단방향)
    - 팀 -> 회원 연관관계 1개(단방향)
  - 테이블 연관관계 = 1개
    - 회원 <- JOIN -> 팀 연관관계 1개(양방향)
- 둘 중 하나로 외래 키를 관리해야 한다.
- 회원과 팀은 다대일 관계(N:1)이다.
- <b>Team객체의 필드에서는 List를 통해서 여러 명의 Member를 조회하고, Member객체의 필드에서는 참조를 통해서 하나의 Team을 조회한다.</b>

<b>양방향 매핑 규칙</b>
- 객체의 두 관계중 하나를 연관관계의 주인으로 지정해야 한다.
- <b>연관관계의 주인만이 외래 키를 관리(등록, 수정)</b>
- <b>주인이 아닌쪽은 읽기만 가능</b>
- 주인은 mappedBy 속성 사용X
- 주인이 아니면 mappedBy 속성으로 주인 지정

<b>@OneToMany</b>
- 일대다(1:N) 관계에서 사용한다.
<table>
<th>속성</th><th>기능</th><th>기본값</th>
<tr>
    <td>mappedBy</td><td>연관관계의 주인 필드를 선택한다.</td><td></td>
</tr>
<tr>
    <td>fetch</td><td>글로벌 페치전략</td><td>@ManyToOne = fetchType.EAGER</br>@OneToMany = fetchType.LAZY</td>
</tr>
<tr>
    <td>cascade</td><td>연속성 전이 기능</td><td></td>
</tr>
<tr>
    <td>targetEntity</td><td>연관된 엔티티의 타입 정보를 설정한다.</br>(이 기능은 거의 사용하지 않는다)</td><td></td>
</tr>
</table>

<b>요약</b>
- <b>복잡도가 늘어나기 때문에 가급적 설계할 때는 단방향으로 맵핑을 하고 개발하다가 필요시 양방향을 추가 하는것이 좋다.</b>
- <b>연관관계의 주인은 외래 키가 있는 곳(다(N)쪽)을 주인으로 정한다.</b>
- <b>양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐이므로 읽기(조회)만 가능하다.</b>
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
    
    // 연관 관계 맵핑(양방향)
    @ManyToOne 
    @JoinColumn(name="TEAM_ID") 
    private Team team;
}
```
```java
@Entity
@Setter @Getter
public class Team {
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    private String name;

    // 연관 관계 맵핑(양방향)
    // List는 꼭 필드에서 초기화
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    //연관관계 편의 메소드
    public void addMember(Member member){
        member.setTeam(this);
        this.members.add(member);
    }
}
```
### `양방향 매핑 시 무한루프 주의`
<pre>
<b>양방향 매핑시에 무한 루프를 주의해야 한다.</b>

<b>lombok이 자동으로 만드는 toString()을 사용하지 말자(재정의하여 사용)</b>
- 양방향 매핑 후 toString()을 호출하는 순간 서로간의 관계로 인해 무한루프 생성(스택오버플로우 발생)

<b>JSON 생성 라이브러리</b>
- 양방향 관계의 엔티티를 JSON으로 시리얼라이즈 하는순간 무한루프에 빠져버린다.
- <b>컨트롤러에서는 엔티티를 절대 직접 반환하지 말자(DTO로 변환해서 반환)</b>
</pre>
### `양방향 매핑 시 값 셋팅 주의`
```java
//잘못된 예시(연관관계의 주인에 값을 입력하지 않음)

//1. 새 팀 생성
Team team = new Team();
team.setName("TeamA");
em.persist(team);

//2. 새 멤버 생성
Member member = new Member();
member.setName("member1");

//3. 팀에 멤버 저장(add)
team.getMembers().add(member);
em.persist(member);
```
<pre>
- 코드 실행 시, 새 팀과 새 멤버는 생성이 되지만 멤버와 팀의 연관관계는 저장되지 않는다.
- 연관관계의 주인은 Member인데, 이 연관관계를 Team에 지정해주었기 때문이다.
- Team 객체의 members List 는 읽기 전용이다.

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/example1.PNG"/>
DB에서도 null을 확인할 수 있다.
</pre>
```java
// 연관관계의 주인에 값 설정

//1. 새 팀 생성
Team team = new Team();
team.setName("TeamA");
em.persist(team);

//2. 새 멤버 생성
Member member = new Member();
member.setName("member1");

//팀에도 연관관계 멤버 값 설정
team.getMembers().add(member);
//연관관계의 주인에 값 설정
member.setTeam(team);
em.persist(member);
```
<pre>
<b>요약</b>
<b>- 양방향 연관관계에서는 연관관계의 주인에 값을 입력해야 한다.</b>
<b>- 순수한 객체 관계를 고려해서 항상 양쪽다 값을 설정해야 한다.</b>
  - Team의 members와 Member의 team에 모두 값을 셋팅하였다.
  - 만약 member.setTeam(team) 만 할 경우 flush 되기 전까지는 Team 객체에는 members List 값이 없는 상태이다.
  - 연관관계 편의 메소드를 생성해서 한 번에 연관관계를 설정하는 것도 좋은 방법이다.

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/example2.PNG"/>
- 연관관계의 주인에 값을 설정함으로써 DB에는 새로운 멤버와 팀, 연관관계가 모두 정상적으로 저장된다.
</pre>
### `연관관계 편의 메서드 작성`
<pre>
<b>연관관계 편의 메서드</b>
<b>- 양방향 연관관계를 한번에 설정하는 편리한 메서드</b>
- 양쪽다 값 셋팅을 위해 매번 반복적인 코드를 작성해야한다. 그러므로 연관관계 편의 메소드를 생성하여 사용하자.</b>
- 엔티티 A와 B가 서로 양방향 연관관계인데, 어디에 연관관계 편의 메서드를 두는게 좋을까?
    -> JPA의 영역이라기 보다는 오히려 객체지향 설계의 영역이기 때문에 어디에 두든 정답은 없다.
</pre>
```java
@Entity
public class Team {
    //연관관계 편의 메서드
    public void addMember(Member member){
        member.setTeam(this);
        this.members.add(member);
    }
}
```
```java
//1. 새 팀 생성
Team team = new Team();
team.setName("TeamA");
em.persist(team);

//2. 새 멤버 생성
Member member = new Member();
member.setName("member1");
em.persist(member);

//3. 연관관계 편의 메서드 사용
// setter method 는 지양하는 편
team.addMember(member);
```
