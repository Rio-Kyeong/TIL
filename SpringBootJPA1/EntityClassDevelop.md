# Entity Class Develop
## 연관관계와 연관관계의 주인
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA1/img/relationship.PNG"/>
</pre>
<pre>
<b>단방향 연관관계</b>
- OrderItem(주문상품)과 Item(상품)은 <b>단방향 매핑으로  OrderItem(주문상품)만 Item(상품)을 참조</b>하고있다.

<b>단방향 연관관계의 주인(Owner)</b>
- Item(상품)을 참조하는 OrderItem(주문상품)쪽에 외래 키 설정(@JoinColumn)을 하여 참조 시킨다.
</pre>
<pre>
<b>양방향 연관관계</b>
- Order(주문)와 OrderItem(주문상품)은 <b>양방향 매핑</b>을 이루고 있다.

<b>객체 연관관계</b>
    - <b>객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단방향 관계 2개</b>다.
    - 객체를 양방향으로 참조하려면 억지로 단방향 연관관계를 2개 만들어야 한다.
        * 주문 -> 주문상품 연관관계 1개(단방향)
        * 주문상품 -> 주문 연관관계 1개(단방향)

<b>테이블 연관관계</b>
    - <b>테이블은 외래 키 하나로 두 테이블의 연관관계를 관리</b>한다.
        * 주문 <-> 주문상품 연관관계 1개(양방향)

<b>양방향 연관관계의 주인(Owner)</b>
    - 객체의 두 관계중 하나를 연관관계의 주인으로 지정해야한다.
    - <b>외래 키가 있는(@JoinColumn이 있는 쪽)을 주인으로 설정한다.</b>
    - 객체에 둘다 정보를 업데이트 해도, <b>연관관계의 주인 것만 DB에 영향</b>을 준다.
    - <b>주인이 아닌쪽은 읽기만 가능</b>
    - <b>주인이 아닌 곳에서는 mappedBy로 주인을 명시</b>해야하며, <b>주인만 FK를 관리</b>한다.
</pre>
<pre>
<b>연관관계 편의 메서드</b>
    - <b>양방향 연관관계를 한번에 설정하는 편리한 메서드</b>
    - 엔티티 A와 B가 서로 양방향 연관관계인데, 어디에 연관관계 편의 메서드를 두는게 좋을까?
      -> JPA의 영역이라기 보다는 오히려 객체지향 설계의 영역이기 때문에 어디에 두든 정답은 없다.
</pre>
### Order(주문)
```java
//Entity Class
@Entity
//Entity Class 와 DB Table 의 이름이 다르면 @Table 을 통해서 이름을 지정해준다.
@Table(name = "orders")
@Getter
public class Order { 

    @Id // PK
    @GeneratedValue // PK의 값을 위한 자동 생성
    @Column(name = "order_id") // "order_id" Column 을 mapping
    private Long id;

    @ManyToOne // Order(상품)입장에서는 Member(회원)과 N:1 관계이다.
    @JoinColumn(name = "member_id") //member_id를 FK로 설정
    private Member member; // 연관관계의 주인

    // mappedBy로 연관관계의 주인을 명시 함으로써 OrderItem의 order(FK)가 주인이 된다.
    // orderItems 의 값을 변경한다고 해서 OrderItem 의 order field(FK)의 값이 변경되지 않는다.
    // OrderItem의 order(FK)를 업데이트 하면 mappedBy로 매핑된 Order의 orderItems로 업데이트 된 DB의 OrderItem을 읽을 수 있다.
    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<OrderItem>();

    // 1:1 (1:1관계에서는 PK를 어디에 넣어도 상관없다)
    // 고로 연관관계의 주인을 어디에 설정하여도 상관없다.
    // 모든 Entity는 기본적으로 parsist()를 하려면 각각 해야한다.
    // CascadeType.ALL 속성값으로 Order 를 parsist 하면 자동으로 Delivery 도 parsist 된다.
    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery; // 배송

    private LocalDateTime orderDate; //주문시간

    @Enumerated(EnumType.STRING) //enum type
    private OrderStatus status; //주문상태 [ORDER, CANCEL]
    
    //==연관관계 편의 메서드==//
    public void setMember(Member member){
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem(OrderItem orderItem){
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery(Delivery delivery){
        this.delivery = delivery;
        delivery.setOrder(this);
    }
}
```
### OrderItem(주문상품)
```java
@Entity
@Getter
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "order_item_id") // "order_item_id" Column 을 mapping
    private Long id;


    // 단방향 연관관계(OrderItem(주문상품)만 Item(상품)을 참조 하고있다)
    @ManyToOne
    @JoinColumn(name = "item_id") //item_id를 FK로 지정
    private Item item;

    @ManyToOne // 여러개 주문한 아이템 : 하나의 주문(N:1)
    @JoinColumn(name = "order_id") //order_id를 FK로 지정
    private  Order order;

    private int orderPrice; //주문가격

    private int count; //주문수량
}
```
## 주소 값 타입
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA1/img/EntityAnalytical.PNG"/>

