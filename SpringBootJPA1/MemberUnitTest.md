# Member unit test
## JUnit
<pre>
<b>JUnit</b>은 <b>Java에서 독립된 단위테스트(Unit Test)를 지원해주는 프레임워크</b>이다.

<b>단위테스트(Unit Test) 란?</b>
- 소스 코드의 특정 <b>모듈이 의도된 대로 정확히 작동하는 지 검증</b>하는 절차
  (즉 모든 함수와 메소드에 대한 테스트 케이스를 작성하는 절차를 뜻한다)

<b>테스트를 지원하는 단정(Assert) 메서드</b>
- <b>assertEquals(a,b)</b> : 객체 A와 B의 값이 같은지 확인 
- <b>assertArrayEquals(a,b)</b> : 배열 A와 B가 일치함을 확인
- <b>assertSame(a,b)</b> : 객체 A와 B가 같은 객체(래퍼런스)임을 확인
- <b>assertTrue(a)</b> : 조건 A가 참인가를 확인
- <b>assertNotNull(a)</b> : 객체 A가 null이 아님을 확인

자세한 내용은 이곳을 <a href="https://junit.org/junit4/javadoc/4.13/org/junit/Assert.html">참조</a>
</pre>
## MemberSerivceTest
```java
//JUnit4 을 실행할 때 Spring 과 같이 엮어서 실행 하기위해 사용
@RunWith(SpringRunner.class)
//SpringBoot 를 띄운 상태에서 Test 를 진행하기 위해 사용
@SpringBootTest
// @Transactional : begin, commit 을 자동 수행하고 예외를 발생시키면, rollback 처리를 자동 수행해준다.
// @Transactional annotation 이 Test case 에 있게되면 test 가 끝날 때 rollback 을 시킨다.
@Transactional
public class MemberServiceTest {

    @Autowired MemberService memberService;
    @Autowired MemberRepository memberRepository;

    //회원가입(엔티티 매니저를 사용해 회원 엔티티를 영속성 컨텍스트에 저장)이 잘 되는지 확인하는 TestCase
    @Test
    @Rollback(false) // Rollback 이 되지않으므로 메서드 실행 후 데이터가 남는다.
    public void join() throws Exception {
        //given
        Member member = new Member();
        member.setName("kim");

        //when
        Long savedId = memberService.join(member);

        //then
        // 회원가입한 회원과 아이디로 찾은 회원과 같은지?
        Assert.assertEquals(member, memberRepository.findOne(savedId));
    }

    // 아이디가 중복일 경우 IllegalStateException Exception 이 발생하는지 확인하는 TestCase
    // expected : try~catch 를 작성하지 않아도 IllegalStateException Exception 이 발생히면 테스트 성공
    @Test(expected = IllegalStateException.class)
    public void dup() throws Exception {
        //given
        Member member1 = new Member();
        member1.setName("kim");

        Member member2 = new Member();
        member2.setName("kim");

        //when
        memberService.join(member1);
        //try {
        memberService.join(member2); //중복회원 예외가 발생한다!!
        //}catch (IllegalStateException e){
        //    return;
        //}

        //then
        Assert.fail("예외가 발생해야 한다.");
        // 주어진 메시지로 테스트에 실패합니다.
        // fail() 까지 오면 테스트 실패!! 예외를 잡아주자.
    }
}
```
