# Entity Mapping
## Object to Table Mapping
<pre>
<b>@Entity</b> : @Entity가 붙은 클래스는 JPA가 관리, 엔티티라 한다.

<b>주의</b> 
- <b>기본 생성자 필수</b>(파라미터가 없는 public 또는 protected 생성자)
- final 클래스, enum, interface, inner 클래스 사용X
- 저장할 필드에 final 사용X


<b>@Table</b> : @Table은 엔티티와 매핑할 테이블 지정
<table>
<th>속성</th><th>기능</th><th>기본값</th>
<tr>
    <td>name</td><td>매핑할 테이블 이름</td><td>엔티티 이름을 사용</td>
</tr>
<tr>
    <td>catalog</td><td>데이터베이스 catalog 매핑</td><td></td>
</tr>
<tr>
    <td>schema</td><td>데이터베이스 SCHEMA 매핑</td><td></td>
</tr>
<tr>
    <td>uniqueConstraints(DDL)</td><td>DDL 생성 시에 유니크 제약 조건 생성</td><td></td>
</tr>
</table>
</pre>
```java
@Entity
@Table(name = "User")
public class Member { 

}
```
## Field and Column Mapping
<pre>
<b>@Column</b> : 컬럼 매핑
- 개발자들이 엔티티를 보고 개발할 수 있도록 컬럼의 속성을 통해 제약 조건을 지정해주는 것이 좋다!
<table>
<th>속성</th><th>설명</th><th>기본값</th>
<tr>
    <td>name</td><td>필드와 매핑할 테이블 컬럼 이름</td><td>객체의 필드 이름</td>
</tr>
<tr>
    <td>insertable,</br>updatable</td><td>등록, 변경 가능 여부</td><td>TRUE(FALSE : 불가능)</td>
</tr>
<tr>
    <td>nullable(DDL)</td><td>null 값의 허용 여부를 설정한다. </br>false로 설정하면 DDL 생성 시에 not null 제약조건이 붙는다.</td><td></td>
</tr>
<tr>
    <td>unique(DDL)</td><td>@Table의 uniqueConstraints와 같지만</br>한 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다.</br>(제약 조건의 이름을 무작위로 설정함으로 잘 사용 안함)</td><td></td>
</tr>
<tr>
    <td>length(DDL)</td><td>문자 길이 제약조건, String 타입에만 사용한다.</td><td>varchar(255)</td>
</tr>
<tr>
    <td>precision,<br>scale(DDL)</td><td>BigDecimal 타입에서 사용한다(BigInteger도 사용할 수 있다)<br>precision은 소수점을 포함한 전체 자릿수를,</br>scale은 소수의 자릿수다.</br><b>아주 큰 숫자나 정밀한 소수를 다루어야 할 때만 사용한다.</b></br>(double, float 타입에는 적용되지 않는다.)</td><td>precision=19,</br>scale=2</td>
</tr>
</table>


<b>@Temporal</b> : 날짜 타입 매핑
- 날짜 타입(java.util.Date, java.util.Calendar)을 매핑할 때 사용
- 참고 : <b>LocalDate, LocalTime, LocalDateTime을 사용할 때는 어노테이션 생략 가능(최신 하이버네이트 지원)</b>
<table>
<th>속성</th><th>설명</th><th>기본값</th>
<tr>
    <td>value</td><td><b>TemporalType.DATE</b> : 날짜, 데이터베이스 date 타입과 매핑</br>(예: 2013–10–11)</br><b>TemporalType.TIME</b> : 시간, 데이터베이스 time 타입과 매핑</br>(예: 11:11:11)</br><b>TemporalType.TIMESTAMP</b> : 날짜와 시간, 데이터베이스 timestamp 타입과 매핑</br>(예: 2013–10–11 11:11:11)</td><td></td>
</tr>
</table>


<b>@Enumerated</b> : enum 타입 매핑
- 무조건 <b>EnumType.STRING</b>을 사용해야 한다.
<table>
<th>속성</th><th>설명</th><th>기본값</th>
<tr>
    <td>value</td><td>EnumType.ORDINAL: enum 순서를 데이터베이스에 저장</br><b>EnumType.STRING: enum 이름을 데이터베이스에 저장</b></td><td>EnumType.ORDINAL</td>
</tr>
</table>

<b>@Lob</b> : BLOB, CLOB 매핑
- <b>데이터베이스 BLOB, CLOB 타입과 매핑</b>
- @Lob에는 지정할 수 있는 속성이 없다.
- 매핑하는 필드 타입이 문자면 CLOB 매핑, 나머지는 BLOB 매핑
    * CLOB : String, char[], java,sql.CLOB
    * BLOB : byte[], java.sql.BLOB


