# Cascade, orphanRemovel
## 영속성 전이 : CASCADE
<pre>
<b>Cascade</b>
- 특정 엔티티를 영속 성태로 만들 때 <b>연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용</b>
  EX) 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장한다.
- @OneToMany나 @ManyToOne에 옵션으로 줄 수 있다.
- javax.persistence.CascadeType

<b>JPA Cascade Type</b>
- <b>ALL</b> : 모두 적용(모든 작업을 전파)
- <b>PERSIST</b> : 영속(연쇄적 저장)
- <b>REMOVE</b> : 삭제(연쇄적 삭제)
- MERGE : 병합(연쇄적 업데이트)
- REFRESH : REFRESH(연쇄적 새로고침)
- DETACH : DETACH(연쇄적 영속성 제거)

<b>주의</b>
- <b>꼭 부모 엔티티가 자식 엔티티의 단일 소유자 일 때만 영속성 전이를 사용해야 한다.</b>
  - 만약 다른 엔티티가 자식 엔티티를 가지고 있을 경우 사용하면 안된다.
  - 자식 엔티티를 따로 영속화 하지 않기 때문에 다른 엔티티에서 자식 엔티티 필드가 사라지는 현상이 발생할 수 있음.
</pre>
```java
// 부모 엔티티

@Entity
@Setter @Getter
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
    
    //CASCADE(영속성 전이), orphanRemoval(고아 객체)
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Child> childList = new ArrayList<>();

    //양방향 연관관계 편의 메서드
    public void addChild(Child child){
        childList.add(child);
        child.setParent(this);
    }
}
```
```java
// 자식 엔티티

@Entity
@Setter @Getter
public class Child {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent; //FK

}
```
### `CASCADE 사용`
<pre>
<b>@OneToMany(mappedBy="parent", cascade=CascadeType.ALL)</b>
- 연쇄적으로 하위 객체에도 상위 객체의 작업을 같이 수행한다.
- Cascade 속성으로 인해서 parent만 영속화 하여도 영속성 전이가 되어 child1 과 child2 는 자동 저장된다.
</pre>
```java
Child child1 = new Child();
Child child2 = new Child();

Parent parent = new Parent();
parent.addChild(child1);
parent.addChild(child2);

// 영속성 전이
em.persist(parent);
//em.persist(child1);
//em.persist(child2);
```
## 고아 객체 : orphanRemoval = true
<pre>
고아 객체 
- 부모 엔티티와 연관관계가 끊어진 자식 엔티티는 자동으로 삭제 (DB에서 지워진다)
- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능
- <b>orphanRemoval = true</b>

<b>주의</b>
- <b>꼭 참조하는 곳이 하나일 때만 사용</b>
  (Child가 Parent의 개인 소유 엔티티일 때만 사용)
- @OneToOne, @OneToMany만 가능

<b>참고</b>
- 개념적으로 부모를 제거하면 자식은 고아가 된다.
- 따라서 고아 객체 제거 기능을 활성화 하면, 부모를 제거할 때 자식도 함께 제거가 된다.
- 이것은 Cascade.Type.REMOVE처럼 동작한다.
</pre>
### `orphanRemoval 사용`
```java
Child child1 = new Child();
Child child2 = new Child();

Parent parent = new Parent();
parent.addChild(child1);
parent.addChild(child2);

//Cascade 미적용
em.persist(parent);
em.persist(child1);
em.persist(child2);

em.flush();
em.clear();

// 상황 1 : 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제
// 컬렉션을 빠져나간 자식 엔티티는 자동 삭제된다.
Parent parentFind = em.find(Parent.class, id);
parentFind.getChildList().remove(0);

// 상황 2 : 부모 객체가 삭제되면서 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제
// 부모 엔티티 삭제 시 연쇄적으로 모든 Child가 삭제된다.
em.remove(parentFind);
```
## 영속성 전이(Cascade) + 고아 객체(orphanRemoval), 생명주기
<pre>
- <b>CascadeType.ALL + orphanRemovel=true</b>
- 스스로 생명주기를 관리하는 엔티티(부모)는 직접 em.persist()로 영속화, em.remove()로 제거
- 두 옵션을 모두 활성화 하면 부모 엔티티를 통해서 자식의 생명주기를 관리할 수 있다.
- 도메인 주도 설계(DDD)의 Aggregate Root 개념을 구현할 때 유용
</pre>
