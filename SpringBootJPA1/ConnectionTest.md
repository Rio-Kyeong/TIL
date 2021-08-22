# Connection Test
## Test Case
```
H2Database를 연결 후 실제 동작하는지 Test Case 를 만들어서 test 해보기
```
### Entity
```java
@Entity
@Getter @Setter
public class Member {

 @Id @GeneratedValue
 private Long id;

 private String username;
}
```
### Repository
```java
@Repository
public class MemberRepository {

    @PersistenceContext // 엔티티매니저를 주입
    private EntityManager em;

    //정보 저장
    public Long save(Member member){
        //객체를 저장한 상태(영속성 컨텍스트에 저장)
        em.persist(member);
        return member.getId();
    }

    //정보 조회
    public Member find(Long id){
        return em.find(Member.class, id);
    }
}
```
### test
```java
// 경로 : src/test/package/MemberRepositoryTest

@RunWith(SpringRunner.class)
@SpringBootTest
public class MemberRepositoryTest {

    @Autowired MemberRepository memberRepository;

    @Test
    @Transactional
    // @Transactional annotation 이 Test case 에 있으면 
    // test 가 끝나면서 rollback 하여 테이블 정보가 사라진다.
    @Rollback(false) // 롤백을 시키지 않는다.
    public void testMember() throws Exception {
        //given
        Member member = new Member();
        member.setUsername("memberA");

        //when
        Long savedId = memberRepository.save(member);
        Member findMember = memberRepository.find(savedId);

        //then
        Assertions.assertThat(findMember.getId()).isEqualTo(member.getId());
        Assertions.assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
        Assertions.assertThat(findMember).isEqualTo(member);
        // 같은 영속성 컨텍스트 안에서는 식별자가 같으면 같은 엔티티로 식별된다 
        // JPA 엔티티 동일성 보장 -> findMember == member : true
        System.out.println("findMember == member : "+(findMember == member));
    }
}
```