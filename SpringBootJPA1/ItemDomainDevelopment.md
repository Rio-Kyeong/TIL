# Item Domain Development
## ItemRepository
```java
@Repository
// final field 에 대한 생성자를 생성해준다.
// 의존성 주입(Dependency Injection) 편의성을 위해 사용
@RequiredArgsConstructor
public class ItemRepository {

    // @PersistenceContext 또는 @Autowired 를 정의하지 않아도
    // 자동으로 스프링이 생성자에 의존성 주입을 해준다.
    private final EntityManager em;

    //상품 추가
    public void save(Item item){
        if(item.getId() == null){
            // 상품 신규 등록
            em.persist(item);
        } else {
            // 상품 업데이트
            // merge 는 한번 persist(영속) 상태였다가 detached(준영속) 된 상태에서
            // 그 다음 persist(영속) 상태가 될 때, merge(병합) 한다고 한다.
            em.merge(item);
        }
    }

    // 개별 상품 조회
    public Item findOne(Long id){
        return em.find(Item.class, id);
    }

    // 전체 상품 조회
    public List<Item> findAll(){
        return em.createQuery("select i from Item i", Item.class)
                .getResultList();
    }
}
```
## ItemService
```java
@Service
@Transactional(readOnly = true) //읽기 전용
@RequiredArgsConstructor // final field 에 대한 생성자 생성
public class ItemService {

    private final ItemRepository itemRepository;
    
    @Transactional
    public void saveItem(Item item){
        itemRepository.save(item);
    }

    public List<Item> findItems(){
        return  itemRepository.findAll();
    }

    public Item findOne(Long itemId){
        return itemRepository.findOne(itemId);
    }
}
```