# 상속 관계 매핑
<pre>
관계형 데이터베이스에서는 객체지향 언어에서의 상속 개념이 없다.
대신 관계형 데이터베이스에서는 <b>슈퍼타입 서브타입 관계</b>라는 모델링 기법이 상속과 유사하다.
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/DBType.PNG"/>
ORM에서 이야기하는 상속 관계 매핑은 <b>객체의 상속 구조와 데이터베이스의 슈퍼타입 서브타입 관계를 매핑하는 것</b>이다.
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/ObjectExtends.PNG"/>
슈퍼타입 서브타입 논리 모델을 실제 물리 모델인 테이블로 구현할 때는 3가지 방법이 있다.
- <b>조인 전략</b> : 각각 테이블을 만들고 조회할 때 조인한다.
- <b>단일(싱글) 테이블 전략</b> : 테이블을 하나만 사용해서 통합한다.
- <b>구현 클래스마다 테이블 전략</b> : 서브 타입마다 하나의 테이블로 만듦

<b>@Inheritance(strategy = InheritanceType.XXX)</b>
- <b>InheritanceType.JOINED</b> : 조인 전략
- <b>InheritanceType.SINGLE_TABLE</b> : 단일 테이블 전략(default)
- <b>InheritanceType.TABLE_PER_CLASS</b> : 구현 클래스마다 테이블 전략

<b>@DiscriminatorColumn(name = "DTYPE")</b>
- 부모 클래스에 선언한다.
- <b>부모테이블에서 자식테이블의 타입을 구분하기 위해 DTYPE 구분컬럼을 사용</b>
- 조인 전략에서는 필수로 선언해주는 것이 좋으며, 단일 테이블 전략에서는 선언하지 않아도 자동으로 들어간다.
- default = DTYPE

<b>@DiscriminatorValue("XXX")</b>
- 자식 클래스에 선언한다.
- 엔티티를 저장할 때 슈퍼타입의 <b>구분 컬럼에 저장할 값을 지정</b>한다.
- 어노테이션을 선언하지 않을 경우 기본값으로 클래스 이름이 들어간다.
</pre>
## 조인 전략
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/JoinStrategy.PNG"/>
- 엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서 기본키 + 외래키(PK,FK)로 사용하는 전략이다.
- 객체는 타입으로 구분가능하지만 테이블에서는 타입이 없기 때문에 <b>부모테이블에서 자식테이블의 타입을 구분하기 위해 DTYPE 
  구분컬럼을 사용</b>한다.
- 조회할 때 조인을 자주 사용한다.

<b>장점</b>
- 테이블 정규화
- 외래 키 참조 무결성 제약조건을 활용할 수 있음
- 저장공간을 효율적으로 사용

<b>단점</b>
- 조회할 때 조인이 많아 성능 저하됨
- 조회 쿼리가 복잡함(조인이 많다)
- 테이블을 등록할 때 INSERT SQL을 두 번 실행해야함
</pre>
```java
// 부모 클래스

@Entity
@Setter @Getter
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name; //상품 명
    private int price; //상품 가격
}
```
```java
// 자식 클래스

@Entity
@Setter @Getter
@DiscriminatorValue("M")
public class Movie extends Item{
    private String director;
    private String actor;
}
```
### `조인 전략에 데이터 저장`
```java
Movie movie = new Movie();
movie.setDirector("감독");
movie.setActor("배우");
movie.setName("바람과함께사라지다");
movie.setPrice(30000);

em.persist(movie);
```
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/JoinDB.PNG"/>
- @DiscriminatorColumn : 부모 클래스에 구분컬럼(DTYPE) 생성
- @DiscriminatorValue("M") : Movie의 구분컬럼(DTYPE) 값을 M으로 지정
</pre>
## 단일 테이블 전략
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/SigleTableStrategy.PNG"/>
- 테이블을 하나만 사용한다.
- 조인을 사용하지 않아서 조회 성능이 빠르다.
- 구분 컬럼(DTYPE)이 자동으로 생성된다.

<b>장점</b>
- 조인이 필요 없으므로 일반적으로 조회 성능이 빠름
- 조회 쿼리가 단순함

