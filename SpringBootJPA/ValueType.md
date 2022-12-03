# Data Type
<pre>
<b>JPA의 데이터 타입 분류</b>
<b>엔티티 타입</b>
- @Entity로 정의하는 객체
- 데이터가 변해도 식별자(@Id)로 지속해서 추적 가능
- 예) 회원 엔티티의 키나 나이 값을 변경해도 <b>식별자로 인식 가능</b>

<b>값 타입</b>
- int, Integer, String 처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체
- <b>식별자가 없고 값만 있으므로 변경시 추적 불가능(값 그 자체)</b>
  예) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체
- <b>생명주기를 엔티티에 의존한다.</b>
  예) 회원을 삭제하면 이름, 나이 필드도 함께 삭제
- <b>값 타입은 여러 곳에서 공유하면 위험하다.</b>
  예) 회원 이름 변경시 다른 회원의 이름도 함께 변경되면 안됨
- 값 타입으로는 <b>기본값 타입, 임베디드 타입, 컬렉션 값 타입</b>이 있다.
    
  <b>기본값 타입</b>
  - 자바 기본 타입(int, double)
  - 래퍼 클래스(Integer, Long)
  - String
  
    <b>참고 : 자바의 기본 타입(Primitive Type)은 절대 공유되지 않는다.</b>
    - int, double 같은 기본 타입은 공유되지 않으므로 값타입으로 썼을 때 안전하다.
    - 기본 타입(Primitive Type)은 항상 값을 복사함 (공유X)
    - Integer 같은 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능한 객체이지만 변경을 할 수 없다 (불변 객체)
    - <b>결론 : 기본 타입이나 불변 객체는 공유 걱정 없이 복사해서 안전하게 사용 가능함</b>

  <b>임베디드 타입</b>(embedded type, 복합 값 타입) - Value Object (VO)
  - 암배디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 공유한 값이 참조되므로 위험하다.
  - 그러므로 불변 객체(immutable object)로 설계해야 한다.

  <b>컬렉션 값 타입</b>(collection value type) - Java Collection Framework
  - 컬렉션에 기본 값 타입이나 임베디드 타입을 넣어서 사용한다.
</pre>
```java
// 자바의 기본 타입(Primitive Type)은 항상 값을 복사함으로 공유가 안됨(안전)
int a = 10;
int b = a; // 값 복사 - 

a = 20;

System.out.println("a = "+ a); // a = 20
System.out.println("b = "+ b); // b = 10

// Wrapper Class는 주소(참조)값이 넘어가기 때문에 공유가 됨 하지만 값을 변경할 방법이 없음(안전)
// 하지만 변경할 수 있는 직접 정의한 객체(임베디드 타입)같은 경우에는 불변객체로 만들어서 변경을 못하도록 해야한다(주의)
// 만약 데이터를 변경할 수 있다면 b변경 시 a도 동일하게 변경된다(side effect 발생)
Integer a = Integer.valueof(10);
Integer b = a; // 참조 값 복사 - 공유됨

// 래퍼 클래스(Wrapper class)는 산술 연산을 위해 정의된 클래스가 아니므로, 인스턴스에 저장된 값을 변경할 수 없습니다.
// 단지, 값을 참조하기 위해 새로운 인스턴스를 생성하고, 생성된 인스턴스의 값만을 참조할 수 있습니다.
```
## 값 타입과 불변 객체
<pre>
- 임베디드 타입의 경우 여러 엔티티에서 공유하면 위험하다.
- 부작용(side effect) 발생
- side effect는 컴파일러 단계에서 추적이 불가능하기 때문에 찾기 매우 어렵다.
</pre>
```java
Address address = new Address("city", "street", "123456");

Member member = new Member();
member.setUsername("member1");
member.setHomeAddress(address); // 동일한 인스턴스 공유
em.persist(member);

Member member2 = new Member();
member2.setUsername("member2");
member2.setHomeAddress(address); // 동일한 인스턴스 공유
em.persist(member2);

// member의 city 만 변경하기를 원하였지만 member2의 city까지 같이 변경된다(side effect 발생)
// 값 타입의 실제 인스턴스인 값을 공유하는 것은 위험하다.
member.getHomeAddress().setCity("newCity");
```
<pre>
- 값 타입의 실제 인스턴스인 값을 공유하는 것은 위험함으로 <b>대신 값(인스턴스)를 깊은 복사(Deep copy)를 통해 사용</b>해야한다.
  EX) Address newAddress = new Address(address.getCity(), address.getStreet(), address.getZipcode());
