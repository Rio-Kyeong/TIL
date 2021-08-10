# Entity, DTO, VO
## Entity
<pre>
Entity 클래스는 <b>실제 DataBase의 테이블과 1 : 1로 매핑 되는 클래스</b>로, <b>DB의 테이블내에 존재하는 컬럼만을 속성(필드)으로</b> 가져야 한다.
Entity 클래스는 상속을 받거나 구현체여서는 안되며, 테이블내에 존재하지 않는 컬럼을 가져서도 안된다.

최대한 외부에서 Entity 클래스의 getter method를 사용하지 않도록 해당 클래스 안에서 필요한 로직 method을 구현 해야하고,
Domain Logic만 가지며 Presentation Logic을 가지고 있어서는 안된다.

구현 method는 주로 Service Layer에서 사용한다.
</pre>
### Entity, DTO 를 분리해야 하는 이유
<pre>
Entity와 DTO를 분리해서 관리해야 하는 이유는 <b>DB Layer와 View Layer 사이의 역할을 분리</b> 하기 위해서다.

Entity 클래스는 실제 테이블과 매핑되어 만일 변경되게 되면 여러 다른 클래스에 영향을 끼친다.
DTO 클래스는 View와 통신하며 자주 변경되므로 분리 해주어야 한다.

결국 DTO는 Domain Model 객체를 그대로 두고, 복사하여 다양한 Presentation Logic을 추가한 정도로 사용하며
Domain Model 객체는 Persistent(지속성)만을 위해서 사용해야한다.
</pre>
### Entity Setter 금지 및 생성자, 접근 제어
```
객체의 일관성을 유지할 수 있어야 유지 보수성이 올라가기 떄문에 Setter를 사용해서는 안되며,
객체의 생성자에 값들을 넣어줌으로써 Setter 사용을 줄일 수 있다.
```
### Entity Class
```java
@Getter
@Setter
@ToString
@Table(name = "user")
@Entity
public class User {

    @Id
    @GeneratedValue
    private int id;

    @Column(name = "name", nullable = false)
    private String name;

    @Column(name = "password", nullable = false)
    private String password;

    @Column(name = "email", nullable = false, unique = true)
    private String email;

    @Column(name = "phone", nullable = false, unique = true)
    private String phone;

    @Column(nullable = true)
    private LocalDateTime create_date;

    private LocalDateTime modify_date;
}
```
## DTO(Data Transfer Object)
<pre>
DTO(Data Transfer Object)는 <b>데이터 전송(이동) 객체</b>라는 의미를 가지고 있다.
DTO는 주로 비동기 처리를 할 때 사용한다.

<b>계층간 데이터 교환을 위한 객체(Java Beans)</b>이다.
<b>DB의 데이터를 Service나 Controller 등으로 보낼 때 사용하는 객체</b>를 말한다.

즉, DB의 데이터가 Presentation Logic Tier로 넘어올때는 DTO로 변환되어 오고가는 것이다.
로직을 갖고 있지 않는 순수한 데이터 객체이며, getter/setter 메서드만을 갖는다.
또한 Controller Layer에서 Response DTO 형태로 Client에 전달한다.

<img src="img"/>
</pre>
### DTO와 VO의 차이점
```
VO는 DTO와 동일한 개념이지만 read only 속성을 갖는다.
VO는 특정한 비즈니스 값을 담는 객체이고, DTO는 Layer간의 통신 용도로 오고가는 객체를 말한다.
```
### DTO Class
```java
@Getter @Setter
class UserDTO {
  private String id;
  private String name;
  private String email;
}
```
## VO(Value Object)
<pre>
VO(Value Object)는 말 그대로 <b>값 객체라는 의미</b>를 가지고 있다.

<b>VO의 핵심 역할은 equals()와 hashcode() 를 오버라이딩</b> 하는 것이다.
즉, VO 내부에 선언된 속성(필드)의 모든 값들이 VO 객체마다 값이 같아야, 똑같은 객체라고 판별한다.
- equals : 두 객체의 내용이 같은지, 동등성(equality) 를 비교하는 연산자
- hashCode : 두 객체가 같은 객체인지, 동일성(identity) 를 비교하는 연산자

VO는 Getter와 Setter를 가질 수 있으며, VO는 테이블 내에 있는 속성 외에 추가적인 속성을 가질 수 있으며,
여러 테이블(A, B, C)에 대한 공통 속성을 모아서 만든 BaseVO 클래스를 상속받아서 사용할 수도 있다.
</pre>
### VO Class
```java
@Getter @Setter
@Alias("article")
class ArticleVO {
    private Long id;
    private String title;
    private String contents;

    // @Override
    // public boolean equals(Object o) { ... }
    // @Override
    // public int hashCode() { ... }
```
## 요약
| |**DTO**|**VO**|**Entity**|
|------|------|------|------|
|**용도**|레이어간의 데이터 전송|의미 있는 값을 표현|DB 테이블과 매핑되는 클래스|
|**가변/불변**|**가변객체**</br>생성 후 상태를 변경할 수 있다|**불변객체**</br>생성 후 상태 변경이 없다.|**가변객체** </br>생성 후 상태를 변경할 수 있다|
|**로직 포함 여부**|로직 포함 불가능|로직 포함 가능|로직 포함 가능|
