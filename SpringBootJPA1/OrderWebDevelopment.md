# Order Web Development
## Order
<pre>
<b>@RequestParam("가져올 데이터의 name") [데이터타입] [변수]</b>
    - 사용자가 요청시 전달하는 값을 Handler(Controller)의 매개변수로 1:1 맵핑할 때 사용되는 어노테이션이다.
    - <b>HttpServletRequest</b> 의 getParameter() 메서드와 같은 역할을 한다.
    - 전달되는 url 상에서 데이터를 찾는다.

<b>@ModelAttribute("Object")</b>
    - 사용자가 요청시 전달하는 값을 오브젝트 형태로 매핑해준다.
    - view단에서 /orders?memberName=KyeongWoo+Ryu&orderStatus=ORDER 로 요청을 하면 각각의 값이 핸들러의 OrderSearch 객체로 바인딩된다.
      (setter가 존재해야 한다.)
</pre>
### OrderSearch
```java
@Getter @Setter
public class OrderSearch {

    private String memberName; //회원 이름
    private OrderStatus orderStatus; // 주문 상태[ORDER, CANCEL]
}
```
### OrderController
```java
@Controller
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;
    private final MemberService memberService;
    private final ItemService itemService;

    //상품 주문 페이지
    @GetMapping("/order")
    public String createForm(Model model){

        List<Member> members = memberService.findMembers();
        List<Item> items = itemService.findItems();

        model.addAttribute("members", members);
        model.addAttribute("items", items);

        return "order/orderForm";
    }

    //상품 주문
    //@RequestParam("가져올 데이터의 name") [데이터타입] [변수]
    @PostMapping("/order")
    public String order(@RequestParam("memberId") Long memberId,
                        @RequestParam("itemId") Long itemId,
                        @RequestParam("count") int count){

        orderService.order(memberId, itemId, count);
        // 핵심 비지니스 로직이 있으면 가급적이면 Controller 에서 Entity 를 찾아서
        // 넘기는 것보다는 식별자(id)를 넘겨주고 Service 에서 비지니스 로직을 작성하게 되면
        // 엔티티를 영속 컨텍스트에 존재하는 상태에서 조회할 수 있다(변경 감지(Dirty Checking)을 할 수 있다)
        // 영속 상태의 엔티티는 변경 감지(dirty checking)기능이 동작해서 값이 셋팅(set)된 후
        // 트랜잭션을 커밋할 때 자동으로 수정되므로 별도의 수정 메서드를 호출할 필요가 없고 그런 메서드도 없다.
        return "redirect:/orders";
    }

    //주문 내역 페이지
    //@ModelAttribute("Object") : 사용자가 요청시 전달하는 값을 오브젝트 형태로 매핑해준다.
    @GetMapping("/orders")
    public String orderList(@ModelAttribute("orderSearch") OrderSearch orderSearch, Model model){

        List<Order> orders = orderService.findOrders(orderSearch);
        model.addAttribute("orders", orders);

        return "order/orderList";
    }

    //주문 취소
    @PostMapping("/orders/{orderId}/cancel")
    public String cancelOrder(@PathVariable("orderId") Long orderId){

        orderService.cancelOrder(orderId);

        return "redirect:/orders";
    }
}
```
