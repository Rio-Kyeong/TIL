# Member API
## REST(REpresentational State Transfer) API
<pre>
<b>REST</b> : HTTP 통신에서 어떤 자원에 대한 CRUD 요청을 <b>Resource와 Method로 표현</b>하여 특정한 형태(json, xml)로 전달하는 방식

<b>REST API</b> : REST의 원리를 따르는 API

- REST의 구성요소

    <b>Resource(자원) - HTTP URI</b>
    서버는 Unique한 ID를 가지는 Resource를 가지고 있으며, 클라이언트는 이러한 Resource에 요청을 보낸다.
    이러한 Resource는 <b>URI</b>에 해당한다.

    <b>Method(행위) - HTTP Method</b>
    서버에 요청을 보내기 위한 방식으로 <b>GET, POST, PUT, PATCH, DELETE</b>가 있다.
    CRUD 연산 중에서 처리를 위한 연산에 맞는 Method를 사용하여 서버에 요청을 보내야 한다.

    Create(생성) - 데이터 생성(POST)
    Read(읽기) - 데이터 조회(GET)
    Update(갱신) - 데이터 수정(PUT or POST)
    Delete(삭제) - 데이터 삭제(DELETE)

    <b>Representation of Resource(자원의 형태) - HTTP Message Pay Load</b>
    클라이언트와 서버가 데이터를 주고받는 형태로 <b>json, xml, text, rss</b> 등이 있다.
    최근에는 Key, Value를 활용하는 json을 주로 사용한다.
</pre>
## MemberApiController
<pre>
<b>주의할점</b>
- API를 만들 때는 컨트롤러단에서 <b>엔티티를 직접 반환 하거나(GET방식)</b>, <b>엔티티를 파라미터로 받아서(POST방식)</b> 외부에 엔티티를 노출하면 안된다.

그러므로 <b>외부에 Entity를 노출하지 않고 API 스팩에 맞는 별도의 객체(DTO)를 만드는것</b>이 좋다.
- DTO를 사용하면 Entity와 API 스펙을 명확하게 분리할 수 있다.
- DTO를 사용하면 엔티티의 정보가 변경되어도 API의 스펙이 바뀌지 않는다.(API에 영향이 없다)
- DTO를 사용하면 API 스펙(파라미터로 넘어오는 값)이 무엇인지 정확히 알 수 있다.

<b>@RequestBody</b> : Json(application/json) 형태의 HTTP Body 내용을 Java Object 로 변환시켜주는 역할
</pre>
### 데이터 조회(GET)
<pre>

<b>Java Collectors 사용</b>

List<Member> findMembers = memberService.findMembers();
List<MemberDto> collect = findMembers.stream()
        .map(m -> new MemberDto(m.getName()))
        .collect(Collectors.toList());

- stream() : List를 stream으로 변환해준다.
- map((파라미터) -> 코드) : 스트림 내 요소들을 하나씩 특정 값으로 변환해준다(람다식 이용)
  스트림에 들어가 있는 값이 input이 되어서 특정 로직을 거친 후 output이 되어 리턴되는 새로운 스트림에 담기게 된다.
- collect(Collectors.toList()) : Stream의 요소를 List 인스턴스로 반환해준다.