- 하지만 임베디드 타입은 객체임으로 객체의 공유참조는 피할 수 없다.
- 그러므로 값을 생성자 이외에 변경할 수 없도록 <b>불변 객체</b>로 만들어야 한다.

<b>값 타입은 불변 객체(immutable object)로 설계해야한다.</b>
- 임베디드 타입 같은 객체 타입은 <b>기본 값 타입의 값 복사(Call By Value)</b>와 다르게 <b>참조 값을 복사(Call By Reference)</b> 하기 때문에,
  어느 한쪽이 변경되면, 참조 주소가 변경되어 복사된 모든 객체에 영향을 미친다.
- 그러므로 객체 타입을 수정할 수 없게 불변 객체를 만들어서 부작용(side effect)을 원천 차단해야한다.

<b>불변 객체</b>
- 생성 시점 이후 절대 값을 변경할 수 없는 객체
- <b>생성자로만 값을 설정하고 수정자(Setter)를 만들지 않는다(또는 객체 내부에서 사용한다면 private setter)</b>
- 참고 : Integer, String 은 자바가 제공하는 대표적인 불변 객체
</pre>
`임베디드 타입의 데이터 변경 방법`
```java
Address address = new Address("city", "street", "123456");

Member member = new Member();
member.setUsername("member1");
member.setHomeAddress(address);
em.persist(member);

// 새로운 임베디드 타입의 객체를 만들고 member 객체에 저장해준다.
// (Shallow copy 가 아닌 Deep copy 를 이용)
// city의 값만 바꾸더라도 모든 데이터를 새로 만들어서 넣어야한다.
// member 테이블만 update한다. 사실 member 엔티티를 수정하는 것과 같다.
member.setHomeAddress(new Address("NewCity", address.getStreet(), address.getZipcode()));
```
## 값 타입의 비교
<pre>
값 타입은 인스턴스 주소가 달라도 그 안의 <b>값</b>이 같다면 같은 것으로 봐야 한다.
- <b>동일성(identity) 비교 : 인스턴스의 참조 값을 비교, == 사용</b>
- <b>동등성(equivalence) 비교 : 인스턴스의 값을 비교, equals(obj) 사용</b>

<b>Equals()</b>
- 임베디드 타입도 값 타입이기 때문에 이를 만족해야 한다.
- 객체를 따로생성하고 == 으로 비교한다면, 어김없이 False를 반환한다.
- 따라서 임베디드 타입이라면 <b>equals() 메소드를 적절하게 재정의</b>하여 같은 값을 갖는지 확인해야 한다.
</pre>
## 임베디드 타입(Value Object: VO)
<pre>
<b>Embedded Type</b>
- 새로운 값 타입을 직접 정의할 수 있음
- JPA는 임베디드 타입(embedded type)이라 한다.
- <b>주로 기본 값 타입을 모아서 만들어서 복합 값 타입이라고도 한다.</b>

<b>@Embeddable</b> : 값 타입을 정의하는 곳에 표시
<b>@Embedded</b> : 값 타입을 사용하는 곳에 표시
- 기본 생성자 필수

<b>임베디드 타입의 장점</b>
- 재사용성 증가
- 높은 응집도
- Period.isWork()처럼 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수 있음
- 임베디드 타입을 포함한 모든 값 타입은, <b>값 타입을 소유한 엔티티에 생명주기를 의존</b>함

<b>임베디드 타입과 테이블 매핑</b>
해당 회원(Member) 엔티티는 임베디드 타입 Period와 Address를 가진다.
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/EmbeddedType.PNG"/>
- <b>임베디드 타입은 엔티티의 값을 뿐이다.</b>
- 임베디드 타입은 사용하기 전과 후에 <b>매핑하는 테이블은 같다.</b>
- 객체와 테이블은 아주 세밀하게(find-grained) 매핑하는 것이 가능
- 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많음

<b>임베디드 타입과 연관관계</b>
- 엔티티는 임베디드 타입을 가질 수 있다.
- 임베디드 타입 클래스는 또 다른 임베디드 타입을 가질 수 있다.
- 임베디드 타입 클래스는 다른 엔티티를 가질 수 있다.

