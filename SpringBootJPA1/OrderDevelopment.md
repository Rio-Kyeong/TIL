# Order Domain Development
## Order Repository
<pre>
<b>동적 쿼리 작성하기</b>
1. JAVA 코드 작성 - 복잡하고 노가다성
2. JPA Criteria - 실무에서 적용하기 어렵다.
3. QueryDSL - SQL(JPQL)과 모양이 유사하면서 자바 코드로 동적 쿼리를 편리하게 생성할 수 있다.
    
<b>QueryDSL</b>
- 복잡한 동적 쿼리를 사용할 때, Querydsl을 사용하면 높은 개발 생산성을 얻으면서 동시에 쿼리 오류를 컴파일 시점에 빠르게 잡을 수 있다.
- 꼭 동적 쿼리가 아니라 정적 쿼리인 경우에도 다음과 같은 이유로 Querydsl을 사용하는 것이 좋다.
- QueryDSL 을 사용하기 위해서는 별도의 Q클래스를 생성해야 한다.

<b>장점</b>
- 직관적인 문법
- 컴파일 시점에 빠른 문법 오류 발견
- 코드 자동완성
- 코드 재사용(이것은 자바다)
- JPQL new 명령어와는 비교가 안될 정도로 깔끔한 DTO 조회를 지원한다.

Querydsl은 JPQL을 코드로 만드는 빌더 역할을 할 뿐이다. 따라서 JPQL을 잘 이해하면 금방 배울 수 있다.
</pre>
```java
@Repository
@RequiredArgsConstructor
public class OrderRepository {

    private final EntityManager em;

    // 주문하기
    public void save(Order order){
        em.persist(order);
    }

    // 주문 단건 검색
    public Order findOne(Long id){
        return em.find(Order.class, id);
    }

    /**
     * JAVA 코드 작성
     * 주문 내역 검색 동적쿼리 방법1
     */
    public List<Order> findAllByString(OrderSearch orderSearch) {

        //language = JPQL
        String jpql = "select o From Order o join o.member m";

        boolean isFirstCondition = true;

        //주문 상태 검색
        if (orderSearch.getOrderStatus() != null) {
            if (isFirstCondition) {
                jpql += " where";
                isFirstCondition = false;
            } else {
                jpql += " and";
            }
            jpql += " o.status = :status";
        }

        //회원 이름 검색
        if (StringUtils.hasText(orderSearch.getMemberName())) {
            if (isFirstCondition) {
                jpql += " where";
                isFirstCondition = false;
            } else {
                jpql += " and";
            }
            jpql += " m.name like :name";
        }

        TypedQuery<Order> query = em.createQuery(jpql, Order.class)
                .setMaxResults(1000); //최대 1000건
        if (orderSearch.getOrderStatus() != null) {
            query = query.setParameter("status", orderSearch.getOrderStatus());
        }
        if (StringUtils.hasText(orderSearch.getMemberName())) {
            query = query.setParameter("name", orderSearch.getMemberName());
        }
        return query.getResultList();
    }

    /**
     * JPA Criteria
     * 주문 내역 검색 동적쿼리 방법2
     */
    public List<Order> findAllByCriteria(OrderSearch orderSearch) {

        // JPA Criteria 로 처리
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<Order> cq = cb.createQuery(Order.class);
        Root<Order> o = cq.from(Order.class);
        Join<Order, Member> m = o.join("member", JoinType.INNER); //회원과 조인
        List<Predicate> criteria = new ArrayList<>();

        //주문 상태 검색
        if (orderSearch.getOrderStatus() != null) {
            Predicate status = cb.equal(o.get("status"),
                    orderSearch.getOrderStatus());
            criteria.add(status);
        }

        //회원 이름 검색
        if (StringUtils.hasText(orderSearch.getMemberName())) {
            Predicate name =
                    cb.like(m.<String>get("name"), "%" +
                            orderSearch.getMemberName() + "%");
            criteria.add(name);
        }
        cq.where(cb.and(criteria.toArray(new Predicate[criteria.size()])));
        TypedQuery<Order> query = em.createQuery(cq).setMaxResults(1000); //최대 1000건
        return query.getResultList();
    }

    /**
     * Querydsl
     * 주문 내역 검색 동적쿼리 방법3
     */
    public List<Order> findAll(OrderSearch orderSearch) {

        JPAQueryFactory query = new JPAQueryFactory(em);
        QOrder order = QOrder.order;
        QMember member = QMember.member;
        return query
                .select(order)
                .from(order)
                .join(order.member, member)
                .where(statusEq(orderSearch.getOrderStatus()), nameLike(orderSearch.getMemberName()))
                .limit(1000)
                .fetch();
    }

    private BooleanExpression statusEq(OrderStatus statusCond) {
        if (statusCond == null) {
            return null;
        }
        return order.status.eq(statusCond);
    }

    private BooleanExpression nameLike(String nameCond) {
        if (!StringUtils.hasText(nameCond)) {
            return null;
        }
        return member.name.like(nameCond);
    }
}
```
## Order Service
<pre>
DB SQL 을 직접 다루는(Mybatis, JdbcTemplate) 라이브러리 같은 경우에는
데이터를 변경한다음 밖에서 업데이트 쿼리를 직접 짜서 DB에 날려야한다.

그러나 <b>JPA 를 활용하여 Entity 안에 데이터들만 바꿔주면 알아서 바뀐 변경 포인트를 감지하여
JPA 가 자동으로 업데이트 쿼리를 날려준다</b>(JPA 의 강점)
</pre>
```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final MemberRepository memberRepository;
    private final ItemRepository itemRepository;

    //주문
    @Transactional
    public Long order(Long memberId, Long itemId, int count){

        //엔티티 조회
        Member member = memberRepository.findOne(memberId);
        Item item = itemRepository.findOne(itemId);

        //배송정보 생성
        Delivery delivery = new Delivery();
        //회원의 주소 값으로 배송을 한다(간단한 예제)
        delivery.setAddress(member.getAddress());

        //주문상품 생성
        //static 이기 때문에 객체생성없이 클래스 선언으로 바로 메서드를 부를 수 있다.
        //하나하나 orderItem.setXXX() 을 할 필요없이 createOrderItem 라는 생성 메서드를 만들어 한번에 값을 관리한다.
        //orderItem.setXXX() 을 못하도록 OrderItem 의 생성자의 접근 지정자를 protected 로 설정한다.
        OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);

        //주문 생성
        Order order = Order.createOrder(member, delivery, orderItem);

        //주문 저장
        //order 만 저장(save)해주어도 order entity 의 cascade = CascadeType.ALL 때문에
        //orderItem 과 delivery 가 연쇄적으로 저장된다.
        //cascade = CascadeType.ALL 설정이 없으면 각각 저장해 주어야 한다.
        orderRepository.save(order);

        return order.getId();
    }

    //주문 취소
    @Transactional
    public void cancelOrder(Long orderId){
        //주문 엔티티 조회
        Order order = orderRepository.findOne(orderId);

        //주문 취소
        order.cancel();
    }


    //동적으로 주문 내역 검색(qeurydsl)
    public List<Order> findOrders(OrderSearch orderSearch){
        return orderRepository.findAll(orderSearch);
    }
}
```
