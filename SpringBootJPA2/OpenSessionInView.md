# OSIV - Open Session In View
## OSIV ON
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA2/img/OSIVTRUE.PNG"/>
<b>spring.jpa.open-in-view : true 기본값</b>

<b>장점</b>
- OSIV 전략은 트랜잭션 시작처럼 <b>최초 데이터베이스 커넥션 시작 시점부터 API 응답이 끝날 때 까지
  영속성 컨텍스트와 데이터베이스 커넥션을 유지한다.</b>
  (그래서 지금까지 View Template이나 API 컨트롤러에서 지연 로딩이 가능했던 것이다.)
- <b>지연 로딩은 컨텍스트가 살아있어야 가능하고, 영속성 컨텍스트는 기본적으로 데이터베이스 커넥션을 유지한다.</b>

<b>단점</b>
- 너무 오랜시간동안 데이터베이스 커넥션 리소스를 사용하기 때문에, 실시간 트래픽이 중요한 애플리케이션에서는 커넥션이 모자랄 수 있다.
  (컨트롤러에서 외부 API를 호출하면 <b>외부 API 대기 시간 만큼 커넥션 리소스를 반환하지 못하고, 유지해야 한다.</b>)
</pre>
## OSIV OFF
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA2/img/OSIVFALSE.PNG"/>
<b>spring.jpa.open-in-view : false OSIV 종료</b>

<b>장점</b>
- OSIV를 끄면 <b>트랜잭션을 종료할 때 영속성 컨텍스트를 닫고, 데이터베이스 커넥션도 반환한다. 따라서 커넥션 리소스를 낭비하지 않는다.</b>

<b>단점</b>
- <b>OSIV를 끄면 모든 지연로딩을 트랜잭션 안에서 처리해야 한다.</b> 따라서 지금까지 작성한 많은 지연 로딩 코드를 트랜잭션 안으로 넣어야 한다.
- View template에서 지연로딩이 동작하지 않는다. 트랜잭션이 끝나기 전에 지연 로딩을 강제로 호출해 두어야 한다.

<b>해결방법</b>
- <b>커멘드와 쿼리 분리</b>, <b>fetch join 사용</b>등이 있다.
- 실무에서 OSIV를 끈 상태로 복잡성을 관리하는 좋은 방법으로는 Command와 Query를 분리하는 것이다.
- 크고 복잡한 애플리케이션을 개발한다면, 이 둘의 관심사를 명확하게 분리하는 선택은 유지보수 관점에서 충분히 의미가 있다.

ex) OrderService
    - OrderService : 핵심 비즈니스 로직
    - OrderQueryService : 화면이나 API에 맞춘 서비스(주로 읽기 전용 틀랜잭션 사용)

<b>고객 서비스의 실시간 API는 OSIV를 끄고, ADMIN 처럼 커넥션을 많이 사용하지 않는 곳에서는 OSIV를 켠다.</b>
</pre>
## 커멘드와 쿼리 분리
### Before
```java
/**
* V2. 엔티티를 조회해서 DTO로 변환(fetch join 사용X)
* - 트랜잭션 안에서 지연 로딩 필요하다.
* - 하지만 OSIV 종료인 상태에서는 트랜잭션 이외에는 LAZY 초기화를 할 수 없다.
* - 해결방법 : 커멘드와 쿼리를 분리한다.
*/
@GetMapping("api/v2/orders")
public List<OrderDto> ordersV2(){
    List<Order> orders = orderRepository.findAllByString(new OrderSearch());
    List<OrderDto> collect = orders.stream()
            .map(o -> new OrderDto(o))
            .collect(Collectors.toList());
    return collect;
}

//DTO
@Data
static class OrderDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    // 엔티티가 외부에 노출되면안된다(완전히 의존성을 끊어야 한다.)
    // OrderItemDto 를 만들어서 OrderItem Entity 를 한 번 더 감싼다.
    private List<OrderItemDto> orderItems;

    public OrderDto(Order order){
        this.orderId = order.getId();
        this.name = order.getMember().getName(); // LAZY 초기화
        this.orderDate = order.getOrderDate();
        this.orderStatus = order.getStatus();
        this.address = order.getDelivery().getAddress(); // LAZY 초기화
        this.orderItems = order.getOrderItems().stream()
                .map(orderItem -> new OrderItemDto(orderItem))
                .collect(Collectors.toList());
    }
}

//DTO
@Data
static class OrderItemDto{

    private String itemName; //상품 명
    private int orderPrice; //주문 가격
    private int count; //주문 수량

    public OrderItemDto(OrderItem orderItem) {
        this.itemName = orderItem.getItem().getName();
        this.orderPrice = orderItem.getOrderPrice();
        this.count = orderItem.getCount();
    }
}
```
### After
```java
/**
* V2. 엔티티를 조회해서 DTO로 변환(fetch join 사용X)
* - 트랜잭션 안에서 지연 로딩 필요하다.
* - 하지만 OSIV 종료인 상태에서는 트랜잭션 이외에는 LAZY 초기화를 할 수 없다.
* - 해결방법 : 커멘드와 쿼리를 분리한다.
*/
@GetMapping("api/v2/orders")
public List<OrderDto> ordersV2(){
    return orderQueryService.ordersV3();
}
```
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderQueryService {

    private final OrderRepository orderRepository;

    /**
    * OrderQueryService를 만들어서 Transaction 안에서 LAZY 를 초기화 하였다.
    * DTO 도 새로 만들어서 사용 하였다.
    */
    public List<OrderDto> ordersV2(){
        List<Order> orders = orderRepository.findAllWithItem();

        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(toList());
        
        return result;
    }
}
```
