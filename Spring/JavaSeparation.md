# Java Spring Config File Seperation
## 기능 단위로 분리
DAO와 Service가 있는 설정 파일
```java
@Configuration
  public class MemberConfig1 {

	@Bean
	public StudentDao studentDao() {
		return new StudentDao();
	}

	@Bean
	public StudentRegisterService registerService() {
		return new StudentRegisterService(studentDao());
	}

	@Bean
	public StudentModifyService modifyService() {
		return new StudentModifyService(studentDao());
	}

	@Bean
	public StudentSelectService selectService() {
		return new StudentSelectService(studentDao());
	}

	@Bean
	public StudentDeleteService deleteService() {
		return new StudentDeleteService(studentDao());
	}

	@Bean
	public StudentAllSelectService allSelectService() {
		return new StudentAllSelectService(studentDao());
	 }
  }
```
DBConnection과 관련된 설정 파일
```java
@Configuration
  public class MemberConfig2 {

	@Bean
	public DataBaseConnectionInfo dataBaseConnectionInfoDev() {
		DataBaseConnectionInfo infoDev = new DataBaseConnectionInfo();
		infoDev.setJdbcUrl("jdbc:oracle:thin:@localhost:1521:xe");
		infoDev.setUserId("scott");
		infoDev.setUserPw("tiger");

		return infoDev;
	}

	@Bean
	public DataBaseConnectionInfo dataBaseConnectionInfoReal() {
		DataBaseConnectionInfo infoReal = new DataBaseConnectionInfo();
		infoReal.setJdbcUrl("jdbc:oracle:thin:@192.168.0.1:1521:xe");
		infoReal.setUserId("masterid");
		infoReal.setUserPw("masterpw");

		return infoReal;
	 }
  }
  ```
Utill과 관련된 설정 파일
```java
@Configuration
  public class MemberConfig3 {
    
	@Autowired
	DataBaseConnectionInfo dataBaseConnectionInfoDev;

	@Autowired
	DataBaseConnectionInfo dataBaseConnectionInfoReal;

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

		ArrayList<String> developers = new ArrayList<String>();
		developers.add("Cheney.");
		developers.add("Eloy.");
		developers.add("Jasper.");
		developers.add("Dillon.");
		developers.add("Kian.");
		info.setDevelopers(developers);

		Map<String, String> administrators = new HashMap<String, String>();
		administrators.put("Cheney", "cheney@springPjt.org");
		administrators.put("Jasper", "jasper@springPjt.org");
		info.setAdministrators(administrators);

		Map<String, DataBaseConnectionInfo> dbInfos = new HashMap<String, DataBaseConnectionInfo>();
		dbInfos.put("dev", dataBaseConnectionInfoDev);
		dbInfos.put("real", dataBaseConnectionInfoReal);
		info.setDbInfos(dbInfos);

		return info;
	 }
  }
  ```
위 코드의 Map에서는 MemberConfig2에서 생성된 DataBaseConnection과 관련된 Bean 객체가 사용되고 있다.
```java
  dbInfos.put("dev", dataBaseConnectionInfoDev);
  dbInfos.put("real", dataBaseConnectionInfoReal);
```
다음 코드와 같이 사용되야 하기 때문에 `@Autowired`로 자동주입을 하여 필요한 객체를 얻는다.
```java
  // MemberConfig2에서 생성된 bean 객체를 자동주입 하라는 의미

  @Autowired
  DataBaseConnectionInfo dataBaseConnectionInfoDev;

  @Autowired
  DataBaseConnectionInfo dataBaseConnectionInfoReal;
```
<pre>
dataBaseConnectionInfoDev() 메소드를 사용하는 것이 아닌 <b>@Autowired</b>로 
참조형 변수에 의존성이 자동주입 되어 bean 객체가 생성 되었기 때문에
dataBaseConnectionInfoDev 참조변수를 적기만 하면 된다.
</pre>
MainClass
```java
 AnnotationConfigApplicationContext ctx =
    new AnnotationConfigApplicationContext(MemberConfig1.class, MemberConfig2.class, MemberConfig3.class);
```
## @Import
* `@Import({파일명1.class, 파일명2.class})`
* 하나의 자바 설정 파일에 Import를 시켜준다.
```java
@Configuration
  @Import({MemberConfig2.class, MemberConfig3.class})
  public class MemberConfigImport {
  	@Bean
  	public StudentDao studentDao() {
  		return new StudentDao();
  	}

  	@Bean
  	public StudentRegisterService registerService() {
  		return new StudentRegisterService(studentDao());
  	}

  	@Bean
  	public StudentModifyService modifyService() {
  		return new StudentModifyService(studentDao());
  	}

  	@Bean
  	public StudentSelectService selectService() {
  		return new StudentSelectService(studentDao());
  	}

  	@Bean
  	public StudentDeleteService deleteService() {
  		return new StudentDeleteService(studentDao());
  	}

  	@Bean
  	public StudentAllSelectService allSelectService() {
  		return new StudentAllSelectService(studentDao());
  	}
  }
```
MainClass
```java
AnnotationConfigApplicationContext ctx =
    new AnnotationConfigApplicationContext(MemberConfigImport.class);
```