<b>@Transient</b> : 특정 필드를 컬럼에 매핑하지 않음(매핑 무시)
- 필드 매핑X
- 데이터베이스에 저장X, 조회X
- <b>주로 메모리상에만 임시로 어떤 값을 보관하고 싶을 때 사용</b>
</pre>
```java
import javax.persistence.*; 
import java.time.LocalDate; 
import java.time.LocalTime;
import java.time.LocalDateTime; 
import java.util.Date; 

@Entity 
public class Member { 

    @Id 
    private Long id; 

    @Column(name = "name") 
    private String username; 

    private Integer age; 

    @Enumerated(EnumType.STRING) 
    private RoleType roleType; 

    @Temporal(TemporalType.TIMESTAMP) 
    private Date createdDate; 

    @Temporal(TemporalType.TIMESTAMP) 
    private Date lastModifiedDate; 

    // 년,월,일 정보
    private LocalDate testLocalDate;
    // 시,분,(초),(나노초) 정보
    private LocalTime testLocalDate;
    // 년,월,일,시,분,(초),(나노초) 정보
    private LocalDateTime testLocalDateTime;

    @Lob 
    private String description; 

    @Transient
    private Integer temp;

    //Getter, Setter… 
} 
```
## Primary Key Mapping
<pre>
<b>@Id</b> : 기본 키 매핑
- @Id만 사용할 경우 직접 기본키를 할당

<b>@GeneratedValue</b> : 기본 키 자동 생성
<table>
<th>속성</th><th>설명</th><th>데이터베이스</th>
<tr>
    <td>IDENTITY</td><td>기본키 생성을 데이터베이스에 위임</br>(Auto_Increment)</td><td>MYSQL</td>
</tr>
<tr>
    <td>SEQUENCE</td><td>데이터베이스 시퀀스 오브젝트 사용</br>(@SequenceGenerator 필요)</td><td>ORACLE</td>
</tr>
<tr>
    <td>TABLE</td><td>키 생성용 테이블 사용</br>(@TableGenerator 필요)</td><td>모든 DB에서 사용</td>
</tr>
<tr>
    <td>AUTO</td><td>방언에 따라 자동 지정</td><td>기본값</td>
</tr>
</table>
<b>권장하는 식별자 전략</b>
- <b>기본 키 제약 조건</b> : null 아님, 유일(unique), 불변해야 한다.
- 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 그 대신 <b>대리키(대체키)를 사용</b>하자.
- 자연키(Natural Key) : 비즈니스적으로 의미가 있는 키(주민번호, 전화번호 등)
- 대리키(대체키) : 비즈니스적으로 상관없는 키(Generate Value, 랜덤 값, 유휴 값 등)

<b>권장 : Long형 + 대체키 + 키 생성전략 사용</b>
- Long Type(큰 숫자)
- 대체키 사용: 랜덤 값, 유휴 ID 등 비즈니스와 관계없는 값 사용
- AUTO_INCREMENT 또는 Sequnce Object 사용
</pre>
### `IDENTITY`
```java
// id 값을 null로 하면 DB가 알아서 AUTO_INCREMENT 해준다.
// IDENTITY 전략에서만 예외적으로 entityManager.persist()가 호출되는 시점에 바로 DB에 INSERT 쿼리를 날린다.
// 그러므로 entityManager.persist() 이 후에 AUTO_INCREMENT 로 생성된 ID 는 바로 조회하여 사용할 수 있다.
public class Member{
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```
```console
-- H2
create table Member (
  id varchar(255) generated by default as identity,
  ...
)

-- MySQL
create table Member (
  id varchar(255) auto_increment,
  ...
)
```
### `SEQUENCE`
<pre>
<b>@SequenceGenerator</b>
- 테이블 별로 시퀀스를 따로 이름을 주어서 관리하고 싶을 때 사용
- @GeneratedValue 의 generator 속성 값으로 부여하여 사용

<b>Sequence를 매번 DB에서 가지고오는 과정에서 네트워크를 타기 때문에 성능 상의 저하를 가져올 수 있다.</b>
- 이를 해결하기 위한 성능 최적화 방법으로 <b>allocationSize 속성값 (기본값: 50) 이용</b>
- allocationSize 사용 시 next call을 할 때 미리 DB에 50개를 한 번에 올려 놓고(DB는 sequence가 51로 셋팅) 메모리 상에서 1개씩 사용
- entityManager.persist() 이 후에 Sequence 로 생성된 ID 는 바로 조회하여 사용할 수 있다.
<table>
<th>속성</th><th>설명</th><th>기본값</th>
<tr>
    <td>name</td><td>식별자 생성기 이름</td><td>필수</td>
</tr>
<tr>
    <td>sequenceName</td><td>데이터베이스에 등록되어 있는 시퀀스 이름</td><td>hibernate_sequence</br>(기본값으로 하이버네이트 시퀀스를 사용)</td>
</tr>
<tr>
    <td><b>initialValue</b></td><td>DDL 생성 시에만 사용됨, 시퀀스 DDL을 생성할</br>때 처음 시작하는 수를 지정한다.</td><td>1</td>
