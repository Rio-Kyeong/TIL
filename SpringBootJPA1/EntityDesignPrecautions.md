# Entity Design Precautions
## 엔티티는 가급적 Setter를 사용하지 말자
<pre>
- Setter가 모두 열려있으면 변경 포인트가 너무 많아서, 유지보수가 어렵다(나중에 리펙토링으로 Seeter 제거)
</pre>
## 실무에서는 @ManyToMany 를 사용하지 말자
<pre>
- @ManyToMany 는 편리한 것 같지만, 중간 테이블에 컬럼을 추가할 수 없고,
  세밀하게 쿼리를 실행하기 어렵기 때문에 실무에서 사용하기에는 한계가 있다.

- 중간 엔티티를 만들고 @ManyToOne , @OneToMany 로 매핑해서 사용하자.
  정리하면 대다대 매핑을 일대다, 다대일 매핑으로 풀어내서 사용하자.
</pre>
## 모든 연관관계는 지연로딩(LAZY)으로 설정
<pre>
<b>즉시 로딩(EAGER)</b>이란 객체 A를 조회할 때 A와 연관된 객체들을 한 번에 가져오는 것이다.
<b>지연 로딩(LAZY)</b>이란 객체 A를 조회할 때는 A만 가져오고 연관된 애들은 저번 게시글에서 본 프락시 초기화 방법으로 가져온다.

- 즉시로딩(EAGER)은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다. 특히 JPQL을 실행할 때 N+1 문제가 자주 발생한다.
- 실무에서 <b>모든 연관관계는 지연로딩(LAZY)으로 설정</b>해야 한다.
- 연관된 엔티티를 함께 DB에서 조회해야 하면, <b>fetch join 또는 엔티티 그래프 기능을 사용하여 한 번에 데이터를 가져올 수 있다.</b>

@xxxToMany 처럼 Many로 끝나는 관계는 기본 값이 지연로딩(LAZY)이다.
@xxxToOne 처럼 <b>One으로 끝나는 관계는 기본 값으로 즉시로딩(EAGER)</b>이므로 직접 지연로딩(LAZY)으로 변경해야 한다.

결론
- 즉시 로딩은 가급적 피하고 지연 로딩(LAZXY)을 사용한다.
- 한 번에 연관된 엔티티들의 데이터를 조회해야 할 때는 페치 조인(fetch join)을 이용한다.
</pre>
### LAZY(지연 로딩)
```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "team_id")
private Team team;
```
## 컬렉션은 필드에서 초기화 하자
<pre>
- 컬렉션(JFC)은 필드에서 바로 초기화 하는 것이 null 문제에서 안전하다.
  (간혹 생성자에서 초기화를 하는 경우가 있는데 좋지않다.)
- 하이버네이트는 엔티티를 영속화 할 때, 컬렉션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경한다.
  만약 getOrders() 처럼 임의의 메서드에서 컬렉션을 잘못 생성하면 하이버네이트 내부 메커니즘에 문제가 발생할 수 있다.
  따라서 필드레벨에서 생성하는 것이 가장 안전하고, 코드도 간결하다.
</pre>
### 컬렉션 초기화
```java
@Entity
@Getter @Setter
public class Member {
    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}
```
```java
// 컬렉션
Member member = new Member();
System.out.println(member.getOrders().getClass());

// 영속화한 컬렉션
em.persist(team);
System.out.println(member.getOrders().getClass());

// 출력 결과
// 하이버네이트가 제공하는 내장 컬렉션으로 변경되어 다른 결과가 나온다.
class java.util.ArrayList
class org.hibernate.collection.internal.PersistentBag
```
## 테이블, 컬럼명 생성 전략
<pre>
스프링 부트에서 하이버네이트 기본 매핑 전략을 변경해서 실제 테이블 필드명은 다르다.

Hibernate는 물리적 전략과  암시적 전략을 사용하여 필드 이름을 매핑한다.

1. 논리명 생성 : 명시적으로 컬럼, 테이블 명을 직접 적지 않으면  ImplicitNamingStrategy 사용
spring.jpa.hibernate.naming.implicit-strategy : 테이블이나, 컬럼명을 명시하지 않을 때 논리명 적용

2. 물리명 적용:
spring.jpa.hibernate.naming.physical-strategy : 모든 논리명에 적용됨, 실제 테이블에 적용
(username usernm 등으로 회사 룰로 바꿀 수 있음)

스프링 부트 신규 설정 (엔티티(필드) -> 테이블(컬럼))
1. 카멜 케이스 -> 언더 스코어(memberPoint member_point)
2. .(점) -> _(언더스코어)
3. 대문자 -> 소문자

예를 들어  AddressBook  엔터티는 address_book  테이블 로 생성된다.
</pre>