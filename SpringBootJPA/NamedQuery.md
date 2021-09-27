# Named Query
<pre>
<b>Named Query</b>
- <b>미리 정의해서 이름을 부여해두고 사용하는 JPQL</b>
- 정적 쿼리만 사용가능
- 어노테이션 또는 XML에 정의하여 사용
- 애플리케이션 로딩 시점에 초기화 후 재사용
- <b>애플리케이션 로딩 시점에 쿼리를 검증</b>

<b>Named Query 환경에 따른 설정</b>
- XML이 항상 우선권을 가진다.
- 애플리케이션 운영 환경에 따라 다른 XML을 배포할 수 있다.
</pre>
## Named Query - Annotation
```java
@Entity
@NamedQuery(
    name = "Member.findByUsername",
    query = "select m from Member m where m.username = :username")
public class Member {
    ...
}
```
```java
//use NamedQuery
List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
        .setParameter("username", "회원1") //setParameter("바인딩 변수", 찾을 값)
        .getResultList();
```
## Named Query - xml
```xml
<!-- META-INF/persistence.xml -->
<persistence-unit name="jpabook">
 <mapping-file>META-INF/ormMember.xml</mapping-file>
```
```xml
<!-- META-INF/ormMember.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">
    <named-query name="Member.findByUsername">
        <query><![CDATA[
                select m
                from Member m
                where m.username = :username
                ]]>
        </query>
    </named-query>
    <named-query name="Member.count">
        <query>select count(m) from Member m</query>
    </named-query>
</entity-mappings>
```
## 정리
<pre>
- Named Query란 미리 정의해서 이름을 부여해두고 사용하는 JPQL
- <b>Entity 가독성이 떨어지기 때문에 권장하지 않음</b>
- Spring Data JPA에서는 Repository interface에 <a href="https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.at-query">@Query</a>으로 사용가능
</pre>