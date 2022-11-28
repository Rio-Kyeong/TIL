# PROXY(EAGER AND LAZY)
<pre>
<b>문제점</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/proxyEntity.PNG"/>
- 연관관계가 있는 경우 엔티티를 조회할 때 연관관계를 맺고있는 다른 엔티티도 같이 조회가 된다.
- 하지만 항상 연관된 엔티티 정보가 필요한 것은 아니므로 불필요한 데이터베이스 조회가 생기는 것이다.
  - Member Entity의 정보만 필요 하여도 연관관계 때문에 불필요하게 Team Entity의 정보도 같이 조회가 된다.

<b>해결</b>
- 즉시로딩(EAGER)이 아닌 <b>프록시를 이용한 지연로딩(LAZY)을 사용</b>해야 한다.
</pre>
## 프록시
<pre>
<b>프록시(Proxy)란?</b>
- 실제 엔티티 객체 대신에 사용되는 객체
- <b>실제로 값이 사용되는 시점에 초기화를 하여 값을 실제 엔티티에서 가져온다.</b>

<b>프록시(Proxy) 특징</b>
- 실제 클래스(Entity)를 상속 받아서 만들어졌다.
- 실제 클래스와 겉 모양이 같다.
- 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 됨(이론상)

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/proxyTarget.PNG"/>
- 프록시 객체는 실제 객체의 참조(target)을 보관한다.
- 프록시의 참조(target)는 최초 null 값을 가진다.
- 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출

<b>em.find()</b> : DB를 통해서 실제 엔티티 객체 조회, 객체 조회 시점에 SELECT QUERY를 날린다.
<b>em.getReference() : DB 조회를 미루는 가짜(프록시) 엔티티 객체 조회, 프록시 조회는 실제 사용 시점에 SELECT QUERY를 날린다.</b>
</pre>
## 프록시 초기화
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/proxyReset.PNG"/>
최초 em.getReference()를 통해서 프록시 객체를 가져온다.
1. 프록시가 사용되는 시점에 값(getName())이 프록시 객체에 있는지 target 을 확인한다.
2. 없을 경우 JPA가 영속성 컨텍스트에 초기화 요청을 한다. (있을 경우 조회 없이 그대로 반환)
3. 영속성 컨텍스트는 직접 DB를 조회한다.
4. 조회해서 실제 엔티티를 생성한다.
5. 생성된 실제 엔티티를 프록시 객체의 target에 연결을 시켜준다.

<b>프록시(Proxy) 특징</b>
- 프록시 객체는 처음 사용할 때 한 번만 초기화
- 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아님
  (초기화되면 프록시 객체를 통해서 실제 엔티티에 접근 가능)
- 프록시 객체는 원본 엔티티를 상속받음, 따라서 타입 체크시 주의해야함
  (== 비교 실패, 대신 instance of 사용)
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환
  (반대로 프록시가 이미 존재할 경우 em.find()를 호출해도 프록시가 반환)
- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면 LazyInitializationException 발생
  (프록시를 호출 후 em.detach() 또는 em.close() 등으로 프록시가 준영속 상태가 되었는데 프록시를 사용(초기화)하면 에러 발생)
</pre>
### `영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환`
<pre>
- JPA는 같은 트랜잭션 안에서 동일한 PK(식별자) 값으로 조회(find) 하면 같은 객체 반환을 보장해 준다.
- 이미 find()로 엔티티를 찾아서 영속성 컨텍스트에 엔티티 정보가 존재하는 경우, getReference()로 호출해도 실제 엔티티를 반환한다.
- 반대로 reference()를 먼저 호출하고 그 뒤에 find()를 호출한다면 둘 다 프록시(Proxy)객체를 반환한다.
</pre>
```java
tx.begin(); // Transaction 시작

Member member = new Member();
member.setName("member1");
em.persist(member);

// em.flush(), em.clear()를 하면 DB에 데이터를 반영하고, 영속성 컨텍스트를 지운다(실습을 위해)
em.flush();
em.clear();

Member m = em.find(Member.class, member.getId());
System.out.println("m = "+ m.getClass()); // m = class hellojpa.Member

Member reference = em.getReference(Member.class, member.getId());
System.out.println("reference : "+ reference.getClass()); // reference : class hellojpa.Member

// 동일한 엔티티 반환을 보장한다.
System.out.println("m == reference : "+ (m == reference)); // m == reference : true

tx.commit(); // Transaction 저장
```
### `영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면 문제 발생`
<pre>
- 프록시(Proxy) 객체가 더 이상 영속성 컨텍스트의 도움을 받을 수 없을 때(준영속 상태), 프록시 객체를 초기화하면 에러가 발생한다.
- 보통 트랜잭션이 끝나고 프록시를 조회할 때 많이 발생한다.
</pre>
```java
Member member = new Member();
member.setName("member1");

em.persist(member);

// em.flush(), em.clear()를 하면 DB에 데이터를 반영하고, 영속성 컨텍스트를 지운다.
em.flush();
em.clear();

Member reference = em.getReference(Member.class, member.getId());
System.out.println("reference : "+ reference.getClass()); //Proxy

// 영속성 컨텍스트에서 분리해 준영속 상태로 전환
em.detach(reference);

// 프록시 초기화
// could not initialize proxy - no Session
String name = reference.getName();
```
## 프록시 확인
<pre>
- 프록시 확인을 도와주는 유틸리티 메서드

