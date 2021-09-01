# Lazy loading and Query performance optimization
## 지연 로딩(LAZY)
<pre>
<b>즉시 로딩(EAGER)</b>이란 객체 A를 조회할 때 A와 연관된 객체들을 한 번에 가져오는 것이다.
<b>지연 로딩(LAZY)</b>이란 객체 A를 조회할 때는 A만 가져오고 연관된 애들은 저번 게시글에서 본 프록시 초기화 방법으로 가져온다.

- 즉시로딩(EAGER)은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다. 특히 JPQL을 실행할 때 N+1 문제가 자주 발생한다.
- 실무에서 <b>모든 연관관계는 지연로딩(LAZY)으로 설정</b>해야 한다.
- 연관된 엔티티를 함께 DB에서 조회해야 하면, <b>fetch join 또는 엔티티 그래프 기능을 사용하여 한 번에 데이터를 가져올 수 있다.</b>

@xxxToMany 처럼 Many로 끝나는 관계는 기본 값이 지연로딩(LAZY)이다.
@xxxToOne 처럼 <b>One으로 끝나는 관계는 기본 값으로 즉시로딩(EAGER)</b>이므로 직접 지연로딩(LAZY)으로 변경해야 한다.

결론
- 즉시 로딩은 가급적 피하고 지연 로딩(LAZXY)을 사용한다.
- 보통 한 번에 연관된 엔티티들의 데이터를 조회해야 할 때는 페치 조인(fetch join)을 이용한다.
</pre>
## 지연 로딩과 조회 성능 최적화
<pre>
<b>목표 : 지연 로딩(LAZY) 설정이 되어있는 XToOne(ManyToOne, OneToOne) 관계를 가지는 엔티티 조회 시 성능 최적화</b>

<b>주문(Order) 엔티티 조회하기</b>
order -> member 와 order -> address 는 지연 로딩이다.
따라서 실제 엔티티 대신에 프록시가 존재한다.

<b>주의 : 지연 로딩(LAZY)을 피하기 위해 즉시 로딩(EARGR)으로 설정하면 안된다.</b>
- 즉시 로딩 때문에 연관관계가 필요 없는 경우에도 데이터를 항상 조회해서 성능 문제가 발생할 수 있다.
- 즉시 로딩으로 설정하면 성능 튜닝이 매우 어려워 진다.

결론 : <b>항상 지연 로딩을 기본</b>으로 하고, 성능 최적화가 필요한 경우에는 <b>페치 조인(fetch join)을 사용</b>해라!
</pre>
### OrderSimpleApiController
```java
/**
 * XToOne(ManyToOne, OneToOne) 관계 최적화
 * Order
 * Order -> Member
 * Order -> Delivery
 */
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {

    private final OrderRepository orderRepository;
    private final OrderSimpleQueryRepository orderSimpleQueryRepository;

    /**
     * V1. 엔티티 직접 노출
     * - 엔티티를 API 응답으로 외부로 노출하는것은 좋지 않다.
     * - Hibernate5Module 모듈 등록, LAZY=null 처리
     * - 양방향 관계 문제 발생 -> @JsonIgnore
     */
    @GetMapping("/api/v1/sample-orders")
    public List<Order> ordersV1(){
        List<Order> all = orderRepository.findAllByString(new OrderSearch());
        //원하는 정보만 출력하기
        for(Order order : all){
            // order.getMember() : 프록시 객체
            // order member 와 order address 는 지연 로딩이다.
            // 따라서 실제 엔티티 대신에 프록시 존재
            order.getMember().getName(); //Lazy 강제 초기화
            order.getDelivery().getAddress(); //Lazy 강제 초기화
        }
        return all;
    }

    /**
     * V2. 엔티티를 조회해서 DTO 로 변환(fetch join 사용X)
     * - 단점: 지연로딩으로 쿼리 N번 호출(많은 쿼리 호출의 발생)
     * - 지연로딩은 영속성 컨텍스트에서 조회하므로, 이미 조회된 경우 쿼리를 생략한다.
     */
    @GetMapping("/api/v2/sample-orders")
    public List<SimpleOrderDto> ordersV2(){
        // 이 method 는 service 에서 구현 한 내용이 없기 때문에 바로 Repository 에서 받았다.
        // Order(o) 를 SimpleOrderDto 로 변환
        List<Order> orders = orderRepository.findAllByString(new OrderSearch());
        List<SimpleOrderDto> result = orders.stream()
                .map(o -> new SimpleOrderDto(o))
                .collect(Collectors.toList());
        return result;
    }

    /**
     * DTO
     */
    @Data
    static class SimpleOrderDto{
        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;

        // DTO 에 Entity 를 파라미터로 받는 건 크게 문제가 되지 않는다.
        // 별로 중요하지 않은 곳에서 중요한 엔티티를 의존하기 때문에
        public SimpleOrderDto(Order order) {
            this.orderId = order.getId();
            // LAZY 초기화?
            // Member 의 ID 를 가지고 영속성 컨텍스트에서 정보를 찾아본다.
            // 정보가 없으면 DB Query 를 날린다.
            this.name = order.getMember().getName(); // LAZY 초기화
            this.orderDate = order.getOrderDate();
            this.orderStatus = order.getStatus();
            this.address = order.getDelivery().getAddress(); // LAZY 초기화
        }
    }

    /**
     * V3. 엔티티를 조회해서 DTO 로 변환(fetch join 사용)
     * - fetch join 으로 쿼리 1번 호출
     * 참고: fetch join 에 대한 자세한 내용은 JPA 기본편 참고(정말 중요함)
     */
    @GetMapping("/api/v3/sample-orders")
    public List<SimpleOrderDto> ordersV3(){
        List<Order> orders = orderRepository.findAllWithMemberDelivery();
        List<SimpleOrderDto> result = orders.stream()
                .map(o -> new SimpleOrderDto(o))
                .collect(Collectors.toList());
        return result;
    }

    /**
     * V4. JPA 에서 DTO 로 바로 조회 * - 쿼리 1번 호출
     * - select 절에서 원하는 데이터만 선택해서 조회
     */
    @GetMapping("/api/v4/sample-orders")
    public List<OrderSimpleQueryDto> ordersV4(){
       return orderSimpleQueryRepository.findOrderDtos();
    }
}
```
### V3. 엔티티를 조회해서 DTO 로 변환(fetch join 사용)
<pre>
V2 에서 일반조인을 했을 때에는 쿼리가 반복적으로 호출되어 성능이 저하되었다.
- Order 의 결과가 2건이므로 order select 1번 + member select 2번 + delivery select 2번 총 5번이 쿼리가 호출되었다.