<b>@AttributeOverride : 속성 재정의</b>
- 하나의 엔티티에서 같은 임베디드 타입을 사용하면 컬럼 명이 중복되어 오류가 난다.
- <b>@AttributeOverrides, @AttributeOverride</b>를 사용해서 컬럼 명 속성을 재정의
</pre>
```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;

    // 기간 Period (임베디드 타입)
    @Embedded
    private Period period;

    // 집 주소 (임베디드 타입)
    @Embedded
    private Address homeAddress;

    // 속성 재정의
    // 회사 주소 (임베디드 타입)
    @Embedded
    @AttributeOverrides({
            @AttributeOverride(name = "city",
                               column = @Column(name = "COMPANY_CITY")),
            @AttributeOverride(name = "street",
                               column = @Column(name = "COMPANY_STREET")),
            @AttributeOverride(name = "zipcode",
                               column = @Column(name = "COMPANY_ZIPCODE"))
    })
    private Address CompanyAddress;
}
```
```java
@Embeddable
@Getter
public class Period {

    private LocalDateTime startDate;
    private LocalDateTime endDate;

    public void isWork(){
        // 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수도 있음
    }

    // 기본 생성자 필수
    public Period() {
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Period period = (Period) o;
        return Objects.equals(startDate, period.startDate) && 
               Objects.equals(endDate, period.endDate);
    }

    @Override
    public int hashCode() {
        return Objects.hash(startDate, endDate);
    }
}
```
```java
@Embeddable
@Getter
public class Address {

    @Column(length = 10) // 주소 글자수 제한
    private String city;

    @Column(length = 10)
    private String street;

    @Column(length = 10)
    private String zipcode;
    
    // 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수도 있음
    public String fullAddress(){
        return getCity()+" "+getStreet()+" "+getZipcode();
    }
    
    // 기본 생성자 필수
    public Address() {
    }
  
   // Use getters during code generation
   @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        return Objects.equals(getCity(), address.getCity()) && Objects.equals(getStreet(), 
                address.getStreet()) && Objects.equals(getZipcode(), address.getZipcode());
    }

    @Override
    public int hashCode() {
        return Objects.hash(getCity(), getStreet(), getZipcode());
    }
}
```
## 값 타입 컬렉션
<pre>
<b>값 타입 컬렉션</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/ValueTypeCollection.PNG"/>
- 값 타입을 하나 이상 저장하고자 할 때 사용하며, 자바의 컬렉션(JCF)을 사용한다.
- PK를 하나만 사용하면 값 타입이 아니고, 엔티티 개념이 되어버리기 때문에 <b>값 타입을 하나로 묶어서 PK로 설정</b>한다.
- RDB에는 컬렉션과 같은 형태의 데이터를 컬럼에 저장할 수 없기 때문에, <b>별도의 테이블을 생성하여 컬렉션을 관리</b>해야한다.

<b>@ElementCollection</b> : 값 타입이 컬렉션임을 명시
<b>@CollectionTable</b> : 컬렉션 테이블 이름, 조인 설정
</pre>
```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ElementCollection
    @CollectionTable(
        name = "FAVORITE_FOOD", 
        joinColumns = @JoinColumn(name = "MEMBER_ID")
    )
    @Column(name = "FOOD_NAME") // 값(String)이 하나고 내가 정의한 것이 아니기 때문에 예외적으로 컬럼명 변경 허용
    private Set<String> favoriteFoods = new HashSet<>();

    @ElementCollection
    @CollectionTable(
        name = "ADDRESS", 
        joinColumns = @JoinColumn(name = "MEMBER_ID")
    )
    private List<Address> addressList = new ArrayList<>();

    public Member(String name) {
        this.name = name;
    }

    public Set<String> getFavoriteFoods() {
        return favoriteFoods;
    }

    public List<Address> getAddressList() {
        return addressList;
    }
}
```
<pre>
- 데이터베이스는 컬렉션을 저장할 수 없기 때문에 <b>별도의 테이블을 만들어 일대다 관계로 풀어서 컬렉션을 저장</b>해야 한다.
  그렇기 때문에 별도로 만들어질 테이블의 이름과 외래 키를 지정해준다.
