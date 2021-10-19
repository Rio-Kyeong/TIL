# HTTP Header
## HTTP 헤더 개요
<pre>
- <b>HTTP Header</b> : HTTP 전송에 필요한 모든 부가정보
- 예) 메시지 바디의 내용, 메시지 바디의 크기, 압축, 인증, 요청 클라이언트, 서버 정보, 캐시 관리 정보 ...등등
- 표준 해더가 있지만 너무 많고 필요시 임의의 헤더 추가 가능하다. 예) TOKEN : AEJEONG
</pre>
## 과거의 HTTP 헤더 (RFC2616)
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/RFC2616.PNG"/>
- <b>Request 헤더</b> : 요청 정보, 예) User-Agent: Mozilla/5.0 (Macintosh; ..) -> 클라이언트가 서버에게 유저의 정보를 알려준다.
- <b>Response 헤더</b> : 응답 정보, 예) Server: Apache -> 서버가 클라이언트에게 오리진 서버의 정보를 알려준다.
- <b>General 헤더</b> : 메시지 전체에 적용되는 정보, 예) Connection: close
- <b>Entity 헤더</b> : 엔티티 바디 정보, 예) Content-Type: text/html, Content-Length: 3423
  -> HTTP Body는 엔티티 본문을 전달할 때 사용하는데 요청/응답에서 전달할 실제 데이터이다.
     여기서 <b>엔티티 헤더는 엔티티 본문의 데이터를 해석할 수 있는 정보를 제공</b>한다.
</pre>
## 2014년 이후 현재 HTTP 헤더 (RFC7230 ~ 7235)
<pre>
<b>변경된 점</b>
- 단어의 변화 : Entity -> Representation(표현)
- 표현 = 표현 메타데이터(표현 헤더) + 표현 데이터(메시지 본문)
- <b>즉 실제 전달되는 데이터를 표현이라고 정의하였다.</b>

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/RFC7230.PNG"/>
- 메시지 본문(body)을 통해 표현 데이터 전달
- 메시지 본문 = 페이로드(payload)
- <b>표현</b>은 요청이나 응답에서 전달할 <b>실제 데이터</b> -> 표현 헤더 + 표현 데이터
- <b>표현 헤더</b>는 표현 데이터를 해석할 수 있는 정보 제공 (데이터 유형, 길이, 압축 정보 등등)

<b>표현(Representation)</b> - 표현 헤더는 전송, 응답 둘다 사용
- <b>Content-Type</b> : 표현 데이터의 형식 설명 미디어 타입, 문자 인코딩
  • EX) text/html; charset=utf-8 , application/json

- <b>Content-Encoding</b> : 표현 데이터의 압축 방식
  • 표현 데이터 압축하기 위해 사용
  • 데이터 전달하는 곳에서 압축 후 <b>인코딩 헤더</b> 추가
  • 데이터 읽는 쪽에서 <b>인코딩 헤더의 정보로 압축 해제</b>
  • EX) gzip, deflate, identity(압축, 변경없음)

- <b>Content-Language</b> : 표현 데이터의 자연 언어
  • 표현 데이터의 자연 언어를 표현
  • EX) ko, en, en-US

- <b>Content-Length</b> : 표현 데이터의 길이(바이트 단위)
  • Transfer-Encoding(전송 코딩) 사용 시 사용하면 안됨
</pre>
## 협상(Content Negotiation)
<pre>
<b>클라이언트가 선호하는 표현 요청</b>
-> 클라이언트가 서버에게 원하는 방식을 요청한다.
-> 즉 협상은 요청(request)할 때만 사용

<b>협상의 종류</b>
- <b>Accept</b> : 클라이언트가 선호하는 미디어 <b>타입</b> 전달
- <b>Accept-Charset</b> : 클라이언트가 선호하는 문자 인코딩
- <b>Accept-Encoding</b> : 클라이언트가 선호하는 압축 인코딩
- <b>Accept-Language</b> : 클라이언트가 선호하는 자연 언어

<b>협상과 우선순위</b>
- Quality Values(q) 값 사용
- 값이 범위는 0~1로, 클수록 높은 우선순위 갖는다. -> 생략하면 1
  <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/ContentNegotiation.PNG"/>

  -> ko-KR;q=1 (생략)
  -> 숫자 높은대로 우선순위 갖는다( ko-KR -> en-US )
  -> <u>이렇게 설계하면, 서버가 독일어(우선)와 영어밖에 지원하지 않을 때,</u>
     <u>클라이언트가 요청한 협상 우선순위 내용에 따라 content-language를 영어로 설정하여 응답한다.</u>
  -> 이러한 서버에, 우선순위 없이 ko-KR만 요청할 경우, 요청받은 랭귀지를 지원하지 않기 때문에 곧바로 독일어로 응답할 수 있다.
  -> 그러니 우선순위를 설정해서 요청하는 것!

