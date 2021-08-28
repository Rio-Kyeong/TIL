# Item Web Development
## Item
<pre>
<b>Form 과 DTO 의 차이</b>
    - 공통점 : Form이나 DTO나 모두 <b>단순히 계층간에 데이터를 전달할 때 사용</b>한다.
    - <b>Form</b>은 제약을 더 두어서 명확하게 컨트롤러 까지만 사용해야 한다는 의미를 둔다.
    - <b>DTO</b>는 어디에 정의해두는가에 따라 다르겠지만, 서비스에서도 사용할 수 있고, 리포지토리에서도 사용할 수 있다.
</pre>
### BookForm
```java
// 상품(Book)의 데이터 전달을 위한 객체
@Getter @Setter
public class BookForm {

    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

    private String author;
    private String isbn;
}
```
### ItemController
```java
@Controller
@RequiredArgsConstructor
public class ItemController {

    private final ItemService itemService;

    //상품 등록 페이지(BookForm 으로 관리)
    @GetMapping("/items/new")
    public String createForm(Model model){
        model.addAttribute("form", new BookForm());
        return "items/createItemForm";
    }

    //상품 등록(Submit)
    @PostMapping("/items/new")
    public String create(BookForm form){
        Book book = Book.createBook
                (form.getName(), form.getPrice(), form.getStockQuantity(), form.getAuthor(), form.getIsbn());

        itemService.saveItem(book);
        return "redirect:/";
    }

    //상품 목록 페이지(List)
    @GetMapping("/items")
    public String list(Model model){
        List<Item> items = itemService.findItems();
        model.addAttribute("items", items);
        return "items/itemList";
    }

    //상품 목록 수정 페이지
    @GetMapping("/items/{itemId}/edit")
    public String updateItemForm(@PathVariable("itemId") Long itemId, Model model){
        
        //수정하기 위한 데이터 찾기
        Book book = (Book) itemService.findOne(itemId);

        //BookForm 에 찾은 데이터를 넣어서 수정 페이지에 보내준다.
        BookForm form = new BookForm();
        form.setId(book.getId());
        form.setName(book.getName());
        form.setPrice(book.getPrice());
        form.setStockQuantity(book.getStockQuantity());
        form.setAuthor(book.getAuthor());
        form.setIsbn(book.getIsbn());

        model.addAttribute("form", form);
        return "items/updateItemForm";
    }

    //상품 수정(변경 감지 기능 권장)
    @PostMapping("/items/{itemId}/edit")
    public String updateItem(@PathVariable("itemId") Long itemId, @ModelAttribute("form") BookForm form){

        // merge(병합)을 이용한 update
        // Book book = Book.updateBook
        //  (form.getId(),form.getName(), form.getPrice(), form.getStockQuantity(), form.getAuthor(), form.getIsbn());

        // itemService.saveItem(book); // id가 null 이 아니면 merge(book)를 한다.
        
        // 변경 감지 기능을 이용한 update
        // 트랜잭션이 있는 서비스 계층에 식별자(id)와 변경할 데이터를 명확하게 전달한다(파라미터 or dto)
        // 해당 예제에서는 저자와 ISBN 외 나머지 값을 업데이트 할 수 있는 로직을 구현했다.
        itemService.updateItem(itemId, form.getName(), form.getPrice(), form.getStockQuantity());
        return "redirect:/items";
    }

}
```
## 변경 감지와 병합(merge)
<pre>
상품 수정(updateItem()) 메서드 에서는 준영속 엔티티를 업데이트하고 있다.

<b>준영속 엔티티</b>
    - <b>영속성 컨텍스트가 더는 관리하지 않는 엔티티</b>를 말한다.
      (여기서는 itemService.saveItem(book) 에서 수정을 시도하는 Book 객체다.
      Book 객체는 이미 DB에 한번 저장되어서 식별자가 존재한다.
      이렇게 임의로 만들어낸 엔티티도 기존 식별자를 가지고 있으면 준영속 엔티티로 볼 수 있다)

<b>준영속 엔티티를 수정하는 2가지 방법</b>
    - <b>변경 감지 기능(권장)</b>
        영속성 컨텍스트에서 엔티티를 다시 조회한 후에 데이터를 수정하는 방법이다.

        <b>영속 상태의 엔티티는 변경 감지(dirty checking)기능이 동작해서 값이 셋팅(set)된 후
        트랜잭션을 커밋할 때 자동으로 수정되므로 별도의 수정 메서드를 호출할 필요가 없고 그런 메서드도 없다.</b>

    - <b>병합(merge)</b>
        병합은 준영속 상태의 엔티티를 영속 상태로 변경할 때 사용하는 기능이다.

        <b>병합(merge)시 동작 방식</b>
        1. 준영속 엔티티의 식별자 값으로 영속 엔티티를 조회한다.
        2. 영속 엔티티의 값을 준영속 엔티티의 값으로 모두 교체한다.(병합한다)
        3. 트랜잭션 커밋 시점에 변경 감지 기능이 동작해서 데이터베이스에 UPDATE SQL이 실행

        <b>주의</b>
        변경 감지 기능을 사용하면 <b>원하는 속성만 선택해서 변경</b>할 수 있지만, 병합을 사용하면 <b>모든 속성이 변경</b>된다.
        <b>병합시 값이 없으면 null 로 업데이트 할 위험</b>도 있다. (병합은 모든 필드를 교체한다)
</pre>
```java
// ItemService 

    // 변경 감지 기능 사용
    // 파라미터 값이 많을 경우 DTO 를 따로 만들어서 받는 방법을 사용하면 된다.
    @Transactional
    public void updateItem(Long itemId, String name, int price, int stockQuantity){
        // findItem 의 상태는 영속상태이다.
        // 영속 상태의 엔티티는 변경 감지(dirty checking)기능이 동작해서 값이 셋팅(set)된 후 
        // 트랜잭션을 커밋할 때 자동으로 수정되므로 별도의 수정 메서드를 호출할 필요가 없고 그런 메서드도 없다.
        // (영속성 컨텍스트에서 변경된 엔티티를 찾고 바뀐값을 update 한다)
        Item findItem = itemRepository.findOne(itemId);

        // setter method 보다는 엔티티에서 의미있는 변경 메서드를 하나 만들어서 값을 주입 해주는게 좋다.
        // EX) findItem.change(price, name, stockQuantity);
        findItem.setName(name);
        findItem.setPrice(price);
        findItem.setStockQuantity(stockQuantity);
    }
```
<pre>
가장 좋은 해결 방법

<b>엔티티를 변경할 때는 항상 변경 감지를 사용하세요</b>

- 컨트롤러에서 어설프게 엔티티를 생성하지 않는다.
- 트랜잭션이 있는 서비스 계층에 식별자( id )와 변경할 데이터를 명확하게 전달한다(파라미터 or dto)
- 트랜잭션이 있는 서비스 계층에서 영속 상태의 엔티티를 조회하고, 엔티티의 데이터를 직접 변경한다.
- 트랜잭션 커밋 시점에 변경 감지가 실행된다.
</pre>