<b>프록시 인스턴스의 초기화 여부 확인</b>
PersistenceUnitUtil.isLoaded(Object proxy)

<b>프록시 클래스 확인 방법</b>
proxy.getClass().getName() - 출력(..javasist.. or HibernateProxy…)

<b>프록시 강제 초기화</b>
org.hibernate.Hibernate.initialize(proxy);

참고 : JPA 표준은 강제 초기화가 없기 때문에 강제 호출을 해야한다(하이버네이트만 존재)
강제 호출 : member.getName(); // 강제 호출을 하려고 멤버의 이름을 호출..? 조금 이상하다.
</pre>
```java
Member member = new Member();
member.setName("member1");

em.persist(member);

// em.flush(), em.clear()를 하면 DB에 데이터를 반영하고, 영속성 컨텍스트를 지운다.
em.flush();
em.clear();

// 프록시 객체 생성
Member reference = em.getReference(Member.class, member.getId());

// 프록시 초기화 전 : isLoaded : false
System.out.println("isLoaded : "+ EntityManagerFactory.getPersistenceUnitUtil().isLoaded(reference));

// 프록시 초기화 후 : isLoaded : true
String name = reference.getName(); //프록시 강제 호출(초기화)
System.out.println("isLoaded : "+ EntityManagerFactory.getPersistenceUnitUtil().isLoaded(reference));

// 프록시 클래스 확인 : reference : hellojpa.Member$HibernateProxy$jGqvAuf9
System.out.println("reference : "+ reference.getClass().getName());

// 하이버네이트 프록시 강제 초기화
Hibernate.initialize(reference);
```
## 즉시 로딩과 지연 로딩
<pre>
<b>즉시 로딩(EAGER)</b>이란 객체 A를 조회할 때 A와 연관된 객체들을 한 번에 가져오는 것이다.
<b>지연 로딩(LAZY)</b>이란 객체 A를 조회할 때는 A만 가져오고 연관된 애들은 저번 게시글에서 본 프락시 초기화 방법으로 가져온다.

- 즉시 로딩(EAGER)은 예측이 어렵고, 어떤 SQL이 실행될지 예상하기 어렵다.
- 즉시 로딩(EAGER)은 JPQL에서 N+1 문제가 자주 발생한다.
- 실무에서 <b>모든 연관관계는 지연로딩(LAZY)으로 설정</b>해야 한다.
- 연관된 엔티티를 함께 DB에서 조회해야 하면, <b>fetch join</b> 또는 <b>엔티티 그래프 기능</b>을 사용하여 한 번에 데이터를 가져올 수 있다.

@xxxToMany 처럼 Many로 끝나는 관계는 기본 값이 지연로딩(LAZY)이다.
@xxxToOne 처럼 <b>One으로 끝나는 관계는 기본 값으로 즉시로딩(EAGER)</b>이므로 직접 지연로딩(LAZY)으로 변경해야 한다.

결론
- 실무에서는 즉시 로딩(EAGER)은 피하고 지연 로딩(LAZXY)을 사용한다.
- 한 번에 연관된 엔티티들의 데이터를 조회해야 할 때는 페치 조인(fetch join)을 이용한다.
- 만약 MEMBER 와 TEAM 이 항상 같이 조회한다면 즉시 로딩(EAGER)이 더 효과적일 수 있다.
</pre>
### `지연 로딩(LAZY)`
```java
@Entity
public class Member {

    // @XToOne으로 끝나는 관계는 모두 LAZY로 변경한다.
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```
### `지연 로딩 LAZY을 사용해서 프록시로 조회`
```java
// 팀 생성
Team team = new Team();
team.setName("team1");
team.setYear(1998);
em.persist(team);

// 회원 생성
Member member = new Member();
member.setName("member1");
em.persist(member);

// DB 에 데이터를 반영
// 실습을 위해 영속성 컨텍스트에서 데이터 삭제
em.flush();
em.clear();

// LAZY 설정으로 Member Entity 정보만 조회
// memberFind = hellojpa.Member
Member memberFind = em.find(Member.class, member.getId());
System.out.println("memberFind = "+ memberFind.getClass().getName());

// 현재 Team 객체는 LAZY 로딩으로 프록시 객체
// team = hellojpa.Team$HibernateProxy$ZQml37oo
Team teamRef = memberFind.getTeam();
System.out.println("team = "+ teamRef.getClass().getName());

// 프록시 초기화(최초 1회 Team SELECT 쿼리 발생)
// 실제 team을 사용하는 시점에 초기화 (DB 조회)
String name = teamRef.getName();
System.out.println("TeamName : "+ name);

// 이후 데이터 조회 시 초기화된 프록시 객체에서 값을 가져온다 (쿼리 발생X)
int year = teamRef.getYear();
System.out.println("TeamYear : "+ year);
```