<b>협상과 우선순위: 구체적인 것이 우선</b>
- Accept: text/*, text/plain, text/plain;format=flowed, */*
  -> 클라이언트가 위와 같은 미디어 타입을 요청할 때
     1. text/plain;format=flowed (가장 구체적이므로 높은 우선순위 갖는다)
     2. text/plain
     3. text/*
     4. */*
</pre>
## HTTP의 전송 방식
<pre>
<b>전송 방식</b>
- <b>단순 전송</b> : 한번에 요청하고 한 번에 다 받는다. (Content-Length 사용)

- <b>압축 전송</b> : 압축해서 클라이언트에게 응답한다. (Content-Encoding 사용)

- <b>분할 전송(Transfer-Encoding)</b> : 서버에서 응답이 만들어질 때마다 분할되서 클라이언트로 보낸다.
  • 클라이언트는 chunked 로 받는다.
  • 한번에 보내기엔 큰 용량일 때 또는 시간이 오래걸릴 때 사용
  • <b>content-length를 사용하지 않는다(쪼개서 보내기에)</b>
  
- <b>범위 전송(Reang, Content-Range)</b> : 클라이언트에서 범위를 정해서 요청하고 서버는 해당 범위만 전달한다.
  • EX) 이미지를 받는데 중간에 받다가 끊겼다. 그럼 다시 처음부터 다시 요청하는 것이 아닌 못 받은 나머지 범위를 지정해서 요청하고 받는다.
</pre>
## 헤더의 일반 정보
<pre>
<b>일반 정보</b>
- <b>From</b> : 유저 에이전트의 이메일 정보
- <b>Referer</b> : 이전 웹 페이지 주소
- <b>User-Agent</b> : 유저 에이전트 애플리케이션 정보
- <b>Server</b> : 요청을 처리하는 origin 서버의 소프트웨어 정보
- <b>Date</b> : 메시지가 생성된 날짜

<b>From</b>
• 유저 에이전트의 이메일 정보 (일반적으로 많이 사용되지 않음)
• 요청시, 검색엔진 같은 곳에서 주로 사용

<b>Referer (referer는 referrer의 오타 -> 여담: 처음에 잘못 세팅되어 현재는 그냥 사용하는 상황)</b>
• EX) 구글에서 hello를 검색하여 어느 웹사이트로 이동힐 때, 검색과정에서 가장 마지막으로 거친 이전의 웹 페이지 주소를 나타낸다.
• Referer를 사용해서 유입 경로 분석 가능

<b>User-Agent</b>
• 사용자 본인의 웹브라우저의 정보 (요청에서 사용)
• 어떤 종류의 브라우저에서 장애가 발생하는지 파악 가능

<b>Server</b>
• 요청을 처리하는 ORIGIN 서버 소프트웨어 정보 (응답에서 사용)
• Http 요청을 보내면 수많은 proxy 서버를 거쳐 최종적으로 origin 서버에 닿게 된다.

<b>Date</b>
• 메시지가 발생한 날짜와 시간 (응답에서 사용)
</pre>
## 헤더의 특별한 정보
<pre>
<b>특별 정보</b>
- <b>Host</b> : 요청한 호스트 정보(도메인)
- <b>Location</b> : 페이지 redirection
- <b>Allow</b> : 허용 가능한 HTTP 메서드
- <b>Retry-After</b> : 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간

<b>Host</b>
• 하나의 서버(or IP)가 여러 도메인을 처리해야 할 경우
• reqeust(요청)에서 보낸 Host 정보를 읽고 서버에서 hosting을 진행한다.
  <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/Host.PNG"/>
  - 하나의 서버에 여러개의 도메인을 호스팅하고 있을 때 IP 만으로 찾아갈수 없기 때문에 Host 필드를 헤더에 필수로 정의

<b>Location</b>
• 웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면, Location 위치로 이동한다 (리다이렉트)
• 201 에서는 요청에 의해 생성된 리소스 URI을 Location에 담아서 응답한다.

<b>Allow</b>
• 405(Method Not Allowed)를 사용할 때 응답 HTTP header의 Allow에 지원하는 메소드를 나열하여 응답한다.
• EX) HTTP/1.1 405 Method Not Allowed
      Allow: GET,PUT,DELETE,OPTIONS,HEAD

