# chapter 7 : DDB 데이터 모델링 접근법

## RDB와의 차이점

### Joins
- RDB는 다른 테이블 간 데이터 연결을 위해 join을 사용
- Join은 CPU 집약적 연산 <- 과거에는 스토리지가 더 비쌌던 시대상 반영
- Join은 데이터를 한곳에 위치해야할 필요성 증대 <-> 분산 데이터 시스템
- **DDB에는 join이 없음** -> 하나 요청하고 후속 요청 보내는데 이런 패턴 피해야함

### Nomalization
- 이상현상과 데이터 중복을 없애기 위한 과정
- 1차(속성 원자성), 2차(부분 종속), 3차(이행 종속) 제거 -> 여러 테이블을 지니게 됨
- 장점
  - 중복을 제거하여 스토리지 비용을 아낌
  - 데이터 정규성을 지켜 수정, 삽입, 삭제 이상을 줄임
- DDB에서는 왜 비정규화를 하나
  - 스토리지가 더 싸지고 CPU가 더 비싸진 현재
  - 데이터 정규성은 이제 DB보다 애플리케이션의 관심사가 되었음 -> 최종적 일관성으로도 충분한 도메인의 등장, 함께 읽는 데이터 경계도 App으로 부터 규정됨
  
### 테이블 별 복수 엔티티 타입
- Single Table에는 여러 엔티티 존재 ex) 한 요청에 Customer, CustomerOrders가 함께 포함되게 설계
- 1) PK가 엔티티 타입에 따라 다를 수 있음 -> CustomerId, OrderId가 한 테이블에 PK로 존재 가능
- 2) 레코드 속성값이 일률적이지 않고 각 아이템마다 다름

### 필터링
- RDB WHERE 절 -> 다양한 접근패턴을 커버하지만 많은 레코드를 읽을 필요 있음
- DDB Filtering -> 유연성을 희생하고 성능/확장성을 택함

---

## DDB Modeling Steps
- 애플리케이션 이해하기
- ERD 작성
- 접근패턴 이해
  - 실제 테이블 이전에 모든 접근패턴을 이해하는 것이 중요
  - 1) API 중심으로 접근 경계 이해하기
  - 2) UI 중심으로 접근 경계 이해하기
- PK 구조 설계
  - "읽는 시점에 클라이언트가 알아야 할 것을 고려하기" -> 변동 가능성이 적은 것을 PK로 잡는 것이 좋음
  - "엔티티 타입 간에는 prefix를 통해 구분"
- 추가적인 접근패턴은 GSI와 Stream으로 충족

---

# Chatper 8. SingleTable Design

## SingleTable Design이란
- RDB는 복합정보를 얻기 위해 Join이 필수 -> DDB는 Join이 없음 
- DDB를 RDB처럼 디자인하는 것은 큰 손해 -> Customer Fetch + Orders For Users Fetch
  - network i/o가 더 느려짐
  - 하나의 요청에서 여러 요청들을 조합해야함 -> 앱이 커질수록 점점 응답속도가 느려짐
- 미리 관련있는 아이템들을 조인해놓자 -> 한번 요청만으로 읽기 경계에서 관련 엔티티들을 모두 로드

## Single Table Design의 장점

- 하나의 엑세스 패턴에서 요청 수 및 연산 수를 줄일 수 있음
- 여러 테이블을 관리하면서 드는 운영 리소스를 줄일 수 있음 -> 하나의 쿼리 == 하나의 읽기 패턴이므로 알림 수도 줄음
- 돈을 아낄 수 있음 -> provision r/w를 하면 각 테이블당 비용 소모

## Single Table Design이 단점

- 높은 학습비용
- 새로운 액세스 패턴 추가의 어려움
  - PK로 읽기 경계가 강제됨 -> GSI등을 통해 새로 읽기 기준을 추가해야 함.
- 분석용으로 쓰기 힘듦(count, filter 등등)

## Single Table Design을 쓰지 말아야 할 때
- 성능보다 개발 속도, 분석이 중요할 때
  - DDB는 대용량 환경에서 일관된 성능을 보장하는 것을 목표로 삼음
  - 초기 그린필드 프로젝트에서는 아직 성능이 그리 중요하지 않을수 있음 
- GraphQL 쓸 때 : GraphQL 동작원리가 single table design하기 힘듦
  - Resolver가 각 엔티티별로 Fetch & Resolve를 진행하므로 단일 fetch를 지향하는 DDB 철학과 충돌됨
  - User Table Resolver , Order Table Resolver가 별도 존재
  - 고객의 주문 목록을 함께 조회하는 query를 발송할 때 -> user resolve -> user 결과 기반 orders resolver로 안티패턴을 추종하게 됨
  - 


