# JPQL 기본 함수
<pre>
<b>JPQL 함수</b>
- DB에서 제공되는 다양한 여러 함수들을 JQPL에서도 사용할 수 있다.
- 직접 사용자 정의함수를 만들어서 사용할 수도 있다.
</pre>
```java
em.persist(new Member("MemberA", 10));

System.out.println("=========================== concat ===========================");
// 문자열 붙이기
em.createQuery("select concat('a', 'b') from Member m", String.class)
        .getResultList().forEach(System.out::println);

System.out.println("=========================== substring ===========================");
// 문자열 자르기
em.createQuery("select substring(m.name, 3, 5) from Member m", String.class)
        .getResultList().forEach(System.out::println);

System.out.println("=========================== TRIM ===========================");
// 공백제거(ltrim(), rtirm() 둘 다 가능)
em.createQuery("select trim(' | ^ _ ^ | ') from Member m", String.class)
        .getResultList().forEach(System.out::println);

System.out.println("=========================== Upper(Lower) ===========================");
// LOWER : 소문자로 변환, UPPER : 대문자로 변환
em.createQuery("select upper(m.name) from Member m", String.class)
        .getResultList().forEach(System.out::println);

System.out.println("=========================== locate ===========================");
// cd가 abcd중 몇번째에 있는지 출력해준다.
em.createQuery("select locate('cd', 'abcd') from Member m", Integer.class)
        .getResultList().forEach(System.out::println);

System.out.println("=========================== length ===========================");
// 문자열 길이
em.createQuery("select length(m.name) from Member m", Integer.class)
        .getResultList().forEach(System.out::println);

System.out.println("=========================== abs ===========================");
// 절대값
em.createQuery("select abs(-12) from Member m", Integer.class)
        .getResultList().forEach(System.out::println);

System.out.println("=========================== mod ===========================");
// 나머지
em.createQuery("select mod(3 , 2) from Member m", Integer.class)
        .getResultList().forEach(System.out::println);

System.out.println("=========================== sqrt ===========================");
// 제곱근
em.createQuery("select sqrt(16) from Member m", Double.class)
        .getResultList().forEach(System.out::println);
```
```console
=========================== concat ===========================
ab

=========================== substring ===========================
mberA

=========================== TRIM ===========================
| ^ _ ^ |

=========================== Upper(Lower) ===========================
MEMBERA

=========================== locate ===========================
3

=========================== length ===========================
7

=========================== abs ===========================
12

=========================== mod ===========================
1

=========================== sqrt ===========================
4.0
```