<b>Retry-After</b>
• 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간
• 503 ERROR 발생 시 서비스가 언제까지 불능인지 알려줄 수 있다.
</pre>
## 헤더의 인증 정보
<pre>
<b>Authorization</b> : 클라이언트 인증 정보를 서버에 전달
• 예) Authorization: Basic xxxxxxxxxxx

<b>WWW-Authenticate</b> : WWW-Authenticate 응답 헤더는 리소스의 액세스를 얻기 위한 인증 방법을 정의
• 401(Unauthorized) 응답과 함께 사용
• 예) HTTP/1.1 401 Unauthorized
      WWW-Authenticate: realm="apps", type=1, title="Login to \"apps\"", Basic realm="simple"

<b>realm이란?</b>
- 요청받은 문서의 집합을 큰따음표로 감싼것으로 클라이언트가 이것을 보고 어떤 비밀번호를 사용해야하는지 알 수 있게 해준다.
</pre>
## 헤더의 쿠키 정보
<pre>
<b>쿠키</b>
- <b>set-cookie</b> : 서버에서 클라이언트로 쿠키 전달(응답:response)
- <b>cookie</b> : 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청시 서버로 전달

<b>쿠키의 문제 - 모든 요청에 정보를 넘기는 문제</b>
- 모든 요청마다 state(사용자 정보)를 넣어 요청한다면
  -> 문제1) 모든 요청에 사용자 정보가 포함되도록 개발해야함 ( 난이도 up )
  -> 문제2) Browser를 완전히 종료하고 다시 열면 저장된 상태가 모두 날라간다.
  <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/CookieProblem.PNG"/>

<b>쿠키 저장소</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/set-Cookie.PNG"/>
1. 서버에서 아이디 또는 인증토큰등을 Set-Cookie 과정을 거쳐 클라이언트로 쿠키 전달
2. 쿠키 저장소에 저장
3. 클라이언트가 요청할 때 브라우저가 자동으로 쿠키저장소의 저장되어 있는 데이터를 찾아서 header에 포함

<b>쿠키 사용처</b>
- 사용자 로그인 세션 관리
- 광고 정보 트래킹
- <b>쿠키 정보는 항상 서버에 전송됨</b>
- 네트워크 트래픽 추가 유발 (단점)
- 최소한의 정보만 사용(세션 id, 인증 토큰)
- 서버에 전송하지 않고, 웹 브라우저 내부에 데이터를 저장하고 싶으면 웹 스토리지 참고
  -> 클라이언트에서 웹 스토리지에 보관하고 있다가 필요할 때만 서버에게 준다.
- <b>보안에 민감한 데이터는 저장하면 안됨!</b>

<b>쿠키 생명주기 (Expires, Max-age)</b>
- set-cookie: <b>expires</b> = Mon, 31-DEC-2021 12:00:00 GMT -> 만료일 되면 쿠키 삭제
- set-cookie: <b>max-age</b> = 3600 (3600초) -> 0이나 음수를 지정하면 쿠키 삭제
- <b>세션 쿠키</b> : 만료 날짜 생략하면 브라우저 종료시 까지만 유지
- <b>영속 쿠키</b> : 만료 날짜 입력하면 해당 날짜까지 유지

<b>쿠키 도메인</b>
- 명시 : 명시한 문서 기준 도메인 + 서브 도메인 포함
- 예) domain=example.org를 지정해서 쿠키 생성 하게 되면 example.org는 물론이고 dev.example.org의 쿠키에도 접근 가능하다.
      만약 도메인 지정을 생략한다면 example.org 에서만 쿠키 접근할 수 있다. dev.example.org에서는 불가능

<b>쿠키 - 경로 (Path=/루트로 지정)</b>
- 해당 경로를 포함한 하위 경로 페이지에서만 쿠키 접근 가능 (하위 경로는 모두 가능)

<b>쿠키 - 보안</b>
- <b>Secure</b>
  • 쿠키는 http, https 구분하지 않고 전송
  • Secure 적용하면 https인 경우에만 전송

- <b>HttpOnly</b>
  • XSS 공격 방지
  • 자바스크립트에서는 접근 불가 (document,cookie)
  • HTTP 전송에만 사용

- <b>SameSite (비교적 최신 기술, 기능이 적용된지 얼마 안됐다)</b>
  • XSRF 공격 방지
  • 요청 도메인과 쿠키에 설정된 도메인이 같은 경우만 쿠키 사용
    (SameSite를 설정하지 않으면, 도메인이 같지 않은 경우에도 쿠키가 전송될 수 있기에, SameSite로 이를 방지)
</pre>