자세한 내용은 이곳을 <a href="https://wakestand.tistory.com/419">참조</a>
</pre>
```java
/**
* 조회 V1: 응답 값으로 엔티티를 직접 외부에 노출한다(잘못된 예시)
* 문제점
* - 엔티티에 프레젠테이션 계층을 위한 로직이 추가된다.
* - 기본적으로 엔티티의 모든 값이 노출된다(노출을 막기위해 @JsonIgnore 추가)
* - 응답 스펙을 맞추기 위해 로직이 추가된다. (@JsonIgnore, 별도의 뷰 로직 등등)
* - 실무에서는 같은 엔티티에 대해 API가 용도에 따라 다양하게 만들어지는데,
*   한 엔티티에 각각의 API를 위한 프레젠테이션 응답 로직을 담기는 어렵다.
* - 엔티티가 변경되면 API 스펙이 변한다.
* - 추가로 컬렉션을 직접 반환하면 항후 API 스펙을 변경하기 어렵다.(별도의 Result 클래스생성으로 해결)
* 결론
* - API 응답 스펙에 맞추어 별도의 DTO를 반환한다.
*/
@GetMapping("/api/v1/members")
public List<Member> membersV1(){
    return memberService.findMembers();
}

@GetMapping("/api/v2/members")
public Result memberV2(){

    // Entity(List<Member>)를 DTO(List<MemberDto>)로 변환
    List<Member> findMembers = memberService.findMembers();
    List<MemberDto> collect = findMembers.stream()
            .map(m -> new MemberDto(m.getName()))
            .collect(Collectors.toList());

    // List 를 바로 반환하면 배열 타입으로 나가기 때문에 유연성이 떨어진다.
    // 그러므로 Result 제네릭 클래스로 컬렉션을 감싸서 향후 필요한 필드를 추가할 수 있다.
    // 리스트가 아닌 클래스로 반환되므로 배열 타입으로 나가지 않는다.
    return new Result(collect);
}

@Data
@AllArgsConstructor
static class Result<T> {
    private T data;
}

// 조회시켜줄 정보만 선언한다(이름만 반환)
@Data
@AllArgsConstructor
static class MemberDto{
    private String name;
}
```
### 데이터 생성(POST)
```java
/**
* 등록 V1: 요청 값으로 Member 엔티티를 직접 받는다(잘못된 예시)
* 문제점
* - 엔티티에 프레젠테이션 계층을 위한 로직이 추가된다.
* - 엔티티에 API 검증을 위한 로직이 들어간다. (@NotEmpty 등등)
* - 실무에서는 회원 엔티티를 위한 API가 다양하게 만들어지는데, 한 엔티티에
*   각각의 API를 위한 모든 요청 요구사항을 담기는 어렵다.
* - 엔티티가 변경되면 API 스펙이 변한다.
* 결론
* - API 요청 스펙에 맞추어 별도의 DTO를 파라미터로 받는다.
*/
@PostMapping("/api/v1/members")
public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member){
    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}

// @RequestBody : Json(application/json) 형태의 HTTP Body 내용을 Java Object 로 변환시켜주는 역할
// 요청이 오면 CreateMemberRequest 에 회원가입 정보가 바인딩 된다.
@PostMapping("api/v2/members")
public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request){

    Member member = new Member();
    member.setName(request.getName());

    Long id = memberService.join(member);

    return new CreateMemberResponse(id);
}

//DTO
@Data
static class CreateMemberRequest {
    @NotEmpty
    private String name;
}

@Data
static class CreateMemberResponse {
    private  Long id;

    public CreateMemberResponse(Long id) {
        this.id = id;
    }
}
```
### 데이터 수정(PUT)
<pre>
<b>회원(Member) 수정</b>
API updateMemberV2 은 회원 정보를 부분 업데이트 한다.
여기서 PUT 방식을 사용했는데, PUT은 전체 업데이트를 할 때 사용하는 것이 맞다.
부분 업데이트를 하려면 PATCH를 사용하거나 POST를 사용하는 것이 REST 스타일에 맞다.

<b>Command(update) 와 Query(findOne) 를 별도로 분리하는 것이 좋다(유지보수성의 증대)</b>
- Command : 결과를 반환하지 않고, 대신 시스템의 상태를 변화시킨다.
- Query : 결과값을 반환하고, 시스템의 관찰가능한 상태를 변화시키지 않는다. 따라서 부작용에서 자유롭다.
</pre>
```java
//MemberSerivce
//변경 감지 기능(Dirty Checking)
@Transactional
public void update(Long id, String name) {
    // 영속 상태의 엔티티를 조회
    Member member = memberRepository.findOne(id);
    // 엔티티의 데이터를 변경
    member.setName(name);
}
```
```java
// @RequestBody : Json(application/json) 형태의 HTTP Body 내용을 Java Object 로 변환시켜주는 역할
// 요청이 오면 UpdateMemberRequest 에 수정된 정보가 바인딩 된다.
@PutMapping("/api/v2/members/{id}")
public UpdateMemberResponse updateMemberV2(
        @PathVariable("id") Long id,
        @RequestBody @Valid UpdateMemberRequest request){

    // Command(update) 와 Query(findOne) 를 별도로 분리
    memberService.update(id, request.getName());
    Member findMember = memberService.findOne(id);
    return new UpdateMemberResponse(findMember.getId(), findMember.getName());
}

//DTO
@Data
static class UpdateMemberRequest {
    private String name;
}

@Data
@AllArgsConstructor
static class UpdateMemberResponse {
    private Long id;
    private String name;
}
```