- 값타입 컬렉션도 값 타입이기 때문에 따로 생명주기를 가지지 않고, <b>엔티티와 같은 생명주기를 따라간다.</b>
  이것은 <b>cascadeType.ALL + orphanRemoval = true</b>와 같다(값 타입은 이 기능을 항상 필수로 가지게 된다)
- 값 타입 컬렉션은 기본적으로 <b>지연 전략(fetch=FetchType.LAZY)을 사용</b>한다.
</pre>
### `값 타입 컬렉션의 제약 사항`
<pre>
- 값 타입은 <b>식별자</b>가 없기 때문에, 값을 변경할 때 추적할 수 없다.
- 값 타입 컬렉션의 변경사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 현재 컬렉션에 있는 모든 데이터를 다시 저장한다.
- 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 하나로 묶어서 기본키를 구성해야 함(<b>null 입력X, 중복 저장X</b>)
</pre>
```java
Member findMember = em.find(Member.class, member.getId());

// 기본 값 타입 컬렉션 수정
findMember.getFavoriteFoods().remove("치킨"); // 기존 데이터 삭제
findMember.getFavoriteFoods().add("한식"); // 새로운 데이터 추가

// 임베디드 값 타입 컬렉션 수정
// 값 타입은 불변해야한다. 때문에 기존 데이터를 삭제하고 새로운 데이터를 추가했다.
findMember.getAddressHistory().remove(new Address("oldCtiy","street","123456")); // 기존 데이터 삭제
findMember.getAddressHistory().add(new Address("newCtiy","street","123456")); // 새로운 데이터 추가
```
### `값 타입 컬렉션의 대안(실무)`
<pre>
- 값 타입 컬렉션을 일대다 관계를 가진 <b>엔티티로 승격</b>시킨다.
- 영속성 전이(Cascade) + 고아 객체 제거를 사용해서 엔티티를 값 타입 컬렉션 처럼 사용
- 기존의 값타입 컬렉션은 양방향을 설계 할 수 없었는데, 엔티티로 승격함으로써 양방향 매핑이 가능
- <b>식별자</b> 개념이 추가 되면서, 추적 할 수 없었던 문제 해결
</pre>
```java
/**
 * 값 타입 컬렉션 -> 엔티티로 승격
 *
 * ID 식별자가 있어서 이제 마음껏 수정하고 삭제가능
 * 일대다 관계를 위한 엔티티 -> AddressEntity를 만들고
 * 여기서 private Address address; -> 값 타입을 사용한다.
 * 영속성 전이(Cascade) + 고아 객체 제거를 사용해서 값 타입 컬렉션 처럼 사용
 */
@Entity
@Table(name = "ADDRESS")
public class AddressEntity {

    @Id
    @GeneratedValue
    private Long id; // ID 식별자 생김

    @Embedded
    private Address address; //여기서 값 타입을 사용

    public AddressEntity() {
    }

    // 이런식으로 값 타입에 생성자를 이용해서 인스턴스를 생성해 넣어준다.
    public AddressEntity(String city, String street, String zipcode){
        this.address = new Address(city, street, zipcode);
    }
}
```
```java
// Member에서 값 타입을 매핑하는게 아니라, 엔티티로 매핑한다.

@Entity
public class Member {

    // @ElementCollection
    // @CollectionTable(
    //     name = "ADDRESS", 
    //     joinColumns = @JoinColumn(name = "MEMBER_ID")
    // )
    // List<Address> addressList = new ArrayList<>();

    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name = "MEMBER_ID")
    private List<AddressEntity> addressHistory = new ArrayList<>();
}
```
```java
Member member = new Member();
member.getAddressHistory().add(new AddressEntity("city", "street", "zipcode"));
em.persist(member);
```
### `값 타입 컬렉션은 언제 사용하는가?`
<pre>
- 정말 단순한 select Box[ ()치킨, ()피자 ] 같은 것을 구현할 때 사용한다.
- 값을 추적할 필요가 없거나 업데이트가 필요 없을 때 사용
</pre>
## 정리
<pre>
<b>엔티티 타입 특징</b>
- 식별자 O
- 생명 주기 관리
- 공유

<b>값 타입 특징</b>
- 식별자 X
- 생명 주기를 엔티티에 의존
- 공유하지 않는 것이 안전(복사해서 사용)
- 불변 객체로 만드는 것이 안전
</pre>
