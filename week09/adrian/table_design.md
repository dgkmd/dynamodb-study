
# 테이블 설계

## 엔티티 분석

문제에서 제시한 상황은 크게 아래 구성요소를 가진다고 볼 수 있을 것이다.

- user(유저 정보, 연락처, 주소, 카트)
- order(주문 명세, 주문 아이템)
- inventory(상품 목록, 재고)

이들을 단일 테이블로 설계해볼 것이다.

## Key 설계

### user 도메인
- 제약조건: username과 이메일 주소는 둘 다 unique해야 한다.
- PK/SK: USER#<username>/USER#<username>, EMAIL#<email>/EMAIL#<email>
  - 계정 생성 시 두 key 모두 존재하지 않음을 체크하며 transactional write로 두 아이템을 동시에 생성하도록 함.
- EMAIL#<email>/EMAIL#<email>은 어트리뷰트로 username만을 가짐. (중복 확인 용도)
- USER#<username>/USER#<username>은 프로필 기본 정보를 가짐.
- USER#<username>/{ADDRESS#<address>, CONTACT#<contact>}로 유저 부가정보를 저장.

### order 도메인
- PK/SK: ORDER#<orderId>/{ORDER#<orderId>, ITEM#<itemId>}.
- ORDER#<orderId>/ORDER#<orderId> 는 주문 정보(주문자, 주문 일시, 결제수단, 총 금액 등)를 어트리뷰트로 가짐.
- 주문자를 PK, 주문번호를 SK로 하는 GSI를 만들면 주문자별 주문 정보 목록을 모을 수 있음.
- 해당 주문의 아이템 목록 등 상세정보를 확인할 때는 메인 테이블의 ORDER#<orderId>/ITEM#... 등으로 쿼리.
- 카트는 CART#<username>/ITEM#<itemId> 로 저장. (CART/ITEM과 ORDER/ITEM은 어트리뷰트 구성이 다를 수 있음)

### inventory 도메인
- PK/SK: ITEM#<itemId>/ITEM#<itemId> 로 저장.
- 재고 현황 등을 위해서는 주문 체결 시 transactional write가 필요함. (쿼리 로직으로 보장)

## Walkthrough 답안의 차이점과 공통점
- 고객별 주소 수를 제한해서, 고객 아이템의 어트리뷰트 내에 주소 목록으로 저장하도록 했음.
- 주문 정보를 고객 PK 내에 저장하고, 주문 정보와 아이템 상세를 묶기 위해 GSI를 사용하였음. (나는 반대로 주문 정보를 ORDER PK로 저장하고, 고객과 주문을 묶기 위해 GSI를 사용하였음)
- 이메일을 별도의 아이템으로 만들어서 중복 검사에 사용한 것은 내 설계와 같았음.