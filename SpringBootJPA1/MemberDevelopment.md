# Member Domain Development
## EntityManger
<pre>
JPA는 <b>EntityManager</b>와 <b>영속성 컨텍스트</b>를 통해 데이터의 상태 변화를 감지하고 필요한 쿼리를 자동으로 수행한다.

<b>영속성 컨텍스트(Persistence Context)</b>
- 엔티티를 영구 저장하는 환경
- Java 영역에서 데이터를 관리하여 DB 접근을 최적화하는 역할을 담당

<b>엔티티의 생명 주기</b>
- <b>비영속(new/transient)</b> : 영속성 컨텍스트와 전혀 관련없는 상태(순수한 객체 상태)
    * 엔티티 객체를 생성 했으나, 저장하지 않은 상태 (순수한 객체 상태)
    * Member member = new Member();

- <b>영속(managed)</b> : 영속성 컨텍스트에 저장된 상태
    * 엔티티 매니저를 통해서 엔티티를 영속성 컨텍스트에 저장한 상태(관리되는 상태)
    * em.persist(member);

- <b>준영속(detached)</b> : 영속성 컨텍스트에 저장되었다가 분리된 상태
    * em.detach() : 엔티티를 분리해 준영속 상태로 만듬
    * em.close() : 영속성 컨텍스트 닫음
    * em.clear() : 양속성 컨텍스트 초기화

- <b>삭제(removed)</b> : 삭제된 상태
    * 엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제
    * em.remove(member);

<b>JPQL(객체지향 쿼리 언어)</b>에 대한 자세한 내용은 이곳을 <a href="https://joont92.github.io/jpa/JPQL/">참조</a>
</pre>
## Member Repository
```java
@Repository //Spring Bean 으로 등록
public class MemberRepository {

    @PersistenceContext // EntityManager 의존성 주입
    private EntityManager em;

    // 회원 등록
    public void save(Member member){
        // 엔티티 매니저를 사용해서 회원 엔티티를 영속성 컨텍스트에 저장
        em.persist(member);
    }

    // 아이디로 개별 회원 검색
    public Member findOne(Long id){
        // find() 메서드는 식별자를 통해서만 데이터 조회
        // find(반환타입, PK)
        return em.find(Member.class, id);
    }

    // 전체 회원 조회
    public  List<Member> findAll(){
        // em.createQuery("JPQL", 반환타입);
        // JPQL 은 Entity(객체)를 대상으로 쿼리를 한다.
        return em.createQuery("select m from Member m", Member.class)
                .getResultList(); // 결과를 컬렉션으로 반환
    }

    // 이름으로 회원 검색
    public List<Member> findByName(String name){
        return em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name) // name 파라미터 바인딩
                .getResultList(); // 결과를 컬렉션으로 반환
    }
}
```
## Transaction
<pre>
<b>Transaction</b>은 <b>데이터베이스의 상태를 변경하는 작업 또는 한번에 수행되어야 하는 연산들</b>을 의미한다.
- JPA 의 모든 데이터 변경이나 로직들은 트랜잭션 안에서 실행 되어야 한다.
- <b>begin, commit</b>을 자동으로 수행해준다.
- 예외 발생 시 <b>rollback</b> 처리를 자동으로 수행해준다.

<b>트랜잭션의 4가지 성질</b>
- <b>원자성(Atomicity)</b> : 한 트랜잭션 내에서 실행한 <b>작업들은 하나의 단위로 처리</b>한다(모두 성공 또는 모두 실패)
- <b>일관성(Consistency)</b> : 트랜잭션은 <b>일관성 있는 데이타베이스 상태를 유지</b>한다(data integrity 만족 등)
- <b>격리성(Islation)</b> : 동시에 실행되는 트랜잭션들이 <b>서로 영향을 미치지 않도록 격리</b>해야한다.
- <b>영속성(Durability)</b> : <b>트랜잭션을 성공적으로 마치면 결과가 항상 저장</b>되어야 한다.

<b>트랜잭션 처리 방법</b>
- <b>@Transactional</b>을 메소드, 클래스, 인터페이스 위에 추가하여 사용(<b>선언적 트랜잭션</b>)
- 적용된 범위에서는 트랜잭션 기능이 포함된 프록시 객체가 생성되어 commit 혹은 rollback을 진행

<b>@Transactional 옵션</b>
- <b>isolation</b> : 트랜잭션에서 일관성없는 데이터 허용 수준을 설정한다.
- <b>propagation</b> : 트랜잭션 동작 도중 다른 트랜잭션을 호출할 때, 어떻게 할 것인지 지정하는 옵션이다.
- <b>noRollbackFor=Exception.class</b> : 특정 예외 발생 시 rollback하지 않는다.
- <b>rollbackFor=Exception.class</b> : 특정 예외 발생 시 rollback한다.
- <b>timeout</b> : 지정한 시간 내에 메소드 수행이 완료되지 않으면 rollback 한다. (-1일 경우 timeout을 사용하지 않는다)
- <b>readOnly</b> : 트랜잭션을 읽기 전용으로 설정한다.
</pre>
## MemberService
```java
@Service
@Transactional(readOnly = true)
// JPA 의 모든 데이터 변경이나 로직들은 트랜잭션 안에서 실행 되어야 한다.
// readOnly = true -> 트렌잭션 읽기 전용 모드로 설정(plush 가 되지 않는다 - 등록 수정 삭제가 안된다)
// 변경 감지를 위한 스냅샷 비교와 같은 무거운 로직들을 수행하지 않으므로 성능향상에 도움을 준다.
// Class 에 @Transactional(readOnly = true) 를 선언함으로써 모든 메서드에 적용된다.
// 그러므로 조회(읽기)가 아닌 메서드에는 따로 @Transactional 명시해주어야한다.
public class MemberService {

    private final MemberRepository memberRepository;

    @Autowired // 생성자 의존성 주입(권장)
    public MemberService(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }

    // 회원 가입
    @Transactional
    public Long join(Member member){
        validateDuplicateMember(member); // 중복 회원 검증
        memberRepository.save(member);

        return member.getId();
    }

    // 중복 회원 검증
    private void validateDuplicateMember(Member member) {
        // Exception
        // 동시 회원가입을 방지하기 위해 DB 에서 member_name 을 unique 설정해주는 것이 좋다.
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if(!findMembers.isEmpty()){
            throw new IllegalStateException("이미 존재하는 회원입니다.");
            // IllegalStateException : 메소드가 요구된 처리를 하기에 적합한 상태에 있지 않을 때
        }
    }

    // 회원 전체 조회
    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    // 개별 회원 조회
    public Member findOne(Long memberId){
        return memberRepository.findOne(memberId);
    }
}
```