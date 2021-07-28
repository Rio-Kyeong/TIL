# Spring MVC
## 프로젝트 전체 구조
![구조](https://github.com/RyuKyeongWoo/TIL/blob/main/Spring/img/rescue.PNG)
## web.xml
<pre>
WEB-INF폴더 하위에 존재하는 <b>Web Application 설정</b>을 위한 <b>Deployment Descriptor(배포서술자)</b>
사용자의 요청이 들어오면 <b>DispatcherServlet</b>을 servlet으로 등록한다. 
</pre>
## servlet-context.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:beans="http://www.springframework.org/schema/beans"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
      <!-- DispatcherServlet과 관련 설정 필수 -->
      <!--어노테이션을 사용한다고 선언-->
      <annotation-driven />
      <!-- HTML 리소스 디렉토리 정의 -->
      <resources mapping="/resources/**" location="/resources/" /> 
      <!-- ViewResolver로 jsp와 name 을 매핑 -->
      <beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"> 
            <beans:property name="prefix" value="/WEB-INF/views/" />
            <beans:property name="suffix" value=".jsp" />
      </beans:bean> 
      
      <context:component-scan base-package="com.bs.lec16" /> 
      <!-- 베이스 패키지 하위 모든 어노테이션을 스캔해서 빈으로 등록하겠다는 것 -->
</beans:beans>
```
* `요청과 관련된 객체를 정의`(주로 View 지원 bean을 설정)
* url과 관련된 **controller**나, **@(어노테이션), ViewResolver, Interceptor, MultipartResolver** 등의 설정
## root-contex.xml
* servlet-context.xml 과는 반대로 `view와 관련되지 않은 객체를 정의`
* **Service, Repository(DAO), DB등 비즈니스 로직**과 관련된 설정
## resources
![resources](https://github.com/RyuKyeongWoo/TIL/blob/main/Spring/img/resources.PNG)
* `IMAGE, CSS, JS HTML`등 Get으로 접근하여 사용하고 싶은 파일들을 리소스 폴더로 설정하여 사용한다.
## Controller
* View와 Model 사이의 인터페이스 역할이다.
* Model/View에 대한 사용자 입력 및 요청을 수신하여 그에 따라 적절한 결과를 Model에 담아 View에 전달한다.
* `Servlet`
## View
* `실제로 렌더링되어 보이는 페이지`를 담당한다.
* `JSP`
