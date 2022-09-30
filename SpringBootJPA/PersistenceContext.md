# Persistence Context
## 영속성 컨텍스트
<pre>
<b>영속성 컨텍스트(Persistence Context) : 엔티티를 영구 저장하는 환경</b>
- 영속성 컨텍스트는 논리적인 개념
- 영속성 컨텍스트 안에는 1차 캐시가 존재
- 엔티티 매니저를 통해서 영속성 컨텍스트에 접근
- J2SE(Java SE) 환경은 엔티티 매니저와 영속성 컨텍스트가 1:1 관계
- J2EE(Java EE) 환경은 엔티티 매니저와 영속성 컨텍스트가 N:1 관계
</pre>
## 엔티티의 생명 주기
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/EntityLifeCycle.PNG"/>
- <b>비영속(new/transient)</b> : 영속성 컨텍스트와 전혀 관계가 없는 <b>새로운</b> 상태(순수한 객체 상태)
    * 엔티티 객체를 생성 했으나, 저장하지 않은 상태
    * Member member = new Member();

- <b>영속(managed)</b> : 영속성 컨텍스트에 <b>관리</b>되는 상태(저장된 상태)
    * 엔티티 매니저를 통해서 엔티티를 영속성 컨텍스트에 저장한 상태
    * em.persist(member);
    * em.find(Member.class, "member") : 최초 1회 DB 조회 후 1차 캐시에 저장 (영속)

- <b>준영속(detached)</b> : 영속성 컨텍스트에 저장되었다가 <b>분리</b>된 상태
    * em.detach(entity) : 엔티티를 분리해 준영속 상태로 전환
    * em.clear() : 양속성 컨텍스트를 완전히 초기화
    * em.close() : 영속성 컨텍스트 종료
    
- <b>삭제(removed)</b> : <b>삭제</b>된 상태
    * 엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제
    * em.remove(member);
</pre>
## 플러시(flush)
<pre>
<b>플러시(flush) : 영속성 컨텍스트의 변경내용을 데이터베이스에 반영해준다.</b>
- em.flush() - 직접 호출
- transaction.commit() - 플러시 자동 호출
- JPQL Query 실행 - 플러시 자동 호출

<b>플러시가 발생하면?</b>
- 변경 감지(Dirty Checking)가 일어난다.
- 수정된 엔티티 쓰기 지연 SQL 저장소에 등록한다.
- 쓰기 지연 SQL 저장소의 (등록, 수정, 삭제)쿼리를 데이터베이스에 전송한다.

<b>플러시는!</b>
- 영속성 컨텍스트를 비우지 않는다.
- 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화
- 트랜잭션이라는 작업 단위가 중요 -> 커밋 직전에만 동기화 하면 된다.
</pre>
## 영속성 컨텍스트의 이점
### `1차 캐시 / 동일성(identity) 보장`
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/Cache.PNG"/>
- 영속성 컨텍스트는 내부에 캐시를 가지고 있으며, 1차 캐시는 (id, instance)의 맵 형태를 갖고 엔티티들이 저장된다.
- 트랜잭션 단위의 짧은 메모리 공간이다(<b>데이터베이스 한 트랜잭션 안에서만 존재, 트랜잭션이 종료되면 삭제</b>)

// 엔티티를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("user1");

// 1차 캐시에 저장(엔티티를 영속)
em.persist(member)

// 1차 캐시에서 조회된다
Member findMember = em.find(Member.class, "member1");

<b>em.persist</b>
1. 영속성 컨텍스트에 영속되면, 1차 캐시는 이를 담는다.
<b>em.find</b>
2. 이후 조회(find) 시, DB에 접근해서 탐색하는 것이 아닌 1차 캐시를 먼저 훝어 찾는 값이 있는지 확인 하고 캐시에 있다면 그 정보를 조회한다.
3. 캐시에 정보가 없다면 DB에서 검색 후 해당 객체를 1차 캐시에 저장하고 반환한다. - <b>영속상태</b>

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/EntityIdentity.PNG"/>
- 이런 1차 캐시를 거친 조회로 <b>엔티티의 동일성 보장</b>이 가능한 것이다.
</pre>
### `트랜잭션을 지원하는 쓰기 지연(transactional write-behind)`
<pre>
- 데이터 변경 시 바로 DB로 Query를 보낼 거 같지만 보내지 않는다.
- SQL Query를 바로 전송할 수도, 나중으로 지연 시킬 수도 있다.

//엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.
transaction.begin(); // [트랜잭션] 시작

em.persist(memberA);
em.persist(memberB);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

//커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
transaction.commit(); // [트랜잭션] 커밋

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/transactional_write-behind.PNG"/>
- 데이터 변경이 일어나게 되면 1차 캐시에 저장과 동시에 <b>SQL을 생성해서 쓰기 지연 SQL 저장소에 차곡차곡 쌓아둔다.</b>
- 이후에, <b>transaction을 commit</b>하거나, <b>컨텍스트에 버퍼를 비우도록 명령하면(flush)</b> 쓰기 지연 SQL 저장소의 쿼리가 DB에 넘어간다.
</pre>
### `변경 감지(Dirty Checking)`
<pre>
- 데이터를 찾아온(find) 후에 데이터를 변경(update)을 하면 자동으로 반경을 감지하여 Update Query를 생성해준다.
- 데이터를 찾아온(find) 후에 데이터를 삭제(delete)을 하면 자동으로 반경을 감지하여 Delete Query를 생성해준다.
- 트랜잭션을 커밋할 때 자동으로 수정되므로 별도의 수정 메서드를 호출할 필요가 없고 그런 메서드도 없다.

<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA/img/DirtyChecking.PNG"/>
- flush가 호출되면 1차 캐시에서 <b>변경된 엔티티</b>와 <b>스냅샷(값을 읽어온 최초 상태)</b>을 비교한다.
- 비교 값이 다르면 Update, Delete SQL을 생성하고 DB에 반영한다.
</pre>
