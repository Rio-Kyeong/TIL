# Collection Query Optimization
## 엔티티 조회 방식
<pre>
<b>문제 : XToMany(OneToMany) 관계를 가지고 있는 컬렉션(JCF)을 조인하면 다(N)를 기준으로 row(행)이 생성되므로 중복 데이터가 발생한다.</b>

<b>주문(Order) 엔티티의 컬렉션 조회하기</b>
-> 주문(Order) 한 건당 주문상품(OrderItem)을 2개씩 주문을 했다고 가정하면
   조인 시 2건의 주문상품이 조회되어야 하지만 다(N)의 기준으로 조인되어 4건의 주문상품이 조회된다.

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
     * V1. 엔티티 직접 노출
     * - Hibernate5Module 모듈 등록, LAZY=null 처리
     * - orderItem, item 관계를 직접 초기화하면 Hibernate5Module 설정에 의해 엔티티를 JSON 으로 생성한다.
     * - 양방향 관계 문제 발생 -> @JsonIgnore
     */
    @GetMapping("api/v1/orders")
    public List<Order> ordersV1(){
        List<Order> all = orderRepository.findAllByString(new OrderSearch());
        for (Order order : all) {
            // XXToOne
            order.getMember().getName(); //LAZY 강제 초기화
            order.getDelivery().getAddress(); //LAZY 강제 초기화
            // XXToMany
            // 상품관련정보(상품의 이름)도 같이 출력하고싶은데 OrderItem 에는 상품에 대한 정보가 없고
            // 객체참조로 Item 이 있다(LAZY 이기 때문에 강제초기화를 해야한다)
            List<OrderItem> orderItems = order.getOrderItems(); //LAZY 강제 초기화
            for (OrderItem orderItem : orderItems) {
                orderItem.getItem().getName();
            }
        }
        return all;
    }

    /**
    * V2. 엔티티를 조회해서 DTO로 변환(fetch join 사용X)
    * - 트랜잭션 안에서 지연 로딩 필요
    * - 지연 로딩으로 너무 많은 SQL 실행
    * - SQL 실행 수 : order 1번 + member, address N번 + orderItem N번 + item N번
    */
    @GetMapping("api/v2/orders")
    public List<OrderDto> ordersV2(){
        List<Order> orders = orderRepository.findAllByString(new OrderSearch());
        List<OrderDto> collect = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(Collectors.toList());
        return collect;
    }

    //DTO(Order)
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

    //DTO(OrderItem)
    @Data
    static class OrderItemDto{

        //고객의 니즈에 맞춰서 필요한 변수만 만들어서 API 스펙을 설정한다.
        private String itemName; //상품 명
        private int orderPrice; //주문 가격
        private int count; //주문 수량

        public OrderItemDto(OrderItem orderItem) {
            this.itemName = orderItem.getItem().getName();
            this.orderPrice = orderItem.getOrderPrice();
            this.count = orderItem.getCount();
        }
    }

    /**
     * V3. 엔티티를 조회해서 DTO로 변환(fetch join 사용)
     * - 페이징 시에는 N 부분을 포기해야함(대신에 batch fetch size? 옵션 주면 N -> 1 쿼리로 변경가능)
     */
    @GetMapping("api/v3/orders")
    public List<OrderDto> ordersV3(){
        List<Order> orders = orderRepository.findAllWithItem();
        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(Collectors.toList());
        return result;
    }

    /**
     * V3.1 엔티티를 조회해서 DTO로 변환(페이징 고려)
     * - ToOne 관계만 우선 모두 페치 조인으로 최적화
     * - 컬렉션 관계는 hibernate.default_batch_fetch_size, @BatchSize로 최적화
     */
    @GetMapping("api/v3.1/orders")
    public List<OrderDto> ordersV3_page(
            @RequestParam(value = "offset", defaultValue = "0") int offset,
            @RequestParam(value = "limit", defaultValue = "100") int limit)
    {
        // XToOne 관계는 모두 패치조인 하고, XToMany 관계(컬렉션)은 지연 로딩으로 조회한다.
        List<Order> orders = orderRepository.findAllWithMemberDelivery(offset, limit);

        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(Collectors.toList());
        return result;
    }