</tr>
<tr>
    <td><b>allocationSize</b></td><td>시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용)</br><b>데이터베이스 시퀀스 값이 하나씩 증가하도록 설정되어 있으면</br>이 값을 반드시 1로 설정해야 한다.</b></br>(보통 50 ~ 100이 적당)</td><td>50</td>
</tr>
<tr>
    <td>catalog,</br>schema</td><td>데이터베이스 catalog, schema 이름</td><td></td>
</tr>
</table>
</pre>
```java
@Entity
@SequenceGenerator(
  name = "MEMBER_SEQ_GENERATOR", 
  sequenceName = "MEMBER_SEQ", // 매핑할 데이터베이스 시퀀스 이름 
  initialValue = 1, 
  allocationSize = 50)
public class Member {
  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE,
                  generator = "MEMBER_SEQ_GENERATOR")
  private Long id; 
}
```
```console
-- 1부터 시작해서 1씩 증가 
create sequence MEMBER_SEQ start with 1 increment by 50 (Sequence Object 생성)
call next value for hibernate_sequence
```
### `TABLE`
<pre>
<b>@TableGenerator</b>
- <b>키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략</b>
- @GeneratedValue 의 generator 속성 값으로 부여하여 사용
- 장점 : 모든 데이터베이스에 적용 가능
- 단점 : 최적화 되어있지 않은 테이블을 직접 사용하기 때문에 성능상의 이슈가 있음
<table>
<th>속성</th><th>설명</th><th>기본값</th>
<tr>
    <td>name</td><td>식별자 생성기 이름</td><td>필수</td>
</tr>
<tr>
    <td>table</td><td>키 생성 테이블 명</td><td>hibernate_sequences</br>(기본값으로 하이버네이트 시퀀스를 사용)</td>
</tr>
<tr>
    <td>pkColumnName</td><td>시퀀스 컬럼명</td><td>sequence_name</td>
</tr>
<tr>
    <td>valueColmnNa</td><td>시퀀스 값 컬럼명</td><td>next_val</td>
</tr>
<tr>
    <td>pkColumnValue<td>키로 사용할 값 이름</td><td>엔티티 이름</td>
</tr>
<tr>
    <td><b>initialValue</b></td><td>초기 값, 마지막으로 생성된 값이 기준</td><td>0</td>
</tr>
<tr>
    <td><b>allocationSize</b></td><td>시퀀스 한 번 호출에 증가하는 수</br>(성능 최적화에 사용)</td><td>50</td>
</tr>
<tr>
    <td>catalog,</br>schema</td><td>데이터베이스 catalog, schema 이름</td><td></td>
</tr>
<tr>
    <td>uniqueConstraints(DDL)<td>유니크 제약 조건을 지정</td><td></td>
</tr>
</table>
</pre>
```java
@Entity
@TableGenerator( 
 name = "MEMBER_SEQ_GENERATOR", 
 table = "MY_SEQUENCES", // 키 생성 테이블 명
 pkColumnValue = "MEMBER_SEQ", allocationSize = 1)
public class Member {
  @Id
  @GeneratedValue(strategy = GenerationType.TABLE,
                  generator = "MEMBER_SEQ_GENERATOR")
  private Long id; 
}
```
```console
create table MY_SEQUENCES (
  sequence_name varchar(255) not null,
  next_val bigint,
  primaty key ( sequence_name )
)
```
### `AUTO`
<pre>
- <b>@GeneratedValue(strategy = GenerationType.AUTO)</b>
- 기본 설정 값
- 설정된 방언에 따라 위의 세 가지 전략을 자동으로 지정한다.
</pre>
## Database Schema Auto-generated
<pre>
<b>데이터베이스 스키마 자동 생성</b>
- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 -> 객체 중심
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성
- 이렇게 <b>생성된 DDL은 개발 장비에서만 사용</b>
- 생성된 DDL은 운영서버에서는 사용하지 않으나, 적절히 다듬은 후 사용가능

<b>hibernate.hbm2ddl.auto</b>
<table>
<th>옵션</th><th>설명</th>
<tr>
    <td>create</td><td>기존테이블 삭제 후 다시 생성(DROP + CREATE)</td>
</tr>
<tr>
    <td>create-drop</td><td>create와 같으나 종료시점에 테이블 DROP</td>
</tr>
<tr>
    <td>update</td><td>변경분만 반영(운영DB에는 사용하면 안됨)</td>
</tr>
<tr>
    <td>validate</td><td>엔티티와 테이블이 정상 매핑되었는지만 확인</td>
</tr>
<tr>
    <td>none<td>사용하지 않음(주석과 같음)</td>
</tr>
</table>

<b>운영 장비에는 절대 create, create-drop, update 사용하면 안된다.</b>
- 개발 초기 단계는 create 또는 update
- 테스트 서버는 update 또는 validate
- 스테이징과 운영 서버는 validate 또는 none
</pre>
```xml
<!-- src/main/resources/META-INF/persistence.xml -->
<properties>
    <property name="hibernate.hbm2ddl.auto" value="create"/>
</properties>
```
