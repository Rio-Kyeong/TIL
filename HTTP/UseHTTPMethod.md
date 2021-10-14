# Use HTTP Method
## 클라이언트에서 서버로 데이터 전송
<pre>
<b>클라이언트에서 서버로 데이터 전송 방식</b>
- <b>쿼리 파라미터</b>를 통한 데이터 전송(GET)
- <b>메시지 바디</b>를 통한 데이터 전송(POST, PUT, PATCH)

<b>클라이언트에서 서버로 데이터를 전송하는 4가지 상황</b>
- <b>정적 데이터 조회</b>
  - 이미지, 정적 텍스트 문서를 <b>쿼리 파라미터 없이</b> 리소스 경로로 단순하게 조회
  - GET /static/star.jpg

- <b>동적 데이터 조회</b>
  - 조회 조건을 줄여주는 필터, 조회 결과를 정렬하는 정렬 조건(검색어)에 주로 사용
  - <b>쿼리 파라미터를 사용</b>해서 데이터를 전달하여 조회
  - GET /search?q=hello&hl=ko

- <b>HTML Form 데이터 전송</b>
  - HTML Form 전송은 GET, POST만 지원
  - POST
    - <b>메시지 바디</b>를 통한 데이터 전송
    - <b>Content-Type : application/x-www-form-unlencoded</b> (default)
      • form의 내용을 메시지 바디를 통해서 전송(key=value, 쿼리 파라미터 형식)
      • Message Body:  username=kim&age=20
      • 전송 데이터를 url encoding 처리
    - <b>Content-Type : multipart/form-data</b>
      • Form 요소가 파일 업로드, 이미지 같은 바이너리 데이터를 전송 시 사용
      • 다른 종류의 여러 파일과 폼의 내용을 함께 전송 가능
  - GET
    - <b>쿼리 파라미터</b>를 통한 데이터 전송
    - /members?username=kim&age=20

- <b>HTTP API 데이터 전송</b>
  - 서버 to 서버
    • 백엔드 시스템 통신
  - 앱 클라이언트
    • 아이폰, 안드로이드
  - 웹 클라이언트
    • HTML에서 Form 전송 대신 자바 스크립트를 통한 통신에 사용(AJAX)
    • 예) React, VueJs 같은 웹 클라이언트와 API 통신
  - POST, PUT, PATCH : <b>메시지 바디를 통해 데이터 전송</b>
  - GET : <b>조회, 쿼리 파라미터로 데이터 전달</b>
  - <b>Content-Type : application/json</b>을 주로 사용 (사실상 표준)
    • TEXT, XML, JSON 등등
</pre>
## HTTP API 설계(자원 등록 방법)
<pre>
<b>신규 자원을 등록하는 방법, POST와 PUT</b>

<b>POST의 신규 자원 등록 특징</b>
- 서버가 새로 등록된 리소스 URI를 생성하고 관리
- POST /members
- <b>컬렉션(Collection)</b>
  - 서버가 관리하는 리소스 디렉토리
  - 여기서 컬렉션은 /members

<b>PUT의 신규 자원 등록 특징</b>
- 클라이언트가 리소스의 URI를 알고 관리
- PUT /members/100
- <b>스토어(Store)</b>
  - 클라이언트가 관리하는 리소스 저장소
  - 여기서 스토어는 /members

<b>HTML FORM 사용</b>
- HTML Form 전송은 GET, POST만 지원
- <b>컨트롤 URI</b>
  - GET, POST만 지원하므로 URI 표현에 제약이 있다.
  - 이런 제약을 해결하기 위해 동사로 된 리소스 경로 사용
  - POST의 /new, /edit, /delete가 컨트롤 URI
  - HTTP 메서드로 해결하기 애매한 경우 사용(HTTP API 포함)
</pre>
## URI 설계 개념
<pre>
- 문서(document)
  • 단일 개념(파일 하나, 객체 인스턴스, 데이터베이스 row)
  • 예) /members/100, /files/star.jpg

- 컬렉션(collection)
  • 서버가 관리하는 리소스 디렉터리
  • 서버가 리소스의 URI를 생성하고 관리
  • 예) /members

- 스토어(store)
  • 클라이언트가 관리하는 자원 저장소
  • 클라이언트가 리소스의 URI를 알고 관리
  • 예) /files

- 컨트롤러(controller), 컨트롤 URI
  • 문서, 컬렉션, 스토어로 해결하기 어려운 추가 프로세스 실행
  • 동사를 직접 사용
  • 예) /members/{id}/delete

<b>- 리소스만으로 URI를 설계하기 어려울 경우(문서와 컬렉션으로 해결이 어려울 경우) 컨트롤 URI를 사용</b>
- URI 설계시 참고 : <a href="https://restfulapi.net/resource-naming">https://restfulapi.net/resource-naming</a>
</pre>
