# Collection Query Optimization
## DTO 직접 조회 방식
<pre>
<b>문제 : XToMany(OneToMany) 관계를 가지고 있는 컬렉션(JCF)을 조인하면 다(N)를 기준으로 row(행)이 생성되므로 중복 데이터가 발생한다.</b>

<b>주문(Order) 엔티티의 컬렉션 조회하기</b>
-> 주문(Order) 한 건당 주문상품(OrderItem)을 2개씩 주문을 했다고 가정하면
   조인 시 2건의 주문상품이 조회되어야 하지만 다(N)의 기준으로 조인되어 4건의 주문상품이 조회된다.

<b>컨트롤러단에서 엔티티를 조회하여 DTO로 변환하지 않고, 리포지토리에서 DTO를 이용하여 바로 조회한다.</b>
- 일반적인 SQL을 사용할 때 처럼 원하는 값을 선택하여 조회한다.
- new 명령어를 사용하여 JPQL의 결과를 DTO로 즉시 변환
- SELECT절에서 원하는 데이터를 직접 선택하므로 DB -> 애플리케이션 네트웍 용량 최적화(생각보다 미비)
- <b>리포지토리 재사용성이 떨어짐, API 스펙에 맞춘 코드가 리포지토리에 들어가는 단점이다.</b>
- 리포지토리는 가급적 순수한 엔티티를 조회하는데 사용하기 때문에 <b>따로 DTO로 조회하는 리포지토리를 만드는 것이 유지보수성에 좋다.</b>
- <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA2/img/dtorepository.PNG"/>

컬렉션 조회 최적화 방법은 <b>엔티티 조회 방식</b>과 <b>DTO 직접 조회 방식</b>을 이용한다.
</pre>
### OrderApiController
```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;
    private final OrderQueryRepository orderQueryRepository;

    /**
     * V4. JPA에서 DTO로 바로 조회, 컬렉션 N 조회 (1 + N Query)
     * - 페이징 가능
     */
    @GetMapping("api/v4/orders")
    public List<OrderQueryDto> ordersV4(){
         return orderQueryRepository.findOrderQueryDtos();
    }

    /**
     * V5. JPA에서 DTO로 바로 조회, 컬렉션 1 조회 최적화 버전 (1 + 1 Query)
     * - 1:N 관계인 컬렉션은 IN 절을 활용해서 메모리에 미리 조회해서 최적화
     * - 페이징 가능
     */
    @GetMapping("api/v5/orders")
    public List<OrderQueryDto> ordersV5(){
        return orderQueryRepository.findAllbyDto_optimization();
    }

    /**
     * V6. JPA에서 DTO로 바로 조회, 플랫 데이터(1 Query) (1 Query)
     * - JOIN 결과를 그대로 조회 후 애플리케이션에서 원하는 모양으로 직접 변환
     * - 페이징 불가능
     */
    @GetMapping("api/v6/orders")
    public List<OrderQueryDto> ordersV6(){
        List<OrderFlatDto> flats = orderQueryRepository.findAllbyDto_flat();

        // OrderFlatDto 를 가지고 loop 를 돌려서 OrderQueryDto 로 변환하는 작업
        return flats.stream()
                .collect(groupingBy(o -> new OrderQueryDto(o.getOrderId(), o.getName(), o.getOrderDate(), o.getOrderStatus(), o.getAddress()),
                        mapping(o -> new OrderItemQueryDto(o.getOrderId(), o.getItemName(), o.getOrderPrice(), o.getCount()), toList())
                )).entrySet().stream()
                .map(e -> new OrderQueryDto(e.getKey().getOrderId(), e.getKey().getName(), e.getKey().getOrderDate(), e.getKey().getOrderStatus(),e.getKey().getAddress(), e.getValue()))
                .collect(toList());
    }
}
```
### OrderQueryRepository
```java
@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {

    private final EntityManager em;

    /**
     * 컬렉션은 별도로 조회
     * Query: 루트 1번, 컬렉션 N 번
     * 단건 조회에서 많이 사용하는 방식
     */
    public List<OrderQueryDto> findOrderQueryDtos(){

        // 루트 조회(toOne 코드를 모두 한번에 조회)
        List<OrderQueryDto> result = findOrders(); //query 1번 -> 2개

        // 루프를 돌리면서 result에서 채우지 못한 컬렉션 부분을 직접 추가(추가 쿼리 실행)
        result.forEach(o ->{
            List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId()); //query N번
            o.setOrderItems(orderItems);
        });
        return result;
    }

    /**
     * V4. 1:N 관계인 orderItems 컬렉션 조회
     * OrderItem oi 에서 oi.item i는 XToOne 관계이기 때문에 그냥 join 을 했다.
     */
    private List<OrderItemQueryDto> findOrderItems(Long orderId){
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name ,oi.orderPrice, oi.count) " +
                        " from OrderItem oi" +
                        " join oi.item i " +
                        " where oi.order.id = :orderId", OrderItemQueryDto.class)
                .setParameter("orderId", orderId)
                .getResultList();
    }

    /**
     * V4. 1:N 관계(컬렉션)를 제외한 Order 를 한번에 조회
     */
    private List<OrderQueryDto> findOrders(){
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                        " from Order o" +
                        " join o.member m" +
                        " join o.delivery d", OrderQueryDto.class)
                .getResultList();
    }

    /**
     * V5. 최적화
     * Query: 루트 1번, 컬렉션 1번
     * 데이터를 한꺼번에 처리할 때 많이 사용하는 방식
     * 1:N 관계인 컬렉션은 IN 절을 활용해서 메모리에 미리 조회해서 최적화
     */
    public List<OrderQueryDto> findAllbyDto_optimization() {

        // 루트 조회(toOne 코드를 모두 한번에 조회)
        List<OrderQueryDto> result = findOrders();

        // orderItem 컬렉션을 MAP 으로 한방에 조회
        Map<Long, List<OrderItemQueryDto>> orderItemMap = findOrderItemMap(toOrderIds(result));

        // 루프를 돌리면서 비어있는 OrderItems(OrderId) 의 값을 넣어준다.
        result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));

        return result;
    }

    // V5. XToOne 관계를 조회한 컬렉션을 map을 이용해서 OrderId로 바꿔준다(식별자를 이용한 조회)
    private List<Long> toOrderIds(List<OrderQueryDto> result) {
        return result.stream()
                .map(o -> o.getOrderId())
                .collect(Collectors.toList());
    }

    // V5. IN 절을 이용해서 파라미터로 들어온 식별자와 동일한 모든 결과를 가져온다(다수 조회)
    private Map<Long, List<OrderItemQueryDto>> findOrderItemMap(List<Long> orderIds) {
        List<OrderItemQueryDto> orderItems = em.createQuery(
                        "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name ,oi.orderPrice, oi.count) " +
                                " from OrderItem oi" +
                                " join oi.item i " +
                                " where oi.order.id in :orderId", OrderItemQueryDto.class)
                .setParameter("orderId", orderIds)
                .getResultList();

        // map 으로 변환
        // KEY : OrderId, VALUE : List<OrderItemQueryDto>
        // MAP 을 사용해서 매칭 성능 향상(O(1))
        Map<Long, List<OrderItemQueryDto>> orderItemMap = orderItems.stream()
                .collect(Collectors.groupingBy(OrderItemQueryDto -> OrderItemQueryDto.getOrderId()));
        return orderItemMap;
    }

    /**
     * V6. 플랫 데이터 최적화
     * JOIN 결과를 그대로 조회 후 애플리케이션에서 원하는 모양으로 직접 변환
     */
    public List<OrderFlatDto> findAllbyDto_flat() {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.query.OrderFlatDto(" +
                    o.id, m.name, o.orderDate, o.status, d.address, i.name, oi.orderPrice, oi.count)" +
                        " from Order o" +
                        " join o.member m" +
                        " join o.delivery d" +
                        " join o.orderItems oi" +
                        " join oi.item i", OrderFlatDto.class)
                .getResultList();
    }
}
```
### OrderQueryDto
```java
// 1:N 관계(컬렉션)를 제외한 XToOne 관계를 조회하는 DTO

@Data
@EqualsAndHashCode(of = "orderId")
public class OrderQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemQueryDto> orderItems;

    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }

    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address, List<OrderItemQueryDto> orderItems) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
        this.orderItems = orderItems;
    }
}
```
### OrderItemQueryDto
```java
// 1:N 관계인 orderItems 컬렉션 조회하는 DTO

@Data
public class OrderItemQueryDto {

    @JsonIgnore
    private Long orderId; //식별 하기위해 사용 (없어도됨) OrderItem.order.id(FK)
    private String itemName;
    private int orderPrice;
    private int count;

    public OrderItemQueryDto(Long orderId, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}
```
### OrderFlatDto
```java
// V6. JOIN된 결과를 받는 DTO

@Data
public class OrderFlatDto {

    //Order
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    //OrderItem, Item
    private String itemName;
    private int orderPrice;
    private int count;

    public OrderFlatDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}
```
### V4. JPA에서 DTO로 바로 조회, 컬렉션 N 조회
<pre>
<b>조회</b>
1. 따로 DTO로 조회하는 리포지토리 생성
2. 1:N 관계(컬렉션)를 제외한 XToOne 관계를 먼저 조회
   - ToOne 관계는 조인해도 데이터 row 수가 증가하지 않는다.
