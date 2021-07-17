# Dependency Injection - xml
## 1. DI

```test
DI(Dependency Injection)란 스프링이 제공하는 의존 관계 주입 기능
객체를 직접 생성(new)하는 게 아니라 외부에서 생성(xml)한 후 주입 시켜주는 방식
의존성 주입을 통해서 모듈 간의 결합도가 낮아지고(약결합) 유연성이 높아진다.
```

## 2. Constructor Injection
1. 의존성 클래스 선언(DAO)
```java
/**
 * 유연성을 높이기 위한 Interface
 * @author Administrator
 */
public interface DAO {
	public Map<Integer,String> selectStudent();
}
```
```java
public class StudentDAOImpl implements DAO{

	@Override
	public Map<Integer, String> selectStudent() {
		Map<Integer, String> map = new HashMap<Integer, String>();
		map.put(1, "홍길동");
		map.put(2, "이순신");
		map.put(3, "임꺽정");
		
		return map;
	}
}
```
2. 의존성 주입받는 생성자 선언(Service)
```java
/**
 * 유연성을 높이기 위한 Interface
 * @author Administrator
 */
public interface Service {
	
	public Map<Integer,String> searchStudent();
}
```
```java
public class StudentServiceImpl implements Service{
	//기능을 유연하게 사용하기 위해서 자식이 아닌 부모를 선언
	private DAO dao;
	
	/**
	 * 의존성 주입받는 생성자
	 * 부모를 정의하므로 모든 자식을 다 받을 수 있다(유연성)
	 * @param dao
	 */
	public StudentServiceImpl(DAO dao) {
		System.out.println("의존성 주입받는 constructor 호출");
		this.dao = dao;
	}

	@Override
	public Map<Integer, String> searchStudent() {
		//부모를 통해 전달된 자식이 구현한 업무가 실행
		Map<Integer, String> map = dao.selectStudent();
		
		//서비스 구현
		map.put(4, "왕건");
		
		return map;
	}
	
}
```
3. 스프링 설정 파일(XML)에 bean설정
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
	 http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
	 http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd
	 http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-4.0.xsd
	 http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
	 http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">
	
	<!-- 의존성 주입 받을 객체 : Singleton으로 자동 생성된다. -->
	<bean id="studentDao" class="ems.member.dao.StudentDAOImpl" ></bean>
	
	<!-- 의존성 주입을 받을 객체생성과 의존성 주입 -->
	<bean id="studentService" class="ems.member.service.StudentServiceImpl">
		<!-- constructor injection -->
		<constructor-arg ref="studentDao" ></constructor-arg>
		<constructor-arg value="high school"></constructor-arg>
	</bean>
</beans>
```
4. 메인 클래스에서 호출(Spring Container)
```java
public class MainSpringApp {

	public static void main(String[] args) {
		
		//1.설정파일을 입력하여 Spring Container를 생성
		// 설정파일 안에 정의된 <bean>들은 사용유무에 상관없이 모두 생성된다.
		ClassPathXmlApplicationContext context = 
				new ClassPathXmlApplicationContext("applicationContext.xml");
		
		//2. 스프링 컨테이너에서 객체를 얻는다(<bean>에 정의된 객체는 모두 얻을 수 있다)
		Service service = context.getBean("studentService",Service.class);
		
		//3. method 호출하여 업무 처리
		System.out.println( service.searchStudent() );
		
		//4. Spring Container의 사용이 종료 되었다면 메모리 누수의 이슈가 있어서 반드시 닫아준다.
		if(context != null) {
			context.close();
		}
	}
}
```

## 3. Setter Method Injection
의존성 주입받는 setter method 선언(Service) - 생성자가 아닌 setter method로 의존성 주입을 받는다.
```java
public class StudentServiceImpl implements Service{
	//기능을 유연하게 사용하기 위해서 자식이 아닌 부모를 선언
	private String school;
	private int grade;
	private DAO dao;
	
	public String getSchool() {
		return school;
	}

	public void setSchool(String school) {
		this.school = school;
	}

	public int getGrade() {
		return grade;
	}

	public void setGrade(int grade) {
		this.grade = grade;
	}
	
	/**
	 * 의존성 주입받는 setter method
	 * @param dao
	 */
	public void setDao(DAO dao) {
		System.out.println("의존성 주입받는 setter method 호출");
		this.dao = dao;
	}
	
	@Override
	public Map<Integer, String> searchStudent() {
		//부모를 통해 전달된 자식이 구현한 업무가 실행
		Map<Integer, String> map = dao.selectStudent();
		
		//서비스 구현
		map.put(4, "왕건");
			
		return map;
	}
	
}
```
스프링 설정 파일(XML)에 bean설정 
```xml
<!-- 의존성 주입 받을 객체 : Singleton으로 자동 생성된다. -->
	<bean id="studentDao" class="ems.member.dao.StudentDAOImpl" ></bean>
	
	<!-- 의존성 주입을 받을 객체생성과 의존성 주입 -->
	<bean id="studentService" class="ems.member.service.StudentServiceImpl">
		<!-- setter method injection -->
		<property name="dao" ref="studentDao" ></property>
		<!-- setter method parameter values -->
		<property name="school" value="고등학교"></property>
		<property name="grade" value="3"></property>
	</bean>
