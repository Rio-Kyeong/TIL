# Dependency Injection - annotation
## 의존객체 자동 주입이란?
```
스프링 설정 파일에서 의존 객체를 주입할 때 <constructor-org> 또는 <property> 태그로 의존 대상 객체를 명시하지 않아도 스프링 컨테이너
가 자동으로 필요한 의존 대상 객체를 찾아서 의존 대상 객체가 필요한 객체에 주입해 주는 기능이다. 
구현 방법은 @Autowired와 @Resource 어노테이션을 이용해서 쉽게 구현할 수 있다.
```

## javax.annotation 종속
```xml
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```
Annotation은 Spring의 구성요소가 아니기 때문에 `pom.xml`에 `javax.annotations` 종속성을 추가해야한다.<br/>
## @Autowired
<pre >
<b>@Autowired</b> 는 주입하려고 하는 객체의 <b>타입</b>이 일치하는 객체를 자동으로 주입한다.
</pre>
* **@Autowired**는 `필드(Property)`, `생성자(Constructor)`, `Setter Method`에 붙여서 사용할 수 있다.
* 단, @Autowired를 필드와 Setter에 붙여서 사용할 경우 반드시 기본 생성자를 정의해야 한다.
* `<context:annotation-config/>` 구문을 꼭 xml 설정파일에 추가해야한다.
* @Autowired(**required = false**)를 통해서 의존객체 없는 등의 이유로 주입받지 못하더라도 빈을 생성하도록 할 수 있다.

* Spring에서 지원하는 어노테이션이다.


### 1. 필드(Property) 주입
* 기본 생성자 정의
```java
public class WordRegisterService{

	@Autowired
	private WordDao wordDao;
	
	public WordRegisterService() { 

    }
}
```
### 2. 생성자(Constructor) 주입
```java
public class WordRegisterService {

	private WordDao wordDao;
	
	@Autowired
	public WordRegisterService(WordDao wordDao) {
		this.wordDao = wordDao;
	}
}
```
### 3. Setter Method 주입
* 기본 생성자 정의
```java
public class WordRegisterService{

	private WordDao wordDao;
	
	public WordRegisterService() { 

    }

	@Autowired
	public void setWordDao(WordDao wordDao) {
		this.wordDao = wordDao;
	}
}
```
## @Qualifier
```xml
	<context:annotation-config/> 
	
	<bean id="wordDao1" class="com.word.dao.WordDao">
		<qualifier value="usedDao"/>
	</bean>
	<bean id="wordDao2" class="com.word.dao.WordDao" />
	<bean id="wordDao3" class="com.word.dao.WordDao" />
	
	<bean id="registerService" class="com.word.service.WordRegisterService"/>
	<bean id="searchService" class="com.word.service.WordSearchService"/>
```
동일한 타입의 빈 객체가 여러개 정의되어 있을 경우 에러가 발생한다.<br/>
우선적으로 사용할 빈 객체에 `<qualifier>`태그를 설정한다.

```java
public class WordRegisterService {

	@Autowired
	@Qualifier("usedDao")
	private WordDao wordDao;
	
	public WordRegisterService() { 

    }
}
```
`@Autowired`와 함께 `@Qualifier`를 사용해서 XML 설정 파일에서 설정한 `<qualifier>`태그의 `value 값`을 선언해준다.

## @Resource
<pre >
<b>@Resource</b> 는 주입하려고 하는 객체의 <b>이름(id)</b>이 일치하는 객체를 자동으로 주입한다.
</pre>
* **@Resource**는 `필드(Property)`, `Setter Method`에 붙여서 사용할 수 있다.
* 단, 반드시 기본 생성자를 정의해야 한다.
* `생성자(Constructor)`에는 사용할 수 없다.
* `<context:annotation-config/>` 구문을 꼭 xml 설정파일에 추가해야한다.
* Java에서 지원하는 어노테이션이다.


## @Inject
<pre >
<b>@Inject</b>는 <b>@Autowired</b>와 유사하게 주입하려고 하는 객체의 <b>타입</b>이 일치하는 객체를 자동으로 주입한다.
</pre>
* **@Inject**는 `필드(Property)`, `생성자(Constructor)`, `Setter Method`에 붙여서 사용할 수 있다.
* 단, @Autowired를 필드와 Setter에 붙여서 사용할 경우 반드시 기본 생성자를 정의해야 한다.
* `required속성`을 지원하지 않는다.
* Java에서 지원하는 어노테이션이다.

## @Named
* `@Autowired`의 `@Qualifier`와 같이 사용할 수 있는 것이 `@Inject`에서는 `@Named`이다.
* @Qualifier와 달리 `@Named에는 빈 이름(id)를 지정` 한다. 
* EX `@named(value ="bean id")`
```xml
<bean id="wordDao1" class="com.word.dao.WordDao"/> 
<bean id="wordDao2" class="com.word.dao.WordDao"/>
<bean id="wordDao3" class="com.word.dao.WordDao"/>
```
`<Qualifier>`를 사용하지 않는다.
```java
public class WordRegisterService {

	@Inject
	@Named(value="wordDao1")
	private WordDao wordDao;
	
	public WordRegisterService() {
		
	}
}
``` 



