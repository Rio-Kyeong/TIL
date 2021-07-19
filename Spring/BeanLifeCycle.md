# Spring Bean LifeCycle

## 스프링 컨테이너 생명주기(Life Cycle)
(사진)

## 빈(Bean)객체 생명주기(Life Cycle)
스프링 컨테이너에 의해 객체가 생성되고 초기화와 소멸의 과정을 거친다.<br/>
(사진)

## InitializingBean, DisposableBean 
### InitializingBean Interface
<pre>
<b>InitializingBean Interface</b>는 BeanFactory에 의해 해당 Bean 객체의 프로퍼티가 모두 설정된 후 단 한 번만 반응(react), 즉 <b>afterPropertiesSet()</b> 콜백 메서드를 호출한다.

객체에 특별한 프로퍼티를 추가적으로 설정하거나 필수적으로 요구되는 사항들이 모두 충족되었는지 검사하는 등 목적으로 사용된다.
</pre>
### DisposableBean Interface
<pre>
<b>DisposableBean Interface</b>는 BeanFactory에 의해 객체 소멸 시  <b>destroy()</b> 콜백 메서드를 호출한다. 어떤 자원을 해제할 필요가 있는 Bean 객체들이 구현하는 인터페이스다.

스프링 컨테이너는 종료 시 애플리케이션 생명주기에 의해 모든 싱글턴 Bean들을 폐기(dispose)할 때 사용한다.
</pre>
### 1. afterPropertiesSet() 메소드와 destroy() 메소드를 오버라이드
`하지만 최근에는 다음과 같은 이유로 이러한 방법을 사용하지 않는다고 한다.`
* 자바 표준 인터페이스가 아닌 스프링 전용 인터페이스로써 스프링에 종속적이다.
* 초기화, 소멸 메소드의 이름을 변경할 수 없다.
* 캐시 등 직접 제어를 해주어야 하는 외부 라이브러리에 적용할 수 없다.

```java
public class BookRegisterService implements InitializingBean, DisposableBean{

	@Autowired
	private BookDao bookDao; //BookDao 객체 의존성 주입
	
	public BookRegisterService() { }
	
	public void register(Book book) {
		bookDao.insert(book);
	}
	
	/**
	 * InitializingBean Interface
	 */
	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("bean 객체 생성되는 시점에 하는 작업");
	}
	
	/**
	 * DisposableBean Interface
	 */
	@Override
	public void destroy() throws Exception {
		System.out.println("bean 객체 소멸되는 시점에 하는 작업");	
	}
}
```
### 2. @PostConstruct, @PreDestroy Annotation 
Spring이 아닌 다른 컨테이너에서도 동작 가능

```java
public class BookRegisterService {

	@Autowired
	private BookDao bookDao; //BookDao 객체 의존성 주입
	
	public BookRegisterService() { }
	
	public void register(Book book) {
		bookDao.insert(book);
	}
	
	@PostConstruct //초기화
	public void init() {
		System.out.println("bean 객체 생성되는 시점에 하는 작업");
	}
	
	@PreDestroy //소멸
	public void destory() {
		System.out.println("bean 객체 소멸되는 시점에 하는 작업");
	}
}
```
어노테이션 사용을 위한 종속성에 `javax.annotation-api` JAR을 추가
```xml
<!-- https://mvnrepository.com/artifact/javax.annotation/javax.annotation-api -->
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```

### 3. init-method, destroy-method 속성
코드에 접근할 수 없는 외부 라이브러리에도 초기화, 종료 메소드를 적용가능

bean테그에서 `init-method`와 `destroy-method` 선언
```xml
<context:annotation-config /> <!-- 의존성 자동주입을 위해 정의-->
<bean id="memberDao" class="com.brms.member.dao.MemberDao"/>
	
	<bean 
		id="memberRegisterService" 
		class="com.brms.member.service.MemberRegisterService"
		init-method="init" 
		destroy-method="destroy"
	/>
    
```
`init-method와 destroy-method 속성값과 메서드 명은 동일하게 해야 한다.`
```java
public class MemberRegisterService {

	@Autowired
	private MemberDao memberDao;
	
	public MemberRegisterService() { }
	
	public void register(Member member) {
		memberDao.insert(member);
	}
	
	public void init() {
		System.out.println("initMethod 호출");
	}
	
	public void destroy() {
		System.out.println("destroyMethod 호출");
	}
}
```