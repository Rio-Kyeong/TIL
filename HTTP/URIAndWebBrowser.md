# URI와 웹 브라우저 요청 흐름
## URI
<pre>
<b>URI</b>
- <b>인터넷에 있는 자원(Resource)을 나타내는 유일한 주소</b>
- 통합 자원 식별자(Uniform Resource Identifier)
  - <b>Uniform</b> : 리소스를 식별하는 통일된 방식
  - <b>Resource</b> : 자원, URI로 식별할 수 있는 모든 것(제한 없음)
  - <b>Identifier</b> : 다른 항목과 구분하는데 필요한 정보(식별)

- URI는 로케이터(locator), 이름(name) 또는 둘 다 추가로 분류될 수 있다.
  (URI의 하위개념으로 URL, URN 이 있다)
  -> <b>URL(locator)</b> : 자원(Resource)이 있는 위치 지정
  -> <b>URN(name)</b> : 리소스에 이름을 부여, EX) urn:isbn:8960777331
  -> 위치는 변할 수 있지만, 이름은 변하지 않는다.

<b>URI 전체문법</b>
<b>문법 : scheme://[userinfo@]host[:[port][/path][?query][#frgment]</b>
예제 : https://www.google.com:443/search?q=hello&hl=ko
명칭 : 프로토콜://호스트명:포트번호/패스?쿼리파라미터

<b>패스</b> - 리소스 경로, 계층적 구조 EX) /home/file1.jpg
<b>쿼리</b> - key=value 형태

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/URI.PNG"/>
</pre>
## 웹 브라우저 요청 흐름
<pre>
예) 구글에서 언어를 한글로 하여 hello를 검색한다.

<b>1. 웹 브라우저가 HTTP 요청 메시지 생성</b>

      <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/HttpRequestMessage.PNG"/>

<b>2. SOCKET 라이브러리를 통해 OS TCP/IP 계층에 메시지 전달</b>
    - TCP/IP 연결(IP, PORT) -> 3 way handshake 통해서 구글서버와 가상 연결

<b>3. OS TCP/IP 계층에서 TCP/IP 패킷 생성(HTTP 요청 메시지 포함)</b>
    - 출발지, 목적지 IP, PORT, 전송데이터를 포함한 TCP/IP 패킷을 생성하고 전달받은 HTTP 요청 메시지를 포함 시킨다.

<b>4. 서버에 TCP/IP 패킷 전송</b>
    - 가공된 TCP/IP 패킷은 여러 노드를 통해서 서버에 전달된다.
    - 서버는 받은 HTTP 메시지를 해석해서 HTTP 응답 메시지를 만들고 그 위에 TCP/IP 패킷을 씌워서 가공한다.
    
      <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/HttpResponseMessage.PNG"/>

<b>5. 웹 브라우저(클라이언트)에 HTTP 응답 메시지를 전달</b>
    - 웹 브라우저는 HTTP 응답 메시지의 데이터에 따라서 렌더링을 하고 결과를 보여준다.
</pre>