<b>단점</b>
- 자식 엔티티가 매핑한 컬럼은 모두 null 허용
- 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다.
  (상황에 따라서 조회 성능이 오히려 느려질 수 있다)
</pre>
```java
// 부모 클래스

@Entity
@Setter @Getter
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name; //상품 명
    private int price; //상품 가격
}
```
### `싱글 테이블 전략에 데이터 저장`
```java
Movie movie = new Movie();
movie.setDirector("감독");
movie.setActor("배우");
movie.setName("바람과함께사라지다");
movie.setPrice(30000);

em.persist(movie);
```
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/SingleTableDB.PNG"/>
- Movie 이외에 하위 테이블 Book과 Album 테이블의 컬럼은 null로 지정된다.
  (그러므로 부모 테이블 Item 이외에 모든 하위 테이블의 컬럼은 null을 허용해야 한다)
</pre>
## 구현 클래스마다 테이블 전략
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/allTableStrategy.PNG"/>
- 자식 테이블이 부모 테이블의 모든것을 다 가지고 있는 형태이다.
- 자식 엔티티마다 테이블을 다 만들어 준다.
- <b>상속 개념이 없어지기 때문에 데이터베이스 설계자나 ORM 전문가 모두 선호하지 않는다(비추천 방법)</b>

<b>장점</b>
- 서브 타입을 구분해서 처리할 때 효과적
- not null 제약조건을 사용할 수 있다(본인것만 있기에)
- 구분 컬럼을 사용하지 않아도 된다.

<b>단점</b>
- 여러 자식 테이블을 함께 조회할 때 성능이 느리다(SQL에 UNION을 사용)
- 자식 테이블을 통합해서 쿼리하기 힘듦
</pre>
```java
// 부모 클래스

@Entity
@Setter @Getter
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
@DiscriminatorColumn
public abstract class Item {

    @Id @GeneratedValue
    private Long id;

    private String name; //상품 명
    private int price; //상품 가격
}
```
### `구현 클래스마다 테이블 전략에 데이터 저장`
```java
Movie movie = new Movie();
movie.setDirector("감독");
movie.setActor("배우");
movie.setName("바람과함께사라지다");
movie.setPrice(30000);

em.persist(movie);
```
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/allTableDB.PNG"/>
- 부모클래스(Item)을 따로 사용할게 아니라면 추상 클래스로 만든다.
- 추상 클래스로 만들어진 부모클래스(Item)는 따로 테이블로 생성되지 않는다.
- 자식클래스(movie)가 부모클래스(Item)의 정보를 다 가지고 있다.
</pre>
## @MappedSuperclass
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/MappedSuperclass.PNG"/>
<b>@MappedSuperclass</b>
- 공통 매핑 정보가 필요할 때 사용(id,name)
- 테이블과 관계 없고, 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모으는 역할

<b>특징</b>
- 상속관계 매핑이 아니다.
- @MappedSuperclass가 선언되어 있는 클래스는 엔티티가 아니다. 당연히 테이블과 매핑도 안된다.
- 단순히 부모 클래스를 상속 받는 <b>자식 클래스에 매핑 정보만 제공</b>한다.
- 조회, 검색 불가(<b>em.find(BaseEntity) 불가</b>)
- 직접 생성해서 사용할 일이 없으므로 <b>추상 클래스 권장</b>

<b>참고</b>
- <b>@Entity 클래스는 엔티티나 @MappedSupercalss로 지정한 클래스만 상속 가능</b>

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/MappedSuperclassDB.PNG"/>
- 상속된 BaseEntity의 컬럼이 같이 나온다.
</pre>
```java
// 엔티티에서 공통으로 들어가야 하는 정보들을 모든 클래스

@MappedSuperclass
@Setter @Getter
public abstract class BaseEntity {
    private String createdBy; // 등록자
    private LocalDateTime createdDate; // 등록일
    private String lastModifiedBy; // 마지막 수정자
    private LocalDateTime lastModifiedDate; // 마지막 수정일
}
```
```java
// BaseEntity 상속

@Entity
@Setter @Getter
public class Member extends BaseEntity{
}
```