<b>임베디드 타입</b>
- 회원(Member)과 배송(Delivery)는 임베디드 타입인 주소(Address)를 참조 하고있다.
- <b>Address를 표현하는 city,street,zipcode 필드를 하나로 묶어서 코드의 가독성을 높이고,
  좀 더 주소라는 의미를 확실하게 표현</b>하였다.
- <b>주소 값 타입은 변경 불가능하게 설계</b>해야 한다.(Setter메소드를 사용하지 않는다)
- <b>@Embeddable</b> : 값 타입을 정의하는 곳에 표시
- <b>@Embedded</b> : 값 타입을 사용하는 곳에 표시
</pre>
### Address(주소)
```java
@Embeddable //해당 클래스가 다른 엔티티의 일부로 저장될 수 있음을 설정
@Getter
public class Address {

    private String city;
    private String street;
    private String zipcode;

    protected Address() {
    }

    // 값 타입은 변경 불가능하게 설계해야 한다.
    // @Setter 를 제거하고, 생성자에서 값을 모두 초기화해서 변경 불가능한 클래스를 만든다.
    // 객체를 생성할 때 인자있는 생성자를 통해서 값을 지정 후 값을 변경할 수 없다.
    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }
}
```
### Member(회원), Delivery(배송)
```java
    @Embedded // 내장 타입
    private  Address address;
```
## Enum Class
<pre>
JDK 1.5부터는 C언어의 열거체보다 더욱 향상된 성능의 <b>열거체를 정의한 Enum 클래스</b>를 사용할 수 있다.

<b>@Enumerated(EnumType.ORDINAL)</b> : enum 순서 값을 데이터베이스에 저장할 때(사용안함)
<b>@Enumerated(EnumType.STRING)</b> : enum 이름을 데이터베이스에 저장할 때
</pre>
### DeliveryStatus(배송상태)
```java
public enum DeliveryStatus {
    READY, //(배송준비)
    COMP; //(배송)
}
```
### Delivery(배송)
```java
@Entity
@Getter
public class Delivery {
    // default 설정이 EnumType.ORDINAL 이다.
    
    //EnumType.ORDINAL : enum 순서 값을 DB에 저장
    // (EnumType.ORDINAL 은 데이터 추가 시 순서 값이 꼬일 수 있기 때문에 쓰지 않는게 좋다)
    // ex) DeliveryStatus.READY 는 1로 저장, DeliveryStatus.COMP 는 2로 숫자로 저장
    
    // EnumType.STRING : enum 이름을 DB에 저장
    // ex) "READY", "COMP" 문자열 자체가 저장
    @Enumerated(EnumType.STRING)
    private DeliveryStatus status; //READY(배송준비), COMP(배송)
}
```
## 상속관계 매핑
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA1/img/dtype.PNG"/>

- Item(상품) 클래스는 여러(Album, Movie, Book) 클래스들에게 상속되어 있다.
- 객체에는 상속관계가 존재하지만, 관계형 데이터베이스는 상속 관계가 없다.
- 그나마 슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 유사하다.
- 상속관계 매핑이라는 것은 객체의 상속 구조와 DB의 슈퍼타입 서브타입 관계를 매핑하는 것이다.

상속관계 전략지정 : <b>@Inheritance(strategy=InheritanceType.XXX)</b>
- default 전략은 SINGLE_TABLE(단일 테이블 전략)이다.

InheritanceType 종류
- InheritanceType.<b>JOINED(각각의 테이블로 변환하는 조인 전략)</b>
- InheritanceType.<b>SINGLE_TABLE(통합 테이블로 변환하는 단일 테이블 전략)</b>
- InheritanceType.<b>TABLE_PER_CLASS(서브타입 테이블로 변환하는 구현 클래스마다 테이블을 생성하는 전략</b>

<b>@DiscriminatorColumn(name="DTYPE")</b>
- 부모클래스에 선언한다.
- 하위 클래스를 구분하는 용도의 컬럼이다. 관례는 default = DTYPE

<b>@DiscriminatorValue("XXX")</b> - 필수아님
- 하위 클래스에 선언한다.
- 엔티티를 저장할 때 <b>슈퍼타입의 구분 컬럼에 저장할 값을 지정</b>한다.
- 어노테이션을 선언하지 않을 경우 기본 값으로 클래스 이름이 들어간다.

자세한 내용은 이곳을 <a href="https://ict-nroo.tistory.com/128">참조</a>
</pre>
### Item(상품)
```java
@Entity
//single_table 전략을 사용한다(한개의 테이블에 모든 컬럼 정의)
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name="DTYPE") //DTYPE으로 구분
@Getter 
public abstract class Item {

    @Id
    @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;
}
```
### Album
```java
@Entity
@DiscriminatorValue("A")// 구분 값
@Getter 
public class Album extends Item{

    private String artist;
    private String etc;
}
```
### Movie
```java
@Entity
@DiscriminatorValue("M")// 구분 값
@Getter 
public class Movie extends Item{

    private String director;
    private String actor;
}
```
### Book
```java
@Entity
@DiscriminatorValue("B")// 구분 값
@Getter 
public class Book extends Item{

    private String author;
    private String isbn;
}
```
