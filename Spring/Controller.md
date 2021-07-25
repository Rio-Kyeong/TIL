# Controller 객체 구현
## @RequestMapping을 이용한 URL맵핑
```java
@Controller
@RequestMapping("/member")
public class MemberController {

    @RequestMapping("/memJoin")
    public String memJoin(){

    }

    @RequestMapping("/memLogin")
    public String memJoin(){

    }
}
``` 
* 클래스에 `@RequestMapping`으로 URL을 명시하면, 클래스 내의 모든 메소드들이 받는 요청URL 앞에 공통으로 붙는다.
* ex memJoin() 메소드는 "/member/memJoin"이라는 요청URL을 받는다.

## 요청 파라미터
## `HttpServletRequest` 객체를 이용한 HTTP 전송 정보 얻기
![HttpServletRequest](https://github.com/RyuKyeongWoo/TIL/blob/main/Spring/img/httpservletrequest.png)<br/>
* `HTML Form Control의 name속성 값`과 `request.getParameter 값`은 반드시 같아야 한다.
```java
	@RequestMapping(value="/memJoin", method=RequestMethod.POST)
	public String memJoin(Model model, HttpServletRequest request) {
		String memId = request.getParameter("memId");
		String memPw = request.getParameter("memPw");
		
		service.memberRegister(memId, memPw);
		
		model.addAttribute("memId", memId); 
		model.addAttribute("memPw", memPw);
		//Model를 이용하지 않고 View에서 el 표현식에 param을 붙여서 사용할 수도 있다.
		//ex ${ param.memId }
		return "memJoinOk";
	}
```

## `@RequestParam` 어노테이션을 이용한 HTTP 전송 정보 얻기 - 단일 데이터형 받기
![RequestParam](https://github.com/RyuKyeongWoo/TIL/blob/main/Spring/img/requestparam.png)<br/>
* `HTML Form Control의 name속성 값`과 `method의 Parameter명`이 반드시 같아야 한다.
```java
@RequestMapping(value="/memLogin", method=RequestMethod.POST)
	public String memLogin(Model model, String memId, 
			@RequestParam(value = "memPw", required = false, defaultValue = "0")String memPw) {
			
		model.addAttribute("memId", memId); 
		model.addAttribute("memPw", memPw);
		//Model를 이용하지 않고 View에서 el 표현식에 param을 붙여서 사용할 수도 있다.
		//ex ${ param.memId }
		return "memLoginOk";
    }
```
* `@RequestParam 어노테이션은 생략가능`
* `required = false` : 값이 넘어 오지 않아도 Exception이 발생하지 않음
* `defaultValue = "0"` : 값이 넘어오지 않았을 경우에 사용할 기본 값 설정
* 보통 javascript에서 유효성 검증을 하고 값을 받기 때문에 잘 쓰지 않는다.

## `커멘드 객체`를 이용한 HTTP전송 정보 얻기 - VO(Value Object)로 받기
![VO](https://github.com/RyuKyeongWoo/TIL/blob/main/Spring/img/vo.png)<br/>
* `HTML Form Control의 name속성 값`과 `VO의 인스턴스 변수명`이 반드시 같아야 한다.<br/>

**Controller**
```java
@RequestMapping(value="/memLogin", method=RequestMethod.POST)
public String memLogin(Member member) {
	String url ="";
	member = service.memberSearch(member.getMemId(), member.getMemPw());
	
	if(member == null) {
		System.out.println("로그인 실패");
		url = "memLoginFail";
	}else{
		System.out.println("로그인 성공");
		url = "memLoginOk";
	}
	return url;
}
```
**View ( memLoginOK )**
```html
<body>
	<h1> memLoginOk </h1>
	ID : ${member.memId}<br />
	PW : ${member.memPw}<br />
	<!-- model에 넣지 않고 Controller 매개변수로 사용한 인스턴스 명을 넣어서 사용할 수 있다.-->
	<a href="/lec17/resources/html/index.html"> Go Main </a>
</body>
```
### `데이터가 중첩 커멘드 객체`를 이용한 List 구조인 경우
**html**
```html
PHONE1 : 
<input type="text" name="memPhones[0].memPhone1" size="5"> -
<input type="text" name="memPhones[0].memPhone2" size="5"> -
<input type="text" name="memPhones[0].memPhone3" size="5">

PHONE2 : 
<input type="text" name="memPhones[1].memPhone1" size="5"> -
<input type="text" name="memPhones[1].memPhone2" size="5"> -
<input type="text" name="memPhones[1].memPhone3" size="5">
```
**VO**
```java
private List<MemPhone> memPhones;
//MemPhone Class는 변수 memPhone1,2,3으로 이루어진 VO
```
**jsp**
```jsp
PHONE1 : ${member.memPhones[0].memPhone1} - ${member.memPhones[0].memPhone2} - ${member.memPhones[0].memPhone3} <br/>
PHONE2 : ${member.memPhones[1].memPhone1} - ${member.memPhones[1].memPhone2} - ${member.memPhones[1].memPhone3} <br/>
```

## @ModelAttribute 
1. 파라미터 객체 옆에 사용 - `커멘드 객체의 이름 변경`<br/>

**Controller**
```java
@RequestMapping(value = "/memRemove", method = RequestMethod.POST)
	public String memRemove(@ModelAttribute("mem") Member member) {
	
	}
```
**View**
```java
ID : ${ mem.memId }
```

2. 메소드 위에 사용 - `공통 호출`<br/>

**Controller**
```java
@ModelAttribute("serverTime")
	public String getServerTime(Locale locale) {
		
		Date date = new Date();
		DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);
		
		return dateFormat.format(date);
	}
```
- 메서드 위에 `@ModelAttribute` 가 사용된 경우, 해당 컨트롤러 내의 어떠한 핸들러 메서드들보다 먼저 동작하게 된다.
- `@modelAttribute` 가 적용되어 있는 메서드는 어느 메서드가 호출되든 간에 항상 공통적으로 호출된다.

## Model & ModelAndView
* `Model` : 뷰에 데이터만을 전달하기 위한 객체
	1. method의 매개변수에 `Model을 선언`한다.  
	2. method안에서 `model객체.addAttribute(“이름”,값);`

* `ModelAndView` : 데이터와 뷰의 이름을 함께 전달하는 객체
```java
	@RequestMapping(value = "/memModify", method = RequestMethod.POST)
	public ModelAndView memModify(Member member) {
		//1. 반환형을 ModelAndView 선언
		
		Member[] members = service.memberModify(member);
		//2. ModelAndView 클래스 객체화
		ModelAndView mav = new ModelAndView();
		
		//3. View에 전달할 값 설정(request scope객체에 값이 설정된다)
		//   forward로 이동한 페이지에서만 값을 사용할 수 있다.
		mav.addObject("memBef", members[0]);
		mav.addObject("memAft", members[1]);
		
		//4. View명 설정
		mav.setViewName("memModifyOk");
		
		//5. View페이지와 View에 전달할 값을 가진 mav 객체를 return한다.
		return mav;
	}
```