```
###  V3. 엔티티를 조회해서 DTO로 변환(fetch join 사용)
<pre>
V2 에서 조회을 했을 때에는 쿼리가 반복적으로 호출되어 성능이 저하되었다.

<b>장점</b>
- 페치 조인으로 SQL이 1번만 실행된다.
- <b>distinct</b>를 사용한 이유는 1:N 조인이 있으므로 데이터베이스 row가 증가한다.
  그 결과 같은 order 엔티티의 조회 수도 증가하게 된다.
  JPA의 distinct는 SQL에 distinct를 추가하고, 더해서 <b>같은 엔티티가 조회되면 애플리케이션에서 중복을 걸러준다.</b>

<b>단점</b>
- 페이징(Paging)이 불가능하다(XToOne 관계는 문제 없이 페이징가능하다)
- 컬렉션 페치 조인은 1개만 사용할 수 있다.
  컬렉션 둘 이상에 페치 조인을 사용하면 데이터가 부정합하게 조회될 수 있다.

<b>주의</b>
- 결과는 distinct으로 인해서 정상적으로 중복 데이터 없이 조회되지만, DB에서는 중복되어 나온다.  
</pre>
```java
//Repository
//XToMany 컬렉션 페치 조인 최적화 메소드(페이징 불가)
public List<Order> findAllWithItem() {
    return em.createQuery(
            "select distinct o from Order o" +
                    " join fetch o.member" +
                    " join fetch o.delivery" +
                    " join fetch o.orderItems oi" +
                    " join fetch oi.item i", Order.class
    ).getResultList();
}
```
###  V3.1 엔티티를 조회해서 DTO로 변환(페이징 고려)
<pre>
V3 에서 조회를 했을 때에는 페치 조인으로 성능 향상이 되었지만 페이징이 불가능하였다.

<b>문제</b>
- 일다대에서 일(1)을 기준으로 페이징을 하는 것이 목적이다. 그런데 데이터는 다(N)를 기준으로 row가 생성된다.
- 이 경우 하이버네이트는 경고 로그를 남기고 모든 DB 데이터를 읽어서 메모리에서 페이징을 시도한다. 최악의 경우 장애로 이어질 수 있다.

<b>해결방법</b>
- 먼저 <b>ToOne</b>(OneToOne, ManyToOne) 관계를 모두 페치조인 한다.
  ToOne 관계는 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.
- 컬렉션은 지연 로딩으로 조회한다.
- 지연 로딩 성능 최적화를 위해 <b>hibernate.default_batch_fetch_size</b> , <b>@BatchSize</b> 를 적용한다.
    * <b>hibernate.default_batch_fetch_size</b> : 글로벌 설정
    * <b>@BatchSize</b> : 개별 최적화
    * <b>이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size 만큼 IN 쿼리로 조회한다.</b>

<b>장점</b>
- <b>쿼리 호출 수가 1 + N -> 1 + 1으로 최적화 된다.</b>
- 조인보다 DB 데이터 전송량이 최적화 된다.(이 방법은 각각 조회하므로 전송해야할 중복 데이터가 없다.)
- 페이 조인 방식과 비교해서 쿼리 호출 수가 약간 증가하지만, DB 데이터 전송량이 감소한다.
- 컬렉션 페치 조인은 페이징이 불가능 하지만 이 방법은 페이징이 가능하다.

<b>결론</b>
- ToOne 관계는 페치 조인해도 페이징에 영향을 주지 않는다.
- <b>따라서 ToOne 관계는 페치조인으로 쿼리 수를 줄이고 해결하고, 나머지는 hibernate.default_batch_fetch_size 로 최적화 하자.</b>

<b>참고</b>
- default_batch_fetch_size 의 크기는 적당한 사이즈를 골라야 하는데, 100~1000 사이를 선택하는 것을 권장한다.
  이 전략은 SQL IN 절을 사용하는데, 데이터베이스에 따라 IN 절 파라미터를 1000으로 제한하기도 한다.

<b>querydsl</b>
- 가장 깔끔한 방법은 querydsl 을 이용해서 처리하는 것이다.
- 위에 고민들을 할필요가 없음. 동적 쿼리면 동적쿼리, 페이징이면 페이징 처리가 가능하다.
</pre>
```java
//Repository
//XToMany 컬렉션 페치 조인 최적화 메소드(페이징)
public List<Order> findAllWithMemberDelivery(int offset, int limit) {
    return em.createQuery(
            "select o from Order o" +
                    " join fetch o.member" +
                    " join fetch o.delivery d", Order.class)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList();
}
```
```yml
# application.yml
# hibernate.default_batch_fetch_size 전역설정
spring:
    jpa:
        properties:
            hibernate:
                default_batch_fetch_size: 1000
```
