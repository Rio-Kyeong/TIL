# redirect&foward
<pre>
forward(기본 페이지 이동방식)
- JSP가 반드시 /WEB-INF/views/ 폴더 하위에 존재해야 한다.
- <b>JSP가 데이터를 처리하여 서비스해야하는 상태</b>이므로 JSP가 직접 요청이 되지 않아야 한다.

redirect
- JSP가 webapp 폴더의 하위에 위치해야 한다.
- <b>JSP가 데이터를 처리하지 않고 서비스되어지는 상태</b>(값이 넘어가지 않는다)
- ViewResolver를 사용하지 않고 JSP를 바로 응답한다.
- 지금의 페이지에서 특정 페이지로 전환하는 기능
</pre>
## redirect

Controller
```java
// Modify(회원정보 수정 컨트롤러)
@RequestMapping(value = "/modifyForm")
public String modifyForm(Model model, HttpServletRequest request) {
    
    HttpSession session = request.getSession(); //세션선언
    Member member = (Member) session.getAttribute("member"); //member session 저장
    
    
    if(null == member) { //session이 null이면
        return "redirect:/"; //redirect로 로그인을 위해 main(/)페이지 반환
    } else {
        model.addAttribute("member", service.memberSearch(member));
    }
    
    return "/member/modifyForm";
}
```
## Interceptor
```
인터셉터는 DispatcherServlet이 컨트롤러를 요청하기 전,후에 요청과 응답을 가로채서 가공할 수 있도록 해준다.
리다이렉트를 사용해야 하는 경우가 많은 경우 HandlerInterceptor를 이용할 수 있다.

HandlerInterceptorAdapter Interface를 상속받는다.

preHandle() : controller가 호출되기 전에 실행
postHandle() : controller가 호출되고 난 후에 실행
afterCompletion() : controller와 view가 모든 작업을 완료 후 실행
```
Intercepter
```java
public class MemberLoginInterceptor extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler) throws Exception {
        
        HttpSession session = request.getSession(false); 
        if(session != null) { //세션이 null이 아니면
            Object obj = session.getAttribute("member"); //obj에 member session을 넣어준다 
            if(obj != null) //obj가 null이 아니면 true를 리턴
                return true;
        }
        //session이 없으면 main(/)페이지 이동
        response.sendRedirect(request.getContextPath() + "/");
        //response.sendRedirect("이동페이지URL") : 페이지 이동
        //request.getContextPath() : 프로젝트 Path (컨텍스트 path) 만 가져옵니다.
        //예) http://localhost/lec21/list.jsp
        //return "/lec21" 

        return false;
    }

    @Override
    public void postHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler,
            ModelAndView modelAndView) throws Exception {
        
        super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
            HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
        
        super.afterCompletion(request, response, handler, ex);
    }
}
```
servlet-context
```xml
<interceptors> 
    <interceptor>
        <mapping path="/member/modifyForm"/> 
        <mapping path="/member/removeForm"/> 
        <!-- mapping path가 요청이 되면 Interceptor가 동작 -->
        <beans:bean class="com.bs.lec21.member.MemberLoginInterceptor"/> 
        <!-- Interceptor class bean 설정 -->
    </interceptor>
</interceptors>
```
## foward
controller
```java
//로그인 method
@RequestMapping(value = "/login/login.do")
public String loginProcess(LoginVO lVO, Model model) {
    
    String url = "";

    LoginInfoDomain lid = null;
    lid = service.searchLogin(lVO);

    if(lid == null || lid.getMember_withdrawal().equals("Y")){//만약 login 정보가 없거나 탈퇴 정보가 Y이면
        model.addAttribute("loginFail","loginFail"); 
        url = "prj3/login/login";
    }else {
        String id = lid.getMember_id();
        model.addAttribute("id", id);//session
        url = "forward:/main/main.do"; //forward를 이용해서 main페이지로 이동
    }
    return url;
}

//메인 페이지 method
@RequestMapping(value = "/main/main.do")
public String mainForm(Model model) {
    
    //forword를 함으로써 mainForm()의 작업도 같이 반환할 수 있다.
    model.addAttribute("topList", service.selectProductListTopUser());
    model.addAttribute("botList", service.selectProductListBotUser());
    
    return "prj3/main/main_all";
}
```
```
forward를 사용 함으로써 login method의 session의 값도 유지가 가능하며, 
mainForm method를 거치기 때문에 그 안에서의 작업도 같이 반환된다.
```
## 요약
```
- redirect를 사용 : 이전페이지에서 사용한 값이 넘어가지 않는다. 
  return “redirect:xx.do”; 

- forward를 사용 : 이전페이지에서 사용한 값을 넘길 수 있다. 
  return “forward:xx.do”; 
```
