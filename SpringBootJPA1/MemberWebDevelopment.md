# Member Web Development
## Member
<pre>
Form 대신 Entity 를 사용해도 되나요?
    - Entity를 Form처럼 사용하려면 Entity에 화면에 대한 종속적인 기능도 추가해야한다.
      (Entity의 가독성이 떨어진다 -> 유지보수가 힘들어진다)
    - 또한 view에서 넘어올 때 원하는 validation(검증)과 실제 domain이 원하는 validation(검증)은 다를 수 있다.
      그러므로 <b>Form또는 DTO를 따로 만들어 주는 것이 좋다.</b>
    - <b>Entity는 최대한 순수하게 핵심 비지니스 로직에만 의존성이 있도록 설계</b>해야한다.

Form 과 DTO 의 차이
    - 공통점 : Form이나 DTO나 모두 단순히 계층간에 데이터를 전달할 때 사용한다.
    - Form은 제약을 더 두어서 명확하게 컨트롤러 까지만 사용해야 한다는 의미를 둔다.
    - DTO는 어디에 정의해두는가에 따라 다르겠지만, 서비스에서도 사용할 수 있고, 리포지토리에서도 사용할 수 있다.
</pre>
### MemberForm
```java
// 회원(Member)의 데이터 전달을 위한 객체
@Getter @Setter
public class MemberForm {

    //값이 비어있으면 오류 발생
    @NotEmpty(message = "회원 이름은 필수 입니다")
    private String name;

    private String city;
    private String street;
    private String zipcode;
}
```
### MemberController
```java
@Controller
@RequiredArgsConstructor
public class MemberController {

    private final MemberService memberService;

    //회원 가입 페이지(MemberForm 으로 관리)
    @GetMapping("/members/new")
    public String createForm(Model model){
        model.addAttribute("memberForm", new MemberForm());
        return "members/createMemberForm";
    }

    //회원 가입(데이터 검증 수행)
    //@Valid : 객체에 대한 검증을 수행한다.
    @PostMapping("/members/new")
    public String create(@Valid MemberForm form, BindingResult result){

        // error 가 있으면 다시 회원가입 폼을 반환
        if(result.hasErrors()){
            return "members/createMemberForm";
        }

        Address address = new Address(form.getCity(), form.getStreet(), form.getZipcode());

        Member member = new Member();
        member.setName(form.getName());
        member.setAddress(address);

        memberService.join(member);

        // 회원가입이 완료된 후 페이지가 재로딩되거나 하면 안좋기 떄문에 redirect 로 첫 페이지로 이동한다.
        return "redirect:/";
    }

    // 회원 목록 페이지(List)
    @GetMapping("/members")
    public String list(Model model){
        // 해당 메서드에서는 Member Entity 그대로 사용했지만
        // DTO 를 따로 만들어서 사용하는 것을 권장한다.
        List<Member> members = memberService.findMembers();
        model.addAttribute("members", members);
        return "members/memberList";
    }
}
```
