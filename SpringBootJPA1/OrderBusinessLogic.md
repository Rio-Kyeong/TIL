# Order Business Logic implement
## Business Logic
<pre>
주문 서비스의 주문과 주문 취소 메서드를 보면 <b>비즈니스 로직 대부분이 엔티티에 있다.</b>
<b>서비스 계층은 단순히 엔티티에 필요한 요청을 위임하는 역할</b>을 한다.
이처럼 <b>엔티티가 비즈니스 로직을 가지고 객체 지향의 특성을 적극 활용하는 것</b>을 <b>도메인 모델 패턴</b>이라 한다.

반대로 <b>엔티티에는 비즈니스 로직이 거의 없고 서비스 계층에서 대부분의 비즈니스 로직을 처리하는 것</b>을 <b>트랜잭션 스크립트 패턴</b>이라 한다.


<b>도메인 모델 패턴</b>을 이용한 <b>Order(주문) Entity</b>

<b>생성 메서드(createOrder())</b>
    - 생성 메서드를 사용함으로써 하나하나 order.setXXX() 으로 값을 설정하지 않아도 된다.
    - 주문 엔티티를 생성할 때 사용한다. 주문 회원, 배송정보, 주문상품의 정보를 받아서 실제 주문 엔티티를 생성한다.

<b>비즈니스 로직(cancel())</b>
    - 주문 취소시 사용한다.
    - 주문 상태를 취소로 변경하고 주문상품에 주문 취소를 알린다.
    - 만약 이미 배송을 완료한 상품이면 주문을 취소하지 못하도록 예외를 발생시킨다.

<b>조회 로직(getTotalPrice())</b>
    - 주문 시 사용한 전체 주문 가격을 조회한다.
    - 전체 주문 가격을 알려면 각각의 주문상품 가격을 알아야 한다.
    - 로직을 보면 연관된 주문상품들의 가격을 조회해서 더한 값을 반환한다.
      (실무에서는 주로 주문에 전체 주문 가격 필드를 두고 역정규화 한다)


<b>도메인 모델 패턴</b>을 이용한 <b>OrderItem(주문 상품) Entity</b>

<b>생성 메서드(createOrderItem())</b>
    - 생성 메서드를 사용함으로써 하나하나 orderItem.setXXX() 으로 값을 설정하지 않아도 된다.
    - 주문 상품, 가격, 수량 정보를 사용해서 주문상품 엔티티를 생성한다.
    - 그리고 item.removeStock(count) 를 호출해서 주문한 수량만큼 상품의 재고를 줄인다.
      (Order.createOrder()에서 매개변수로 들어오는 OrderItem에는 감소된 재고량이 들어온다)

<b>비즈니스 로직(cancel())</b>
    - getItem().addStock(this.count) 를 호출해서 취소한 주문 수량만큼 상품의 재고를 증가시킨다.

<b>조회 로직(getTotalPrice())</b>
    - 주문 가격에 수량을 곱한 값을 반환한다(전체 주문 가격 계산)
</pre>
## Order Entity
```java
@Entity
//Entity Class 와 DB Table 의 이름이 다르면 @Table 을 통해서 이름을 지정해준다.
@Table(name = "orders")
@Setter @Getter
public class Order {

    @Id // PK
    @GeneratedValue // PK의 값을 위한 자동 생성
    @Column(name = "order_id") // "order_id" Column 을 mapping
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY) // Order(상품)입장에서는 Member(회원)과 N:1 관계이다.
    @JoinColumn(name = "member_id") //member_id를 FK로 설정
    private Member member; // 연관관계의 주인

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<OrderItem>();

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

    //createOrder 를 통해서만 값을 설정(set)할 수 있도록 생성자를 막아둔다.
    protected Order() {
    }

    //==생성 메서드==//
    // 하나하나 찾아서 설정(set)할 필요 없이 해당 메서드의 매개변수 변경으로만 Entity 를 제어를 할 수 있다.
    public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems){
        Order order = new Order();
        order.setMember(member);
        order.setDelivery(delivery);
        for(OrderItem orderItem : orderItems){
            order.addOrderItem(orderItem);
        }
        // 주문 상태의 기본 값을 ORDER 로 설정
        order.setStatus(OrderStatus.ORDER);
        // 주문 시간을 현재로 설정
        order.setOrderDate(LocalDateTime.now());
        return order;
    }

    //==비즈니스 로직==//
    // 주문 취소
    public void cancel(){
        //배송완료 시 취소 불가능
        if(delivery.getStatus() == DeliveryStatus.COMP){
            throw new IllegalStateException("이미 배송완료된 상품은 취소가 불가능합니다.");
        }

        // 주문 상태를 CANCEL(취소)로 변경
        this.setStatus(OrderStatus.CANCEL);
        // loop 를 돌리면서 주문상품의 재고수량을 원복
        for(OrderItem orderItem : this.orderItems){
            orderItem.cancel();
        }
    }

    //==조회 로직==//
    // 전체 주문 가격 조회
    public int getTotalPrice(){
        int totalPrice = 0;
        for(OrderItem orderItem : this.orderItems){
            totalPrice += orderItem.getTotalPrice();
        }
        return  totalPrice;
    }
}
```
## OrderItem Entity
```java
@Entity
@Getter @Setter
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "order_item_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "item_id") //item_id를 FK로 지정
    private Item item;

    @ManyToOne(fetch = FetchType.LAZY) // 여러개 주문한 아이템 : 하나의 주문(N:1)
    @JoinColumn(name = "order_id") //order_id를 FK로 지정
    private  Order order;

    private int orderPrice; //주문가격
    private int count; //주문 수량

    //createOrderItem 를 통해서만 값을 설정(set)할 수 있도록 생성자를 막아둔다.
    protected OrderItem() {
    }

    //==생성 메서드==//
    public static OrderItem createOrderItem(Item item, int orderPrice, int count){
        OrderItem orderItem = new OrderItem();
        orderItem.setItem(item);
        orderItem.setOrderPrice(orderPrice);
        orderItem.setCount(count);

        //OrderItem 을 생성할 때는 기본적으로 재고수량을 감소시켜야한다.
        //Order.createOrder()에서 매개변수로 들어오는 OrderItem에는 감소된 재고량이 들어온다.
        item.removeStock(count);
        return orderItem;
    }

    //==비즈니스 로직==//
    public void cancel(){
        getItem().addStock(this.count); //재고수량을 원복
    }

    //==조회 로직==//
    //전체 주문 가격 계산
    public int getTotalPrice() {
        return getOrderPrice() *  getCount();
    }
}
```