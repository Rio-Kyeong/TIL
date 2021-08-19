# Creating Entity and useing Spring Data JPA 
## Entity
<pre>
Entity 클래스는 <b>실제 DataBase의 테이블과 1 : 1로 매핑 되는 클래스</b>로, <b>DB의 테이블내에 존재하는 컬럼만을 속성(필드)으로</b> 가져야 한다.

<b>@Entity</b> : 해당 클래스의 이름을 가지고 데이터 테이블을 생성하고(실행 시에 데이터베이스 자동 생성)
           해당 클래스의 필드를 가지고 테이블 생성에 필요한 정보를 얻어 컬럼을 생성한다.
<b>@Id</b> : 기본키(Primary key) 설정
<b>@GeneratedValue</b> : key값 자동 생성
</pre>
### User
```java
@Data 
@AllArgsConstructor 
@NoArgsConstructor 
@ApiModel(description = "사용자 상세 정보를 위한 도메인 객체") 
@Entity 
public class User { 

    @Id 
    @GeneratedValue 
    private Integer id; 
    
    @Size(min=2, message = "Name은 2글자 이상 입력해주세요.") 
    @ApiModelProperty(notes = "사용자의 이름을 입력해주세요.") 
    private String name; 
    
    @Past 
    @ApiModelProperty(notes = "사용자의 등록일을 입력해주세요.") 
    private Date joinDate; 
    
    @ApiModelProperty(notes = "사용자의 비밀번호를 입력주세요.") 
    private String password;
    
    @ApiModelProperty(notes = "사용자의 주민번호를 입력해주세요.") 
    private String ssn; 

    // 사용자가 쓴 게시글 정보를 저장하는 필드
    @OneToMany(mappedBy = "user") //1:N 관계
    private List<Post> posts ; //전체 게시글 반환
}
```
### Post
```java
@Entity
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Post {

    @Id
    @GeneratedValue //키 자동 생성
    private Integer id;

    private String description; //게시글 내용

    
    @JsonIgnore 
    //외부에 유저정보를 노출하지 않기 위해 사용
    @ManyToOne(fetch = FetchType.LAZY) //N:1 관계
    // User : Post -> 1 : (0~N) 관계, Main : Sub -> Parent : Child
    // LAZY : 지연 로딩 방식
    // 매번 Post 객체 데이터를 로딩하는것이 아니라 Post 데이터가 로딩되는 시점에 필요한 user(사용자)데이터를 가져온다
    // 한명의 사용자가 여러 개의 게시글을 작성한다
    private User user; // 게시글을 작성한 User 정보
}
```
## Create virtual data
```sql
-- main/resources/data.sql
-- 매번 H2DB에 데이터를 추가하는 것이 아닌 테이블이 로딩되면서 데이터도 같이 등록될 수 있도록 가상 데이터를 만들어준다.
insert into user values(90001, sysdate(), 'User1', 'test1111', '701010-1111111');
insert into user values(90002, sysdate(), 'User2', 'test2222', '801010-2222222');
insert into user values(90003, sysdate(), 'User3', 'test3333', '901010-3333333');

insert into post values (10001, 'My first post', 90001);
insert into post values (10002, 'My second post', 90001);
```
## Spring Data JPA
<pre>
Spring Data JPA는 일반적인 JPA 기능과는 달리 
Entity를 제어하기 위해서 JPA Manager를 사용하지않고 <b>Repository interface를 선언</b>하도록 되어있다.
-> @Repository 선언하는 것만으로도 <b>CRUD 기능을 사용</b>할 수 있다.
</pre>
### Repository interface
```java
package com.example.restfulwebservice.user;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
// 데이터베이스에 관련된 bean임을 명시하는 어노테이션
public interface UserRepository extends JpaRepository<User, Integer> {
        // JpaRepository<Entity, 기본키의 데이터 타입>
}

@Repository
// 데이터베이스에 관련된 bean임을 명시하는 어노테이션
public interface PostRepository extends JpaRepository<Post, Integer> {
        // JpaRepository<Entity, 기본키의 데이터 타입>
}
```
### RestContoroller
<pre>

<b>CRUD</b>
- <b>@PostMapping</b> : C(생성)
- <b>@GetMapping</b> : R(읽기)
- <b>@PutMapping</b> : U(업데이트)
- <b>@DeleteMapping</b> : D(삭제)

<b>JpaRepository</b>
- <b>save()</b> : 레코드 저장(insert, update)
- <b>findOne()</b> : primary key로 레코드 한건 찾기
- <b>findById()</b> : primary key로 레코드 한건 찾기(반환값 : Optional)
- <b>findAll()</b> : 전체 레코드 불러오기. 정렬(sort), 페이징(pageable) 가능
- <b>count()</b> : 레코드 갯수
- <b>delete()</b> : 레코드 삭제
- <b>deleteById()</b> : 레코드 찾기(findById) + 레코드 삭제(delete)

  자세한 내용은 이곳을 <a href="https://jobc.tistory.com/120">참조</a>