<b>Fetch  Join</b>
- <b>JPQL에서 성능 최적화를 위해 제공하는 기능</b>
- 엔티티를 페치 조인(fetch join)을 사용해서 연관된 Entity들을 SQL 한번으로 조회할 수 있다 - <b>성능 최적화</b>
- 데이터베이스로 전송되는 실제 SQL을 보면 <b>FetchTpye.EAGER를 사용한 즉시로딩 전략으로 조회한 것과 SQL이 같다는 것</b>을 알 수 있다.
  (엔티티에서 지연로딩 전략으로 세팅을 해도 JPQL에서 페치조인으로 날리는 것이 우선순위)
- 페치조인은 객체 그래프를 유지할 때 사용하면 효과적이다.
- <b>여러 테이블을 Join해서 Entity가 가진 모양이 아닌, 전혀 다른 결과를 내야 하면,
  페치 조인보다는 일반 조인을 사용하고 필요한 데이터들만 조회해서 DTO로 변환하는 것이 효과적이다.</b>

<b>페치조인과 일반조인의 차이점</b>
- <b>일반조인</b> : JPQL은 결과를 반환할 때 연관관계 고려 X, <b>SELECT절에 지정한 Entity만 조회</b>할 뿐
- <b>페치조인</b> : 페치조인을 사용할때만 <b>연관된 Entity도 함께 조회(즉시로딩)</b>, 페치조인은 객체 그래프를 SQL 한번에 조회하는 개념

<b>페치 조인의 한계</b>
- 페치 조인 대상에는 별칭을 줄 수 없다(하이버네이트는 가능하지만 가급적 사용을 권장하지 않는다)
- 둘 이상의 컬렉션은 페치조인을 할 수 없다.
- 컬렉션을 페치조인하면 페이징 API를 사용할 수 없다.
</pre>
```java
// Repository
// 페치 조인 최적화 메소드
public List<Order> findAllWithMemberDelivery() {
    return em.createQuery(
            "select o from Order o" +
                    " join fetch o.member" +
                    " join fetch o.delivery d", Order.class
    ).getResultList();
}
```
### V4. JPA에서 DTO로 바로 조회
<pre>
<b>컨트롤러단에서 엔티티를 조회하여 DTO로 변환하지 않고, 리포지토리에서 DTO를 이용하여 바로 조회한다.</b>
- 일반적인 SQL을 사용할 때 처럼 원하는 값을 선택하여 조회한다.
- new 명령어를 사용하여 JPQL의 결과를 DTO로 즉시 변환
- SELECT절에서 원하는 데이터를 직접 선택하므로 DB -> 애플리케이션 네트웍 용량 최적화(생각보다 미비)
- <b>리포지토리 재사용성이 떨어짐, API 스펙에 맞춘 코드가 리포지토리에 들어가는 단점이다.</b>
    * 리포지토리는 가급적 순수한 엔티티를 조회하는데 사용하기 때문에 따로 DTO로 조회하는 리포지토리를 만드는 것이 유지보수성에 좋다.
      <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA2/img/OrderSimpleQueryRepository.PNG"/>

<b>쿼리 방식 선택 권장 순서</b>
1. 우선 엔티티를 DTO로 변환하는 방법을 선택한다.
2. 필요하면 페치 조인으로 성능을 최적화 한다. 대부분의 성능 이슈가 해결된다.
3. 그래도 안되면 DTO로 직접 조회하는 방법을 사용한다.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접 사용한다.
</pre>
```java
package jpabook.jpashop.repository.order.simplequery;

@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {

    public final EntityManager em;

    // Repository 는 가급적 순수한 엔티티를 조회하는데 사용하기 때문에 따로 Repository 를 만드는 것이 유지보수성에 좋다.
    // JPA 에서 DTO 로 바로 조회 메소드(재사용 불가)
    public List<OrderSimpleQueryDto> findOrderDtos() {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                        " from Order o" +
                        " join o.member m "+
                        " join o.delivery d ", OrderSimpleQueryDto.class
        ).getResultList();
    }
}
```
```java
package jpabook.jpashop.repository.order.simplequery;

/**
 * Repository 에서 Controller 안의 static class SimpleOrderDto 를 조회하여
 * Controller 에 의존관계가 생기면 안되므로 새로 DTO 를 만들어 주었다.
 * 의존관계는 Repository -> Service -> Controller 로 한 방향으로 흘러가야 한다.
 */
@Data
public class OrderSimpleQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public OrderSimpleQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}
```
