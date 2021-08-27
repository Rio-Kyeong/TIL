# Order Unit Test
## Unit Test
```
좋은 테스트는 DB 나 Spring 통합 없이 service 에 의존시켜 순수하게 java를 이용해서 메서드를 단위테스트 하는 것이 좋다.
(실무에서는 mockito 라는 편리한 프레임워크를 사용한다)

해당 테스트는 JPA 가 잘 동작하는가를 보기위해 통합하여 테스트를 작성하였다.

실무에서 test code를 짤 때 controller, service, repository에 대해서 모두 다 짜는지...?
- 이 부분은 선택이다. 그런데 우선순위를 따지면 비즈니스 로직이 있는 부분의 테스트가 가장 중요합니다.
```
## OrderServiceTest
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class OrderServiceTest {

    @Autowired EntityManager em;
    @Autowired OrderService orderService;
    @Autowired OrderRepository orderRepository;

    // 주문 테스트 
    @Test
    public void order() throws Exception {
        //given
        Member member = createMember();

        Item book = createBook("시골 JPA",10000,10);

        int orderCount = 2;

        //when
        Long orderId = orderService.order(member.getId(), book.getId(), orderCount);

        //then
        Order getOrder = orderRepository.findOne(orderId);

        assertEquals("상품 주문시 상태는 ORDER", OrderStatus.ORDER, getOrder.getStatus());
        assertEquals("주문한 상품 종류 수가 정확해야 한다.", 1, getOrder.getOrderItems().size());
        assertEquals("총 주문 가격은 가격 * 수량이다.", 10000 * orderCount, getOrder.getTotalPrice());
        assertEquals("주문 수량만큼 재고가 줄어야 한다.",8, book.getStockQuantity());
    }

    // 상품주문 재고수량초과 시 정상적으로 예외처리가 되는지 테스트
    // expected = NotEnoughStockException.class : 해당 예외가 발생하면 예외를 잡아준다(테스트 성공)
    @Test(expected = NotEnoughStockException.class)
    public void stockExcess() throws Exception {
        //given
        Member member = createMember();
        Item item = createBook("시골 JPA", 10000, 10);

        // 재고수량은 10개인데 주문 수량을 11개로 지정
        // 재고수량보다 주문 수량이 많을 경우 NotEnoughStockException 이 발생하도록 해놓음
        int orderCount = 11;

        //when
        orderService.order(member.getId(), item.getId(), orderCount);

        //then
        //예외처리 되지않고 fail에 도착하면 테스트 실패
        fail("재고 수량 부족 예외가 발생");
    }
        
    // 주문 취소 테스트
    @Test
    public void cancelOrder() throws Exception {
        //given
        Member member = createMember();
        Item item = createBook("시골 JPA", 10000, 10);

        int orderCount = 2;

        // 주문 취소를 위한 주문
        Long orderId = orderService.order(member.getId(), item.getId(), orderCount);

        //when
        // 주문취소
        orderService.cancelOrder(orderId);

        //then
        // 재고가 복귀 되었는지 검증
        Order getOrder = orderRepository.findOne(orderId);

        assertEquals("주문 취소시 상태는 CANCEL 이다.",OrderStatus.CANCEL, getOrder.getStatus());
        assertEquals("주문이 취소된 상품은 그만큼 재고가 증가해야 한다.", 10, item.getStockQuantity());
    }

    // 책 상품 정보 입력 후 영속성 컨텍스트에 저장
    private Item createBook(String Name, int price, int stockQuantity) {
        Item book = new Book();
        book.setName(Name);
        book.setPrice(price);
        book.setStockQuantity(stockQuantity);
        em.persist(book);
        return book;
    }

    // 회원 정보 입력 후 영속성 컨텍스트에 저장
    private Member createMember() {
        Member member = new Member();
        member.setName("회원1");
        member.setAddress(new Address("서울","한강","123-123"));
        em.persist(member);
        return member;
    }
}
```
