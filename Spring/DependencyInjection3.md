# 자바파일로 스프링 설정 파일 만들기
## Java로 설정하면 좋은 점
1. 스프링 설정 파일을 따로 관리할 필요가 없다. 그냥 자바 클래스이다. 찾기 쉽다.

2. 보다 명료하다. 어떤 것들이 빈으로 만들어지는지 파악하기 쉽다.

3. IDE의 자동완성 기능을 사용할 수 있다. 자바 코드이기 때문이다. 그래서 작성과 수정이 빠르다.

4. 어플리케이션 로직과 설정 코드를 동일한 언어로 만들 수 있다. 한 언어만 쓰는게 간편하니 좋다.

5. 설정 코드에 break point 를 걸어서 디버깅할 수 있다.

6. 참고자료 : [자바파일로 스프링 관련 설정](https://swdevelopment.tistory.com/313?category=915007)

## @Configuration, @Bean
```
java파일에 @Configuration 어노테이션을 선언 시 클래스는 환경구성 파일이 되며 @Bean 어노테이션을 사용할 수 있다.
```
### java파일 어노테이션 설정
```java
package ems.member.configration;

@Configuration
public class MemberConfig {
	/**
	 * //<bean id="studentDao" class="ems.member.dao.StudentDao"/>
	 * @return
	 */
	@Bean
	public StudentDao studentDao() {
		return new StudentDao();
	}
}
```
* Class에 `@Configuration` 선언으로 환경구성 파일로 설정한다.
* Method에 `@Bean` 선언으로 Bean 설정한다.
* bean의 `id속성 값`은 `method명`이 된다.
* bean의 `class속성 값`은 `반환형`이 된다.

### StudentRegisterService 의존성 주입
 ```java
    /*
	 * <bean id="registerService" class="ems.member.service.StudentRegisterService">
	        <constructor-arg ref="studentDao" />
	   </bean>
	 * @return
	 */
	@Bean
	public StudentRegisterService registerService() {
		return new StudentRegisterService(studentDao());
	}
 ```
### setter method를 이용해서 property값 설정
 ```java
 	/**
	 * <bean id="dataBaseConnectionInfoDev" class="ems.member.DataBaseConnectionInfo">
            <property name="jdbcUrl" value="jdbc:oracle:thin:@localhost:1521:xe" />
            <property name="userId" value="scott" />
            <property name="userPw" value="tiger" />
	   </bean>
	 * @return
	 */
	@Bean
	public DataBaseConnectionInfo dataBaseConnectionInfoDev() {
		DataBaseConnectionInfo inforDev = new DataBaseConnectionInfo();
		inforDev.setJdbcUrl("jdbc:oracle:thin:@localhost:1521:xe");
		inforDev.setUserId("scott");
		inforDev.setUserPw("tiger");
		
		return inforDev;
	}
 ```
### Map, List 값 설정<br/>
applicationContext.xml
```xml
<bean id="informationService" class="ems.member.service.EMSInformationService">
		<property name="info">
			<value>Education Management System program was developed in 2015.</value>
		</property>
		<property name="copyRight">
			<value>COPYRIGHT(C) 2015 EMS CO., LTD. ALL RIGHT RESERVED. CONTACT MASTER FOR MORE INFORMATION.</value>
		</property>
		<property name="ver">
			<value>The version is 1.0</value>
		</property>
		<property name="sYear">
			<value>2015</value>
		</property>
		<property name="sMonth">
			<value>1</value>
		</property>
		<property name="sDay">
			<value>1</value>
		</property>
		<property name="eYear" value="2015" />
		<property name="eMonth" value="2" />
		<property name="eDay" value="28" />
		<property name="developers">
			<list>
				<value>Cheney.</value>
				<value>Eloy.</value>
				<value>Jasper.</value>
				<value>Dillon.</value>
				<value>Kian.</value>
			</list>
		</property>
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
		<property name="dbInfos">
			<map>
				<entry>
					<key>
						<value>dev</value>
					</key>
					<ref bean="dataBaseConnectionInfoDev"/>
				</entry>
				<entry>
					<key>
						<value>real</value>
					</key>
					<ref bean="dataBaseConnectionInfoReal"/>
				</entry>
			</map>
		</property>
	</bean>
```
MemberConfig.java
```java
	@Bean
	public EMSInformationService informationService() {
		EMSInformationService info = new EMSInformationService();
		info.setInfo("Education Management System program was developed in 2015.");
		info.setCopyRight("COPYRIGHT(C) 2015 EMS CO., LTD. ALL RIGHT RESERVED. CONTACT MASTER FOR MORE INFORMATION.");
		info.setVer("The version is 1.0");
		info.setsYear(2015);
		info.setsMonth(1);
		info.setsDay(1);
		info.seteYear(2015);
		info.seteMonth(2);
		info.seteDay(28);
		
		List<String> developers = new ArrayList<String>();
		developers.add("Cheney.");
		developers.add("Eloy.");
		developers.add("Jasper.");
		developers.add("Dillon.");
		developers.add("Kian.");
		
		//List를 set method에 넣어준다.
		info.setDevelopers(developers);
		
		Map<String, String> administrators = new HashMap<String, String>();
		administrators.put("Cheney", "cheney@springPjt.org");
		administrators.put("Jasper","jasper@springPjt.org");
		
		//Map을 set method에 넣어준다.
		info.setAdministrators(administrators);
		
		Map<String, DataBaseConnectionInfo> dbInfos = new HashMap<String, DataBaseConnectionInfo>();
		dbInfos.put("dev", dataBaseConnectionInfoDev());
		dbInfos.put("real", dataBaseConnectionInfoReal());
		
		//Map을 set method에 넣어준다.
		info.setDbInfos(dbInfos);
		
		return info;
	}
```
