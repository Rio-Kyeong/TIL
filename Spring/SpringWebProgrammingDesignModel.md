# 웹 프로그래밍 설계 모델
## Model1
![Model1](https://github.com/RyuKyeongWoo/TIL/blob/main/Spring/img/Moel1.PNG)
* 사용자에게 보여주는 `모든 것을 하나의 파일로` 처리하기 때문에 개발 속도가 빠르지만 `유지보수`가 어렵다.
## Model2
![Model2](https://github.com/RyuKyeongWoo/TIL/blob/main/Spring/img/Model2.PNG)
* Model1 방식을 보완하기 위해 나온 방식으로 MVC를 기본으로 한다.
* 각가의 기능을 다 `모듈화`시켰기 때문에 유지보수가 좋아진다.
## 스프링 MVC 프레임워크 설계 구조
![MVC](https://github.com/RyuKyeongWoo/TIL/blob/main/Spring/img/SpringRescue.PNG)
1. `DispatcherServlet`이 클라이언트(Client)로부터 **요청(Request)** 을 받는다.
2. `HandlerMapping`이 받아온 정보를 통해 적합한 **Contorller**를 찾는다.<br/>
    (`DispatcherServlet` → `HandlerMapping` → `DispatcherServlet`)
3. `HandlerAdapter`가 선택된 **Controller**에서 적합한 Method를 찾고 실행한다.<br/>
    (`DispatcherServlet` → `HandlerAdapter` → `DispatcherServlet`)
4. 사용자 응답에 필요한 데이터를 **Model**로 가져오고 `ViewResolver`가 **Model**과 가장 적합한 **JSP** 문서를 찾는다.<br/>
    (`DispatcherServlet` → `ViewResolver` → `DispatcherServlet`)
5. `View`응답을 생성하고 브라우저로 해당 **JSP파일** 를 클라이언트에게 **응답(Response)** 시킨다.
```
개발자는 MVC를 사용하기위해 모든 부분을 구현할 필요가 없고 Controller와 JSP(View)만 만들면 된다.
```
## DispatcherServlet 설정
* web.xml에 서블릿을 매핑한다. 모든 요청을 처리하므로 /(root)로 url을 매핑한다.
* ex `<url-pattern>*.do</url-pattern>` : *.do로 들어오는 요청을 처리한다.
```xml
<servlet-mapping>
<servlet-name>appServlet</servlet-name>
<url-pattern>/</url-pattern>
</servlet-mapping>
```
* 서블릿으로 등록 후 init-param(초기 파라미터)로 스프링 설정파일을 설정해줘야한다.
* 파일을 설정하지 않으면 디폴트 설정파일을 생성한다.(appServlet-context.xml)
* 설정 파일에서 스프링 컨테이너가 생성되고 `HandlerMapping`, `HandlerAdapter`, `ViewResolver`는 자동으로 생성된다.
```xml
<servlet>
		<servlet-name>appServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
</servlet>
```
## Controller 객체 - @Controller
```java
@Controller //Controller 객체로 사용할 클래스 정의
public class HomeController{
    
}
```
* 스프링 설정 파일에 `<annotation-driven />`테그를 명시해준다.<br/>
* `HandlerMapping`이 적합한 Controller를 찾을 수 있도록 `@Controller` annotation을 붙여준다.
## Controller 객체 - @RequestMapping
```java
@RequestMapping("/success")
public String success(Model model){
    return "success";
}
```
* `HandlerAdapter`가 적합한 메소드를 찾을 수 있도록 메소드에 `@RequestMapping` annotation을 붙여준다.
## View 객체
```java
@RequestMapping("/success")
public String success(Model model){
    return "success";
}
```
```xml
<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <beans:property name="prefix" value="/WEB-INF/views/" />
    <beans:property name="suffix" value=".jsp" />
</beans:bean>
```
* JSP파일명 : /WEB-INF/views/success.jsp
* `ViewResolver` 객체는 **prefix값 + 컨트롤러 안에 맵핑되는 리턴 값 + suffix값** 을 합쳐서 jsp의 경로를 만든다.

## 전체적인 웹 프로그래밍 구조
![all](https://github.com/RyuKyeongWoo/TIL/blob/main/Spring/img/AllRescue.PNG)
1. 사용자가 /success를 요청한다.
2. `DispatcherServlet`이 받는다.<br/>
    (Servlet등록, 초기 파라미터로 스프링 설정 파일등록)
3. Controller를 찾는다(`HandlerMapping`)
4. Method를 찾고 실행한다.(`HandlerAdapter`)
5. View를 검색한다(`ViewResolver`에서 검색 및 실행)
6. 사용자에게 찾은 jsp를 응답시켜 보여준다.
