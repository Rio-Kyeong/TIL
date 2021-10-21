# HTTP Header Cache
## 캐시와 조건부 요청
<pre>
<b>캐시가 없을 때</b>
- 클라이언트의 매 요청마다 서버측의 데이터가 변경되지 않아도 계속 네트워크를 통해 데이터를 다운받아야 한다.
- 요청마다 반복 : 클라이언트가 이미지를 요청한다 -> 서버가 클라이언트 요청에 대한 응답을 보낸다.

<b>캐시 적용할 때</b>
- 첫 요청 때 서버로 부터 받은 데이터를 캐시 저장소에 저장한다.
- 두번 째 요청부터는 <b>웹 브라우저는 네트워크를 타지 않고, 저장된 캐시를 사용해서 곧 바로 데이터를 조회</b>할 수 있다.
- <b>장점</b> : 통신량과 통신 시간을 절약할 수 있다.

  <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/Cache.PNG"/>
  - 이 때 리소스는 서버가 max-age로 지정한 시간만큼 캐시는 유효하다.
  - Cache-control: Max-age = 60
  - 캐시의 유효시간 60초이며 이후 부터는, 서버에서 새로 데이터를 받아와 캐시에 저장해야 한다 (캐시 갱신)
</pre>
## 검증 헤더 추가
<pre>
<b>첫 번째 요청 - 검증 헤더 추가</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/VerificationHeader1.PNG"/>
- 서버에서 <b>Header에 last-modified 검증 헤더 추가 후 응답</b>
- 데이터가 마지막에 수정된 시간 표시

<b>두 번째 요청 - 캐시 시간 초과</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/VerificationHeader2.PNG"/>
- 브라우저 캐시 저장소에 캐시가 유효 시간이 지나면 서버에 다시 요청을 한다.
- <b>요청 시 header에 if-modified-since : (날짜) 를 포함</b>해서 요청한다.

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/VerificationHeader2-1.PNG"/>
- 만약 웹 브라우저에서 요청받은 if-modified-since(데이터 최종 수정일)가 수정된 사항이 없으면
- 서버는 <b>상태코드 304 Not Modified + 헤더 메타 정보만 응답(바디 x)</b>
- 클라이언트는 서버가 보낸 응답 헤더 정보로 캐시의 메타 정보를 갱신
- 클라이언트는 브라우저 캐시 저장소의 캐시 데이터 재활용
- <b>수정이 있을 시 요청에 대한 모든 데이터를 응답</b>한다.

결과적으로, <b>네트워크 다운로드가 발생하지만, 용량 적은 헤더 정보만 다운로드(매우 실용적인 해결책)</b>

<b>if-Modified-Since : Last-modified 의 단점</b>
- 1초 미만 단위로 캐시 조정이 불가능
- 날짜 기반의 로직 사용
- 데이터 수정해서 날짜가 다르지만, 같은 데이터를 수정해서 데이터 결과가 똑같은 경우에도 재 업로드
- <b>서버에서 별도의 캐시 로직을 관리하지 못함</b>
  예) 스페이스나 주석처럼 영향이 없는 변경에서 캐시를 유지하고 싶은 경우
</pre>
## ETag (Entity Tag)
<pre>
<b>if-None-Match, if-Match :  ETag(Entity Tag) 값 사용</b>
- 캐시용 데이터에 임의의 고유한 버전 이름을 달아둔다.
  예) ETag" 'v1.0', ETag" 'v2.0'
- <b>데이터가 변경되면 이 이름을 바꾸어서 변경</b>함 (Hash를 다시 생성)
  예) ETag: 'aaaa' -> ETag: 'bbbb'

<b>ETag 동작 원리-결과</b>
- 캐시 시간 초과시 ETag만 보내서 같으면 304 Not Modified + 헤더 메타 정보만 응답(바디 x), 다르면 다시 네트워크를 탄다.
- <b>캐시 제어 로직을 서버에서 완전히 관리할 수 있다.</b>
- 클라이언트에서는 캐시 메커니즘을 모른다.
- 참고) Last-Modified를 사용 시 데이터를 수정 후 다시 되돌려 놓으면 데이터는 같으나 마지막 수정일이 다르기 때문에 네트워크를 탄다.
  하지만 ETag 값을 해쉬 값으로 사용한다면 해쉬 값은 동일하기 떄문에 동일하다고 본다.
