# Dependency Injection
## 1. DI

```test
DI(Dependency Injection)란 스프링이 제공하는 의존 관계 주입 기능
객체를 직접 생성하는 게 아니라 외부에서 생성한 후 주입 시켜주는 방식
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
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
 		http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	<!-- 의존성 주입 받을 객체 : Singleton으로 자동 생성된다. -->
	<bean id="studentDao" class="ems.member.dao.StudentDAOImpl" ></bean>
	
	<!-- 의존성 주입을 받을 객체생성과 의존성 주입 -->
	<bean id="studentService" class="ems.member.service.StudentServiceImpl">
		<!-- constructor injection -->
		<constructor-arg ref="studentDao" ></constructor-arg>
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
	
	@Override
	public String toString() {
		return "school : " + school + "\ngrade : " + grade+"반";
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
		
		//3. method 호출하여 업무 처리
		
		//toString을 오버라이드 했기 때문에 자동으로 toStirng을 호출
		System.out.println(service);
		
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
