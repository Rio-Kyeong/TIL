# REST
## Web Service & Web Application
```
Web Service : 네트워크 상에서 서로 다른 종류의 컴퓨터들 간에 상호작용하기 위한 소프트웨어 시스템

Web Application : 인터넷으로 연결된 웹 환경에서 사용자들간의 연결을 통해 서비스를 제공하고 제공받는 어플리케이션
 
- 사용자(Client)가 웹브라우저의 주소창이나 하이퍼링크 등을 이용해 서비스를 요청(Request)하면 서버쪽에서 그 요청 정보를
  처리하여 결과를 특정한 형태로 사용자의 웹 브라우저에 응답(Response)한다.
- 웹메일, 뱅킹등이 있다.

네트워크상에서 클라이언트와 서버사이의 통신방식으로는 SOAP와 REST가 존재한다.
이런 웹 서비스를 개발하기 위해서 SOAP 또는 REST를 사용할 수 있다.
```
## SOAP(Simple Object Access Protocol)
<pre>
<b>SOAP</b> : HTTP, HTTPS, SMTP 등의 프로토콜을 이용해서 XML 기반의 메시지를 컴퓨터 네트워크상에서 전달할 수 있는 시스템

- SOAP Messege의 구조

    <b>SOAP 엔벨로프(Envelope)</b>
    모든 SOAP 메시지의 루트 요소이며 두 개의 하위 요소인 선택적 Header 요소 및 필수 Body 요소를 포함한다.
    
    <b>SOAP 헤더(Header)</b>
    SOAP 엔벨로프의 선택적 하위 요소이며 메시지 경로를 따라 SOAP 노드로만 처리될 애플리케이션 관련 정보를 전달하는 데 사용된다.
    
    <b>SOAP 본문(Body)</b>
    SOAP 엔벨로프의 필수 하위 요소이며 메시지의 최종 수신인을 대상으로 하는 정보를 포함한다.
    
    <b>SOAP 결함(Fault)</b>
    SOAP 본문의 하위 요소이며 오류 보고에 사용된다.
    
    
- SOAP은 복잡한 구조로 되어있고 거기에 따른 오버헤드가 심하고 개발하기가 어렵기 때문에 REST를 더 많이 사용한다.
</pre>
## REST(REpresentational State Transfer)
<pre>

<b>REST</b> : HTTP 통신에서 어떤 자원에 대한 CRUD 요청을 Resource와 Method로 표현하여 특정한 형태(json, xml)로 전달하는 방식

- REST의 구성요소

    <b>Resource(자원) - HTTP URI</b>
    서버는 Unique한 ID를 가지는 Resource를 가지고 있으며, 클라이언트는 이러한 Resource에 요청을 보낸다.
    이러한 Resource는 <b>URI</b>에 해당한다.
   
    <b>Method(행위) - HTTP Method</b>
    서버에 요청을 보내기 위한 방식으로 <b>GET, POST, PUT, PATCH, DELETE</b>가 있다.
    CRUD 연산 중에서 처리를 위한 연산에 맞는 Method를 사용하여 서버에 요청을 보내야 한다.
    
    <b>좋은 URI 설계에서는 리소스(Resource) 식별이 가장 중요하다.</b>
    또 <b>리소스</b>와 해당 리소스를 대상으로 하는 <b>행위</b>를 분리해야 한다.
    
    예를 들어 회원 조회 URI 를 만들 경우
    - <b>Method(GET, POST, PUT, DELETE 등) : 회원을 등록하고 수정하고 조회하는 행위(과정)</b>
    - <b>Resource : 회원이라는 개념 자체</b>
    
    URI는 리소스만으로 표현한다.
    GET /read-member-by-id X
    GET /members/{id} O 
    참고 : 계층 구조상 상위를 컬렉션으로 보고 복사단어 사용 권장(member -> members)

    Create(생성) - 데이터 생성(POST)
    Read(읽기) - 데이터 조회(GET)
    Update(갱신) - 데이터 수정(PUT or POST)
    Delete(삭제) - 데이터 삭제(DELETE)

    <b>Representation of Resource(자원의 형태) - HTTP Message Pay Load</b>
    클라이언트와 서버가 데이터를 주고받는 형태로 <b>json, xml, text, rss</b> 등이 있다.
    최근에는 Key, Value를 활용하는 json을 주로 사용한다.


