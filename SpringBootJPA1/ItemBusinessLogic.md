# Item Entity Business Logic
## Business Logic
<pre>
Item Entity의 stockQuantity(재고수량) field 가 증가 또는 감소하는 비즈니스 로직을 구현한다.

stockQuantity field 를 변경해야 할 일이 생기면 setter method 를 이용하는 것이 아닌
Entity 에 business logic method 를 정의하여 사용한다(객체지향적)

<b>Data 를 가지고 잇는 Entity 쪽에 비지니스 메서드가 존재하는것이 가장 좋다(응집력이 높다)<b>
</pre>
## Item Entity
```java
@Entity
//single_table 전략을 사용한다(한개의 테이블에 모든 컬럼 정의)
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
@Getter @Setter
public abstract class Item {

    @Id
    @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();

    //==비즈니스 로직==//
    // Data 를 가지고 잇는 Entity 쪽에 비지니스 메서드가 존재하는것이 가장 좋다(응집력이 높다)
    // stock 증가
    public void addStock(int quantity){
        this.stockQuantity += quantity;
    }

    // stock 감소
    public void removeStock(int quantity){
        int restStock = this.stockQuantity - quantity;
        if(restStock < 0){
            // 재고수량이 0보다 작을경우 예외발생
            throw new NotEnoughStockException("need more stock");
        }
        this.stockQuantity -= quantity;
    }
}
```
## NotEnoughStockException
```java
// 재고수량이 0보다 작을경우 RuntimeException 을 발생시킨다.
public class NotEnoughStockException extends RuntimeException {

    public NotEnoughStockException() {
        super();
    }

    public NotEnoughStockException(String message) {
        super(message);
    }

    public NotEnoughStockException(String message, Throwable cause) {
        super(message, cause);
    }

    public NotEnoughStockException(Throwable cause) {
        super(cause);
    }
}
```