```
```text
property의 name은 해당하는 setter method의 이름에서 set을 제외하고 첫 번째 글자를 소문자로 바꿔서 표현한다.
property의 value는 파라미터로 들어온 값을 적는다.
```
메인 클래스에서 호출(Spring Container)
```java
public class MainSpringApp{

	public static void main(String[] args) {
		
		//1.설정파일을 입력하여 Spring Container를 생성
		// 설정파일 안에 정의된 <bean>들은 사용유무에 상관없이 모두 생성된다.
		ClassPathXmlApplicationContext context = 
				new ClassPathXmlApplicationContext("applicationContext.xml");
		
		//2. 스프링 컨테이너에서 객체를 얻는다(<bean>에 정의된 객체는 모두 얻을 수 있다)
		StudentServiceImpl service = context.getBean("studentService",StudentServiceImpl.class);
		
		//3. method 호출하여 업무 처리(getter method 이용)
		System.out.println("grade : "+service.getGrade());
		System.out.println("school : "+service.getSchool());
		
		Map<Integer, String> map = service.searchStudent();
		for( Map.Entry<Integer, String> entry : map.entrySet()) {
			System.out.println("number : "+entry.getKey()+" name : "+entry.getValue());
		}
		
		//4. Spring Container의 사용이 종료 되었다면 메모리 누수의 이슈가 있어서 반드시 닫아준다.
		if(context != null) {
			context.close();
		}
	}
}
```
Console
```console
의존성 주입받는 setter method 호출
school : 고등학교
grade : 3반
number : 1 name : 홍길동
number : 2 name : 이순신
number : 3 name : 임꺽정
number : 4 name : 왕건
```

## 4. Collection wiring - bean property가 JCF인 경우
### List
* 의존성 주입받는 setter method 선언(Service)
* Set타입은 중복된 값 제거하고 전달
```java
public class EMSInformationService {

	private List<String> developers;

	public List<String> getDevelopers() {
		return developers;
	}

	public void setDevelopers(List<String> developers) {
		this.developers = developers;
	}
}
```
스프링 설정 파일(XML)에 bean설정 
```xml
<!-- 의존성 주입을 받을 객체생성과 의존성 주입 -->
<bean id="informationService" class="ems.member.service.EMSInformationService">
	<property name="developers">
		<list>
			<value>Cheney.</value>
			<value>Eloy.</value>
			<value>Jasper.</value>
			<value>Dillon.</value>
			<value>Kian.</value>
		</list>
	</property>
</bean>
```
### Map
의존성 주입받는 setter method 선언(Service)
```java
public class EMSInformationService {

	private Map<String, String> administrators;

	public Map<String, String> getAdministrators() {
		return administrators;
	}

	public void setAdministrators(Map<String, String> administrators) {
		this.administrators = administrators;
	}	
}
```
스프링 설정 파일(XML)에 bean설정 
```xml
<!-- 의존성 주입을 받을 객체생성과 의존성 주입 -->
<bean id="informationService" class="ems.member.service.EMSInformationService">
	<property name="administrators">
		<map>
			<entry>
				<key>
					<value>Cheney</value>
				</key>
				<value>cheney@springPjt.org</value>
			</entry>
			<entry>
				<key>
					<value>Jasper</value>
				</key>
				<value>jasper@springPjt.org</value>
			</entry>
		</map>
	</property>
</bean>
```
메인 클래스에서 호출(Spring Container)
```java
public class MainSpringApp3 {

	public static void main(String[] args) {
		
		//1.설정파일을 입력하여 Spring Container를 생성
		// 설정파일 안에 정의된 <bean>들은 사용유무에 상관없이 모두 생성된다.
		ClassPathXmlApplicationContext context = 
				new ClassPathXmlApplicationContext("applicationContext.xml");
		
		//2. 스프링 컨테이너에서 객체를 얻는다(<bean>에 정의된 객체는 모두 얻을 수 있다)
		EMSInformationService service = context.getBean("informationService",EMSInformationService.class);
		
		//3. method 호출하여 업무 처리
		List<String> developers = service.getDevelopers(); 
		for(int i = 0; i < developers.size(); i++) {
			System.out.println(developers.get(i));
		}
		
		Map<String, String> administrators = service.getAdministrators();
		for( Map.Entry<String, String> entry : administrators.entrySet()) {
			System.out.println(entry.getKey()+" / "+entry.getValue());
		}
		
		//4. Spring Container의 사용이 종료 되었다면 메모리 누수의 이슈가 있어서 반드시 닫아준다.
		if(context != null) {
			context.close();
		}
	}
}
```
## 5. SpEL 이용한 DI 설정
Properties 파일 생성
```properties
#database properties 
db.driver=oracle.jdbc.OracleDriver
db.jdbcUrl=jdbc:oracle:thin:@localhost:1521:xe
db.userId=scott
db.userPw=tiger
```
스프링 설정 파일(XML)에 bean설정 
```xml
<!-- <util:properties id="빈아이디" location="classpath:프로퍼티파일명" /> -->
<!-- <context:property-placeholder location="classpath:프로퍼티파일명"/> -->

<context:property-placeholder location="classpath:database.properties"/>

<bean class="ems.member.service.DatabaseService" id="database">
	<property name="driver" value="${db.driver}"/>
	<property name="jdbcUrl" value="${db.jdbcUrl}"/>
	<property name="userId" value="${db.userId}"/>
	<property name="userPw" value="${db.userPw}"/>
</bean>
```
