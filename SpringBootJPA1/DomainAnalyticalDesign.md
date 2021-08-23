# Domain Analytical Design
## 요구분석(Requirements)
<pre>
<b>기능 목록</b>

회원 기능
    - 회원 등록(Join)
    - 회원 조회
상품 기능
    - 상품 등록
    - 상품 수정
    - 상품 조회
주문 기능
    - 상품 주문
    - 주문 내역 조회
    - 주문 취소
기타 요구사항
    - 상품은 재고 관리가 필요하다.
    - 상품의 종류는 도서, 음반, 영화가 있다.
    - 상품을 카테고리로 구분할 수 있다.
    - 상품 주문시 배송 정보를 입력할 수 있다.
</pre>
## 도메인 모델과 테이블 설계
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA1/img/DomainModelAndTableDesign.PNG"/>

<b>회원, 주문, 상품의 관계</b> 
    - 회원은 여러 상품을 주문할 수 있다(1:N)
    - 한 번 주문할 때 여러 상품을 선택할 수 있으므로 주문과 상품은 다대다 관계다(N:M)
      하지만 이런 <b>다대다 관계는 관계형 데이터베이스는 물론이고 엔티티에서도 거의 사용하지 않는다.</b>
      따라서 그림처럼 주문상품이라는 엔티티를 추가해서 다대다 관계를 일대 다, 다대일 관계로 풀어냈다.

<b>상품 분류</b>
    - 상품은 도서, 음반, 영화로 구분되는데 상품이라는 공통 속성을 사용하므로 <b>상속 구조로 표현</b>했다.
</pre>
## 회원 엔티티(Entity) 분석
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA1/img/EntityAnalytical.PNG"/>

<b>회원(Member)</b> 
    - 이름과 임베디드 타입인 주소( Address ), 그리고 주문( orders ) 리스트를 가진다.

<b>주문(Order)</b> 
    - 한 번 주문시 여러 상품을 주문할 수 있으므로 주문과 주문상품( OrderItem )은 일대다 관계다(1:N)
    - 주문은 상품을 주문한 회원과 배송 정보, 주문 날짜, 주문 상태( status )를 가지고 있다.
    - 주문 상태는 열거형을 사용했는데 주문( ORDER ), 취소( CANCEL )을 표현할 수 있다.

<b>주문상품(OrderItem)</b> 
    - 주문한 상품 정보와 주문 금액( orderPrice ), 주문 수량( count ) 정보를 가지고있다.
      (보통 OrderLine , LineItem 으로 많이 표현한다.)

<b>상품(Item)</b> 
    - 이름, 가격, 재고수량( stockQuantity )을 가지고 있다.
    - 상품을 주문하면 재고수량이 줄어든다.
    - 상품의 종류로는 도서, 음반, 영화가 있는데 각각은 사용하는 속성이 조금씩 다르다.

<b>배송(Delivery)</b>
    - 주문시 하나의 배송 정보를 생성한다. 주문과 배송은 1:1 관계다.

<b>카테고리(Category)</b>
    - 상품과 다대다 관계를 맺는다.
    - parent , child 로 부모, 자식 카테고리를 연결한다.

<b>주소(Address)</b>
    - 값 타입(임베디드 타입)이다. 회원과 배송(Delivery)에서 사용한다.

<b>참고</b>
    - 회원 엔티티 분석 그림에서 Order(주문)와 Delivery(배송)가 단방향 관계로 잘못 그려져 있다. 양방향 관계가 맞다.
    - 회원이 주문을 하기 때문에, 회원이 주문리스트를 가지는 것은 얼핏 보면 잘 설계한 것 같지만, 객체 세상은 실제 세계와는 다르다.
      실무에서는 회원이 주문을 참조하지 않고, 주문이 회원을 참조하는 것으로 충분하다.
      여기서는 일대다, 다대일의 양방향 연관관계를 설명하기 위해서 추가했다.
</pre>
## 회원 테이블 분석
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBootJPA1/img/TableAnalytical.PNG">

<b>MEMBER</b>: 회원 엔티티의 Address 임베디드 타입 정보가 회원 테이블에 그대로 들어갔다. 이것은 DELIVERY 테이블도 마찬가지다.

<b>ITEM</b>: 앨범, 도서, 영화 타입을 통합해서 하나의 테이블로 만들었다. <b>DTYPE 컬럼으로 타입을 구분</b>한다.

<b>참고</b>
    - 테이블명이 ORDER 가 아니라 ORDERS 인 것은 데이터베이스가 order by 때문에 예약어로 잡고 있는 경우가 많다. 
      그래서 관례상 ORDERS 를 많이 사용한다.
    - 실제 코드에서는 DB에 <b>소문자 + _(언더스코어) 스타일</b>을 사용한다.

<b>연관관계 매핑 분석</b>

<b>회원(Member)과 주문(Orders)</b>
    - 일대다 , 다대일의 양방향 관계다.
      따라서 연관관계의 주인을 정해야 하는데, <b>외래 키가 있는 주문을 연관관계의 주인으로 정하는 것이 좋다.</b>
      그러므로 Order.member 를 ORDERS.MEMBER_ID 외래 키와 매핑한다.

<b>주문상품(Order_Item)과 주문(Orders)</b>
    - 다대일 양방향 관계다.
      <b>외래 키가 주문상품에 있으므로 주문상품이 연관관계의 주인이다.</b>
      그러므로 OrderItem.order 를 ORDER_ITEM.ORDER_ID 외래 키와 매핑한다.

<b>주문상품(Order_Item)과 상품(Item)</b>
    - 다대일 단방향 관계다.
      OrderItem.item 을 ORDER_ITEM.ITEM_ID 외래 키와 매핑한다.

<b>주문(Orders)과 배송(Delivery)</b>
    - 일대일 양방향 관계다.
      Order.delivery 를 ORDERS.DELIVERY_ID 외래 키와 매핑한다.

<b>카테고리(Category)와 상품(Item)</b>
    - @ManyToMany 를 사용해서 매핑한다.
      (<b>실무에서 @ManyToMany는 사용하지 말자. 여기서는 다대다 관계를 예제로 보여주기 위해 추가했을 뿐이다</b>)

<b>참고</b>
    - <b>외래 키가 있는 곳을 연관관계의 주인으로 정해라.</b>
      ex) 예를 들어서 자동차와 바퀴가 있으면,
          일대다 관계에서 항상 다쪽에 외래 키가 있으므로 외래 키가 있는 바퀴를 연관관계의 주인으로 정하면 된다.
</pre>