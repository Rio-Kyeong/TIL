# Session&Cookie
## Connectionless Protocol
<pre>
웹 서비스는 HTTP 프로토콜을 기반으로 하는데, HTTP 프로토콜은 클라이언트와 서버의 관계를 유지 하지 않는 특징이 있다.
응답 후 연결을 해제함으로써 서버의 부하를 줄일 수 있는 장점이 있으나, 클라이언트의 요청 시마다 
서버와 매번 새로운 연결이 생성되기 때문에 일반적인 로그인 상태 유지, 장바구니 등의 기능을 구현하기 어렵다.

이러한 Connectionless Protocol의 불편함을 해결하기 위해서 세션과 쿠키를 이용한다.

세션과 쿠키는 <b>클라이언트와 서버의 연결 상태를 유지</b>해주는 방법으로,
<b>세션은 서버에서 연결 정보를 관리</b>하는 반면 <b>쿠키는 클라이언트에서 연결 정보를 관리</b>하는데 차이가 있다.
</pre>

## Session

### 세션 선언
1. HttpServletRequest 객체 이용
```java
@RequestMapping(value = "/login", method = RequestMethod.POST)
public String memLogin(Member member, HttpServletRequest request) {
    //1. 파라미터에 HttpServletRequest 객체 선언

    Member mem = service.memberSearch(member);
    //2. getSession()으로 세션을 얻는다
    HttpSession session = request.getSession();
    //3. 세션 값 설정 
    //session.setAttribute("세션 명", 세션 값);
    session.setAttribute("member", mem);
    
    return "/member/loginOk";
}
```
2. HttpSession 객체 이용
```java
@RequestMapping(value = "/login", method = RequestMethod.POST)
public String memLogin(Member member, HttpSession session) {
    //1. 파라미터에 HttpSession 객체 선언

    Member mem = service.memberSearch(member);
    //2. 세션 값 설정 
    //session.setAttribute("세션 명", 세션 값);
    session.setAttribute("member", mem);
    
    return "/member/loginOk";
}
```
### 세션 사용
```java
@RequestMapping(value = "/modifyForm", method = RequestMethod.GET)
public ModelAndView modifyForm(HttpServletRequest request) {
    //1. 파라미터에 HttpServletRequest 객체 선언

    //2. getSession()으로 세션을 얻는다
    HttpSession session = request.getSession();

    //3. member 세션을 불러와서 사용
    Member member = (Member) session.getAttribute("member");
    
    ModelAndView mav = new ModelAndView();
    mav.addObject("member", service.memberSearch(member));
    
    mav.setViewName("/member/modifyForm");
    
    return mav;
}
```
### 세션 삭제
```java
@RequestMapping("/logout")
public String memLogout(Member member, HttpSession session) {
    //1. 파라미터에 HttpSession 객체 선언

    //2. 세션 값 삭제
    session.invalidate();
    
    return "/member/logoutOk";
}
```
### 세션 주요 메소드 및 플로어
![session](https://github.com/RyuKyeongWoo/TIL/blob/main/Spring/img/session.PNG)

## Cookie

### 쿠키 설정
```java
@RequestMapping("/main")
public String mallMain(Mall mall, HttpServletResponse response){    
    //1. 파라미터에 HttpServletRequest 객체 선언

    //2. 쿠키생성 : new Cookie("쿠키 명", 쿠키 값);
    //   (쿠키는 문자열만 심어지며, 여러개의 쿠키를 심을 수 있다)
    Cookie genderCookie = new Cookie("gender", mall.getGender());

    //쿠키의 존재여부 확인
    if(mall.isCookieDel()) {
        //쿠키가 파기시간 설정
        genderCookie.setMaxAge(0);
        mall.setGender(null);
    } else {
        //쿠키의 파기시간 설정 : 1분 * 1시간 * 24시간 * 30일 = 한달을 초로 환산한 시간
        //ex) setMaxAge(60*2); - 2분
        genderCookie.setMaxAge(60*60*24*30);
    }
    //파라미터로 받은 HttpServletResponse에 쿠키를 담는다.
    response.addCookie(genderCookie);

    return "/mall/main";
}
```
### 쿠키 가져오기
```java
@RequestMapping("/index")
public String mallIndex(Mall mall, 
    @CookieValue(value="gender", required=false) Cookie genderCookie, HttpServletRequest request){
    //@CookieValue(value="쿠키 명", defaultValue=“쿠키가 없을 때 읽어들일 값”, required=false) Cookie genderCookie
    
    if(genderCookie != null){
        // 쿠키 값 가져오기
        mall.setGender(genderCookie.getValue());
    }	
    
    return "/mall/index";
}
```
### View에서 쿠키 사용
```jsp
${ cookie.gender.value }
```