3. 1:N 관계인 orderItems 컬렉션 조회(loop를 돌려서 컬렉션 부분을 별도로 추가)
   - ToMany(1:N) 관계는 조인하면 row 수가 증가한다.

<b>결론</b>
- row 수가 증가하지 않는 ToOne 관계는 조인으로 최적화 하기 쉬우므로 한번에 조회하고,
  ToMany 관계는 최적화 하기 어려우므로 findOrderItems() 같은 별도의 메서드로 조회한다.

<b>성능</b>
- Query: 루트 1번, 컬렉션 N 번 실행
</pre>
### V5. JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화
<pre>
<b>조회</b>
1. 따로 DTO로 조회하는 리포지토리 생성
2. 1:N 관계(컬렉션)를 제외한 XToOne 관계를 먼저 조회
3. XToOne 관계 조회에서 얻은 식별자 OrderId로 XToMany 관계인 OrderItem 을 한꺼번에 조회
   - <b>Map으로 변환해서 매칭 성능 향상(O(1))</b>

<b>성능</b>
- Query: 루트 1번, 컬렉션 1 번 실행
</pre>
### V6. JPA에서 DTO로 직접 조회, 플랫 데이터 최적화
<pre>
<b>조회</b>
1. 따로 DTO로 조회하는 리포지토리 생성
2. JOIN 결과를 그대로 조회한다(JOIN 결과를 그대로 받는 DTO 생성)
3. 컨트롤러단에서 조회된 JOIN 결과를 가지고 loop 를 돌려서 View단으로 내보낼 API스펙으로 변환

<b>성능</b>
- Query: 1번

<b>단점</b>
- 쿼리는 한번이지만 조인으로 인해 DB에서 애플리케이션에 전달하는 데이터에 중복 데이터가 추가되므로 상황에 따라 V5 보다 더 느릴 수도 있다.
- 애플리케이션에서 추가 작업이 크다.
- 페이징이 불가능하다.
</pre>