<b>Optional<T> findById(ID var1);</b> -> findById은 반환값으로 Optional<T>를 받는다.
- Optional<T> 클래스는 'T' 타입의 객체를 포장해주는 래퍼 클래스이다.
- java 에서 값이 없음을 표현하기 위한 null 값을 그대로 사용하지 않고 Optional 인스턴스로 대체하여
  값이 없음에 대한 예기치 못한 에러 발생으로 부터 안전한 값의 처리를 지원한다는 점이 특징이다.
- get() 메소드를 사용하여 Optional 객체에 저장된 값에 접근할 수 있다.
</pre>
```java
@RestController
@RequestMapping("/jpa")
public class UserJpaController {

    // 저장소 의존성 주입
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private  PostRepository postRepository;

    // http://localhost/jpa/users
    // 전체 회원 조회 : findAll 메소드 사용
    @GetMapping("/users")
    public List<User> retrieveAllUsers(){
        return userRepository.findAll();
    }

    // 개별 회원 조회 : findById 메소드 사용 
    @GetMapping("/users/{id}")
    public EntityModel<User> retrieveUser(@PathVariable int id){

        // Optional<T> findById(ID var1); <- findById()의 반환 값은 Optional<T> 이다.
        // Optional<T> 클래스는 'T' 타입의 객체를 포장해주는 래퍼 클래스이다.
        Optional<User> user = userRepository.findById(id);

        // isEmpty() 메소드를 사용하여 Optional 객체에 지정된 값이 null 인지 아닌지 확인한다.
        if(user.isEmpty()){ // user 가 null 이면 예외발생
            throw new UserNotFoundException(String.format("ID[%s] not found", id));
        }

        // HATEOAS
        // get() 메소드를 사용하면 Optional 객체에 지정된 값에 접근할 수 있다.
        EntityModel<User> resource = EntityModel.of(user.get());
        WebMvcLinkBuilder linkTo = linkTo(methodOn(this.getClass()).retrieveAllUsers());
        // retrieveAllUsers() 메소드와 "all-users" 이름 하이퍼미디어 작업
        resource.add(linkTo.withRel("all-users"));

        return resource;
    }

    // 회원 삭제 : deleteById 메소드 사용
    @DeleteMapping("/users/{id}")
    public void deleteUser(@PathVariable int id){
        userRepository.deleteById(id);
    }

    // 회원 등록 : save 메소드 사용
    @PostMapping("/users")
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        //save(S) : 새로운 엔티티는 저장하고 이미 잇는 엔티티는 수정(insert, update)
        User saveUser = userRepository.save(user);

        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(saveUser.getId())
                .toUri();
        return ResponseEntity.created(location).build();
    }

    // user 의 정보를 받아온 후 그 유저가 작성한 posts 의 정보를 가져온다.(개별 회원의 게시글 조회)
    // /jpa/users/90001/posts
    @GetMapping("/users/{id}/posts")
    public List<Post> retrieveAllPostByUser(@PathVariable int id){

        Optional<User> user = userRepository.findById(id);
        // 리턴 값이 Optional인 이유 : 데이터가 존재할수도 안할수도 있기 때문에

        // isEmpty() 메소드를 사용하여 Optional 객체에 지정된 값이 null 인지 아닌지 확인한다.
        if(user.isEmpty()){ // user 가 null 이면 예외발생
            throw new UserNotFoundException(String.format("ID[%s] not found", id));
        }

        return user.get().getPosts(); //user 의 Posts 를 가져와서 반환
    }

    // 게시글 추가
    @PostMapping("/users/{id}/posts")
    public ResponseEntity<Post> createPost(@PathVariable int id, @Valid @RequestBody Post post) {

        Optional<User> user = userRepository.findById(id);

        if(user.isEmpty()){ // user 가 null 이면 예외발생
            throw new UserNotFoundException(String.format("ID[%s] not found", id));
        }

        post.setUser(user.get()); //user 값 저장
        //save(S) : 새로운 엔티티는 저장하고 이미 잇는 엔티티는 수정
        Post savePost = postRepository.save(post);

        // 사용자에게 요청 값을 변환해주기 
        // fromCurrentRequest() : 현재 요청되어진 request값을 사용한다는 뜻 
        // path : 반환 시켜줄 값 
        // savedPost.getId() : {id} 가변변수에 새롭게 만들어진 id값 저장 
        // toUri() : URI형태로 변환 
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(savePost.getId())
                .toUri();
        return ResponseEntity.created(location).build();
    }
}
```