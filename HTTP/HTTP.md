# HTTP
<pre>
<b>HTTP</b>
- <b>HyperText Transfer Protocol</b>
- 클라이언트와 서버가 정보를 교환할 때 사용하는 프로토콜
- HTML, TEXT, IMAGE, 음성, 영상, 파일, JSON, XML 등등 거의 모든 형태의 바이너리(이진법) 데이터 전송 가능

<b>HTTP 역사</b>
- HTTP/0.9 (GET만 지원 HTTP 헤더 X)
- <b>HTTP/1.1 (가장 많이 사용) -> TCP기반 프로토콜</b>
- HTTP/2 (성능 개선) -> TCP기반 프로토콜
- HTTP/3 (진행중: TCP 대신에 UDP사용, 성능 개선) -> UDP기반 프로토콜 
</pre>
## HTTP 특징
<pre>
<b>클라이언트 서버 구조</b>
- <b>Request / Response 구조</b>

1. 클라이언트는 서버에 요청(Request)을 보내고, 응답이 올 때까지 대기
2. 서버가 요청에 대한 결과를 만들어서 응답(Response)

- <b>클라이언트와 서버를 분리하면 클라이언트와 서버가 각각 독립적으로 진화할 수 있다.</b>
-> 비지니스 로직과 데이터 처리는 서버에서 집중, 클라이언트는 UI와 사용성에 집중하여 역할을 분리시킨다.
   예) 대용량 트래픽이 발생했을 때 클라이언트와 별개로 서버에서 아키텍쳐 번경만으로 해결할 수 있다.


<b>무상태(Stateless)</b>
- HTTP는 Stateless한 성격을 가진다.
- 서버가 클라이언트의 상태를 보존하지 않는다.
- 장점 : 서버 확장성 높음(스케일 아웃)
- 단점 : 클라이언트 상태를 보존하지 않아 클라이언트가 추가 데이터를 전송해야 한다.

<b>무상태(Stateless)</b>와 반대의 의미로 <b>상태유지(Stateful)</b>가 존재한다.

<b>상태유지(Stateful)</b>
- 클라이언트의 상태를 보존하여 다음 요청이 들어와도 클라이언트의 상태를 알 수 있다.
- 클라이언트의 상태를 보존하고 있는 서버 외에 다른 서버에 접속 했을 때 정보를 알 수 없다.

<b>Stateless한 개발을 지향하자</b>
- Stateless한 설계를 하면 대용량 트래픽이 발생하여도 서버를 늘려서 대응할 수 있다.
- <b>최대한 Stateless하게 가고 꼭 상태유지가 필요한 부분만 Stateful하게 설계한다.</b>


<b>비 연결성(Connectionless)</b>
- HTTP는 비 연결성을 가진다.
- 클라이언트의 요청을 받으면 서버는 응답을 주고 연결을 끊는다.
- 최소한의 자원으로 서버를 유지할 수 있다(효율성 증가)
- 일반적으로 초 단위의 이하의 빠른 속도로 응답

<b>비 연결성(Connectionless)의 단점</b>
- 클라이언트의 요청이 있을 때마다 TCP/IP 연결을 새로 맺어야 한다 - 3 way handshake 시간 추가
- 웹 브라우저로 사이트를 요청하면 HTML, Javascript, css, img 등등 수 많은 자원이 함께 다운로드 된다.
- 하나의 자원을 받을 때마다 연결을 새로 다시 해야 한다면 많은 시간이 걸릴 것이다.
- 지금은 <b>HTTP 지속 연결(Persistent Connections)</b>로 문제 해결

  <b>HTTP 초기 - 연결 종료를 반복하여 자원낭비</b>
  <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/Connectionless.PNG"/>

  <b>HTTP 현재 - 지속 연결(Persistent Connections)</b>
  - 정보를 모두 받을 때 까지 지속 연결을 통해서 데이터를 받는다.
  <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/PersistentConnections.PNG"/>


<b>단순함, 확장 가능</b>
- HTTP는 단순하며 확장 가능하다.
</pre>
## HTTP 메시지의 구조
<pre>
<b>HTTP 메시지의 구조</b>

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/HTTPMessage.PNG"/>

<b>요청 메시지</b>
----------------------------
<b>start-line(시작라인)</b>         GET /search?q=hello&hl=ko HTTP/1.1 -> <b>HTTP 메서드 /요청대상(절대경로?쿼리스트링), HTTP Version</b>
<b>header(헤더)</b>                 Host: www.google.com
<b>CRLF(공백라인)</b>
<b>message body</b>                 POST일 경우 body 작성, 물론 GET도 되지만 권장하지 않음
----------------------------

<b>응답 메시지</b>
----------------------------
<b>start-line(시작라인)</b>         HTTP/1.1 200 OK -> <b>HTTP Version, HTTP State Code</b>
<b>header(헤더)</b>                 Content-Type: text/html;charset=UTF-8 | Content-Length: 3423 | ...
<b>CRLF(공백라인)</b>
<b>message body</b>                 실제 전송할 데이터(HTML 문서 등)
----------------------------

<b>header의 용도</b>
- <b>HTTP 전송에 필요한 모든 부가정보가 포함</b>되어있다.
- 메시지 바디외에 필요한 모든 메타 데이터가 들어가 있다.
- 예를들어 메시지 바디의 내용, 바디의 크기, 압축, 인증, 요청 클라이언트 정보, 캐시 관리 정보 등등.. 필요시 임의의 헤더 추가도 가능
- 스펙 : <b>field-name":" OWS field-value OWS</b> (OWS:띄어쓰기 허용)

<b>massege body의 용도</b>
- <b>실제 전송할 데이터를 포함</b>(HTML 문서, 이미지 영상, JSON 등등 byte로 표현할 수 있는 모든 데이터 전송 가능)
</pre>
