# JPQL 타입 표현과 기타식, 조건식
## JPQL 타입 표현
<pre>
<b>문자 표현</b> : 'HELLO', 'She''s' (' : 작은 따옴표 두개로 표현)

<b>숫자 표현</b> : 10L(Long), 10D(Double) 10F(Float)

<b>Boolean 표현</b> : TRUE, FALSE

<b>ENUM 표현</b> : jpabook.MemberType.Admin (패키지명 포함)

<b>엔티티 표현</b> : TYPE(m) = Member (상속 관계에서 사용)
</pre>
```java
// Member(회원명, 나이, 팀, 권한)
em.persist(new Member("A", 10, MemberType.ADMIN));
em.persist(new Member("B", 20, MemberType.USER));

//Book(책이름, 가격, 수량, 저자, ISBN)
em.persist(Book.createBook("책이름", 20000, 10, "저자", "isbn"));

// 문자, 숫자, Boolean 표현
List<Object[]> list = em.createQuery("select 'She''s', 10L, TRUE from Member m")
        .getResultList();

for (Object[] objects : list) {
    System.out.println("문자 = "+ objects[0]);
    System.out.println("숫자 = "+ objects[1]);
    System.out.println("Boolean = "+ objects[2]);
}

// ENUM 표현(Member Type이 ADMIN인 회원 조회)
List<Member> enums = em.createQuery("select m from Member m where m.type = jpql.MemberType.ADMIN", Member.class)
        .getResultList();

System.out.println("================ Enum ================");
enums.forEach(System.out::println);

//엔티티 표현(Item의 자식 클래스인 Book 클래스 조회(상속관계))
List<Item> type = em.createQuery("select i from Item i where type(i) = Book", Item.class)
        .getResultList();

System.out.println("================ Entity Tpye ================");
type.forEach(System.out::println);
```
## JPQL 기타식
<pre>
<b>SQL과 문법이 같은 식</b>
- EXISTS, IN
- AND, OR, NOT
- =, >, >=, <=, <>
- BETWEEN A AND B, LIKE, IS NULL
</pre>
## JPQL 조건식
<pre>
<b>조건식</b>
- 기본적인 조건식 대부분 JPQL에서 사용이 가능하다.
</pre>
### `기본, 단순 CASE`
```java
em.persist(new Member("A", 10));
em.persist(new Member("D", 15));
em.persist(new Member("B", 20));
em.persist(new Member("C", 30));

// 기본 CASE 식
List<String> normalCase = em.createQuery
        ("select case" +
                "       when m.age > 10 then '10대'" +
                "       when m.age > 20 then '20대'" +
                "       else '30대 이상'" +
                " end" +
                " from Member m", String.class)
        .getResultList();

normalCase.forEach(System.out::println);

// 단순 CASE 식
List<String> simpleCase = em.createQuery
        ("select case m.name" +
                "       when 'A' then '에이'" +
                "       when 'B' then '비'" +
                "       else '나머지'" +
                " end" +
                " from Member m", String.class)
        .getResultList();

simpleCase.forEach(System.out::println);
```
```console
Hibernate: 
        select
            case 
                when member0_.age>10 then '10대' 
                when member0_.age>20 then '20대' 
                else '30대 이상' 
            end as col_0_0_ 
        from
            Member member0_
30대 이상
10대
10대
10대

Hibernate: 
        select
            case member0_.name 
                when 'A' then '에이' 
                when 'B' then '비' 
                else '나머지' 
            end as col_0_0_ 
        from
            Member member0_
에이
나머지
비
나머지
```
### `COALESCE`
<pre>
<b>COALESCE</b>
- 하나씩 조회해서 null이 아니면 반환
- null일 경우 대체 값을 지정할 수도 있다.
</pre>
```java
em.persist(new Member("홍길동", 10));
em.persist(new Member(null, 15));
em.persist(new Member("B", 20));
em.persist(new Member(null, 30));

// 사용자 이름이 없으면(null) 이름 없는 회원을 반환
List<String> coalesce = em.createQuery
    ("select coalesce(m.name, '이름 없는 회원') from Member m", String.class)
    .getResultList();

coalesce.forEach(System.out::println);
```
```console
Hibernate: 
        select
            coalesce(member0_.name,
            '이름 없는 회원') as col_0_0_ 
        from
            Member member0_
홍길동
이름 없는 회원
이순신
이름 없는 회원
```
### `NULLIF`
<pre>
<b>NULLIF</b>
- 두 값이 같으면 null 반환, 다르면 첫번째 값 반환
</pre>
```java
em.persist(new Member("홍길동", 10));
em.persist(new Member("이순신", 20));

// 사용자 이름이 '홍길동'이면 null을 반환하고 나머지는 본인의 이름을 반환
List<String> nullif = em.createQuery
        ("select nullif(m.name, '홍길동') from Member m", String.class)
        .getResultList();

nullif.forEach(System.out::println);
```
```console
Hibernate: 
        select
            nullif(member0_.name,
            '홍길동') as col_0_0_ 
        from
            Member member0_

null
이순신
```