</pre>
## 캐시 제어 헤더
<pre>
<b>Cache-Control : 캐시 제어</b>
- <b>max-age</b> : 캐시 유효 시간, 초 단위
- <b>no-cashe</b> : 데이터는 캐시해도 되지만, 항상 <b>원(origin) 서버에 검증하고 사용</b>
- <b>no-store</b> : 데이터에 민감한 정보가 있으므로 저장하면 안됨 - 메모리에서 사용하고 최대한 빨리 삭제
- <b>must-revalidate</b> : 캐시 만료 후 최초 조회시 <b>origin 서버에 검증하고 사용</b>
  -> <b>origin 서버 접근 실패 시 반드시 오류가 발생해야함(504, Gateway Timeout)</b>


<b>Pragma: 캐시 제어(하위 호환)</b>
- HTTP1.0 하위 호환

<b>Expires : 캐시 만료일 지정(하위 호환)</b>
- 캐시 만료일을 정확한 날짜로 지정
- 지금은 Cashe-Control: max-age 권장한다. -> 이게 사용되면 Expires는 무시

<b>검증 헤더와 조건부 요청 헤더</b>
- <b>검증 헤더 (Validator)</b>
  • 서버 -> 클라이언트
  • <b>ETag</b>: "v1.0", ETag: "asid93jkrh2l"
  • <b>Last-Modified</b>: Thu, 04 Jun 2020 07:19:24 GMT
- <b>조건부 요청 헤더</b>
  • 캐시 유효 시간 만료 후 클라이언트 -> 서버
  • If-Match, If-None-Match : <b>ETag 값 사용</b>
  • If-Modified-Since, If-Unmodified-Since : <b>Last-Modified 값 사용</b>
</pre>
## 프록시 캐시
<pre>
<b>프록시란?</b>
- 서버와 클라이언트의 양쪽 역할을 하는 중계 프로그램
- 클라이언트로부터의 요청을 서버에 전송하고, 서버로부터의 응답을 클라이언트에 전송
- <b>클라이언트 <-> 프록시 서버 <-> 오리진 서버(Origin Server, 리소스 본체를 가진 서버)</b>

<b>프록시 사용 방법 2가지</b>
- <b>캐싱 프록시 (Cashing Proxy)</b>
  • 프록시에 다시 똑같은 리소스에 요청이 온 경우, 원 서버로부터 리소스를 획득하는 것이 아니라 캐시를 응답으로 되돌려 주는 타입의 프로시
- <b>투명 프록시 (Transparent Proxy)</b>
  • 요청과 응답을 중계할 때 메시지 변경을 하지 않는 타입의 프록시, 반대로 메시지에 변경을 가하는 타입의 프록시를 비투과 프록시라고 한다.

<b>캐시 서버</b>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/ProxyServer.PNG"/>
- 캐시 서버는 <b>프록시 서버</b> 중 하나로 <b>캐싱 프록시</b>로 분류
- 장점은 같은 데이터를 몇 번이고 오리진 서버에 전송할 필요가 없다는 것
- 자주 사용되는 데이터를 proxy 캐시 서버에 배치
- 0.5초 걸려 미국 서버(원 서버)까지 가지 않고, 프록시 서버에서 0.1초만에 해결한다.
- Cache-Control: <b>public</b> : 응답이 public 캐시에 저장되어도 됨
- Cache-Control: <b>private</b> : 응답이 해당 사용자만을 위한 것 (private 캐시에 저장해야한다(기본 값))
</pre>
## 캐시 무효화
<pre>
<b>캐시 무효화</b>
- 캐시를 적용하지 않아도 웹 브라우저가 GET 방식인 경우 임의로 캐시하기도 한다.
- 중요한 정보나 민감 데이터를 다룰 경우 확실히 캐시를 무효화 설정해야 한다.
- <b>Cache-Control: no-cache, no-store, must-revalidate</b> -> 모두 적용 시 캐시 무효화

<b>no-cache와 must-revalidate의 차이점</b>
- <b>Cache-Control: no-cache</b>
  <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/no-cache.PNG"/>
  • (원 서버 접근 불가 시) 데이터가 만약 업데이트 됐더라도 프록시 캐시 서버에 남아있는 이전 버전의 데이터라도 보여준다.

- <b>Cache-Control: must-revalidate</b>
  <img src="https://github.com/RyuKyeongWoo/TIL/blob/main/HTTP/img/must-revalidate.PNG"/>
  • (원 서버 접근 불가 시) 옛날 데이터를 보여주지 않음. 확실하게 에러를 표시
</pre>