- REST의 특징

    1. <b>Server-Client(서버-클라이언트 구조)</b>
        - REST Server - API를 제공하고, 제공된 API를 이용해서 비즈니스 로직 처리 및 저장을 책임진다.
        - Client - 사용자 인증이나 컨택스트(세션,로그인 정보)등을 직접 관리하고 책임진다.
        - 서로간의 의존성이 줄어들게 된다.

    2. <b>Stateless(무상태성)</b>
        - 사용자나 클라이언트의 컨택스트를 서버쪽에 유지 하지 않는다.
        - 세션이나 쿠키등을 별도로 관리하지 않기 때문에 API서버는 요청만을 들어오는 메시지로만 처리하기 때문에 구현이 단순하다.
        - REST는 Statelss한 성격을 가진 HTTP 프로토콜 상에 구현된 Resource Oriented Architecture (ROA) 설계 구조 이다.
      
    3. <b>Cacheable(캐시 처리 가능)</b>
        - HTTP 프로토콜 표준에서 사용하는 Last-Modified태그나 E-Tag를 이용하면 캐싱 구현이 가능하다.

    4. <b>Layered System(계층화)</b>
        - 클라이언트 입장에서는 REST ApI 서버만 호출한다.
        - REST 서버는 다중 계층으로 구성될 수 있다. 예를 들어 보안, 암호화, 사용자 인증등등 추가하여 구조상의 유연성을 줄 수 있다.

    5. <b>Uniform Interface(인터페이스 일관성)</b>
        - Uniform Interface는 URI로 지정한 리소스에 대한 조작을 통일되고 한정적인 인터페이스로 수행하는 아키텍처 스타일

    6. <b>Self-descriptiveness(자체 표현 구조)</b>
        - REST API 메시지만 보고도 이를 쉽게 이해 할 수 있는 자체 표현 구조로 되어 있다.


- REST의 장단점

    장점
     * HTTP 프로토콜의 인프라를 그대로 사용하므로 REST API 사용을 위한 별도의 인프라를 구출할 필요가 없다.
     * HTTP 프로토콜의 표준을 최대한 활용하여 여러 추가적인 장점을 함께 가져갈 수 있게 해 준다.
     * HTTP 표준 프로토콜에 따르는 모든 플랫폼에서 사용이 가능하다.
     * Hypermedia API의 기본을 충실히 지키면서 범용성을 보장한다.
     * REST API 메시지가 의도하는 바를 명확하게 나타내므로 의도하는 바를 쉽게 파악할 수 있다.
     * 여러 가지 서비스 디자인에서 생길 수 있는 문제를 최소화한다.
     * 서버와 클라이언트의 역할을 명확하게 분리한다.

    단점
     * 사용할 수 있는 메소드가 4가지밖에 없다.
     * HTTP Method 형태가 제한적이다.


- HTTP Status Codes(응답 코드)
    
    성공(Success)
     * 200 : 정상적 수행
     * 201 : 성공적으로 수행, 그 결과로 새로운 자원(Resource)이 생성됨

    클라이언트 에러(Client Error)
     * 400 : 잘못된 문법으로 인하여 서버가 요청을 이해할 수 없을 경우 응답 코드 
     * 401 : 클라이언트가 권한이 없는 자원(Resource)을 요청하였을 때 응답 코드
     * 403 : 보호되는 자원(Resource)을 요청하였을 때 응답 코드
     * 405 : 클라이언트가 사용 불가능한 Method를 이용했을 때 응답 코드

    기타
     * 301 : 클라이언트가 요청한 자원(Resource)에 대한 URI가 변경 되었을 때 응답 코드
     * 500 : 서버에 문제가 있을 때 사용되는 응답 코드

     자세한 내용은 이곳을 <a href="https://brunch.co.kr/@leedongins/65">참조</a> 

</pre>
<pre>
<b>REST API</b> : REST의 원리를 따르는 API

- REST API 설계 규칙
    1. 리소스(Resource)에 대한 행위는 HTTP Method(POST, GET, PUT, DELETE)로 표현해야한다.

    2. URI는 동사보다 명사를, 대문자보다는 소문자를 사용한다.

    3. 슬래시(/)로 계층 관계를 표현하며, URI의 마지막에는 슬래시를 포함하지 않는다.

    4. 언더바(_) 대신 하이픈(-)을 사용한다(띄어쓰기를 대체하고 가독성을 높일 수 있다)

    5. 파일확장자는 URI에 포함하지 않는다.

    6. HTTP Method (GET, PUT, POST, DELETE등등)의 행위가 URI 표현으로 들어가서는 안된다.
</pre>
<pre>
<b>RESTful</b> : REST API Service를 제공하는 Web Service

- REST API 설계 규칙을 따르는 시스템
- REST API의 설계 규칙을 올바르게 지킨 시스템을 RESTful하다 말할 수 있다.

- <b>self-descriptiveness</b> : RESTful API는 그 자체만으로도 API의 목적이 무엇인지 쉽게 알 수 있습니다.
</pre>
