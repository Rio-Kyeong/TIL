# Sample Data Input
## Spring Bean Lifecycle Method
<pre>
<b>@PostConstruct</b>
- 객체의 초기화 부분
- 객체가 생성된 후 별도의 초기화 작업을 위해 실행하는 메소드에 선언한다.
- <b>@PostConstruct 어노테이션을 설정해놓은 init 메소드는 WAS가 띄워질 때 실행된다.</b>
- <b>의존성 주입이 끝난 뒤 실행될 메소드에 적용</b>

<b>@PreDestroy</b>
- 마지막 소멸 단계
- 스프링 컨테이너에서 객체(빈)를 제거하기 전에 해야할 작업이 있다면 메소드위에 사용하는 어노테이션.
- <b>close() 하기 직전에 실행 -> ((AbstractApplicationContext) context).close()</b>
</pre>
```java
@PostConstruct //초기화
public void init() {
    System.out.println("bean 객체 생성되는 시점에 하는 작업");
}

@PreDestroy //소멸
public void destory() {
    System.out.println("bean 객체 소멸되는 시점에 하는 작업");
}
```
## InitDb
<pre>
<b>@PostConstruct와 @Transactional 분리해야 하는이유</b>

@PostConstruct는 해당 빈 자체만 생성되었다고 가정하고 호출된다.
해당 빈에 관련된 AOP등을 포함한, 전체 스프링 애플리케이션 컨텍스트가 초기화 된 것을 의미하지는 않는다.

그러나 트랜잭션을 처리하는 AOP등은 스프링의 후 처리기(post processer)가 완전히 동작을 끝내서, 스프링 애플리케이션 컨텍스트의 
초기화가 완료되어야 적용된다.

정리하면 @PostConstruct는 해당빈의 AOP 적용을 보장하지 않기 때문에 @Transactional를 추가하여도 무시되어진다.

자세한 내용은 이곳을 <a href="https://stackoverflow.com/questions/17346679/transactional-on-postconstruct-method">참조</a>
</pre>
```java
// H2Database Sample Data(automatic registration)

@Component
@RequiredArgsConstructor
public class InitDb {

    private final InitService initService;

    @PostConstruct // 초기화 
    public void init(){
        initService.dbInit1();
        initService.dbInit2();
    }

    @Component
    @Transactional
    @RequiredArgsConstructor
    static class InitService{

        private final EntityManager em;

        public void dbInit1() {
            Member member = createMember("userA", "서울", "1", "1111");
            em.persist(member);

            Book book1 = createBook("JPA1 BOOK", 10000, 100);
            em.persist(book1);

            Book book2 = createBook("JPA2 BOOK", 20000, 100);
            em.persist(book2);

            OrderItem orderItem1 = OrderItem.createOrderItem(book1, 10000, 1);
            OrderItem orderItem2 = OrderItem.createOrderItem(book2, 20000, 2);

            Order order = Order.createOrder(member, createDelivery(member), orderItem1, orderItem2);
            em.persist(order);
        }

        public void dbInit2() {
            Member member = createMember("userB", "진주", "2", "2222");
            em.persist(member);

            Book book1 = createBook("SPRING1 BOOK", 20000, 200);
            em.persist(book1);

            Book book2 = createBook("SPRING2 BOOK", 40000, 300);
            em.persist(book2);

            Delivery delivery = createDelivery(member);

            OrderItem orderItem1 = OrderItem.createOrderItem(book1, 20000, 3);
            OrderItem orderItem2 = OrderItem.createOrderItem(book2, 40000, 4);

            // 주문상품은 여러개(배열)로 넣을 수 있다
            // public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems)
            Order order = Order.createOrder(member, delivery, orderItem1, orderItem2);
            em.persist(order);
        }

        private Member createMember(String name, String city, String street, String zipcode) {
            Member member = new Member();
            member.setName(name);
            member.setAddress(new Address(city, street, zipcode));
            return member;
        }

        private Book createBook(String name, int price, int stockQuantity) {
            Book book = new Book();
            book.setName(name);
            book.setPrice(price);
            book.setStockQuantity(stockQuantity);
            return book;
        }

        private Delivery createDelivery(Member member) {
            Delivery delivery = new Delivery();
            delivery.setAddress(member.getAddress());
            return delivery;
        }
    }
}
```
