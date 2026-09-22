
# 18. Session Store 만들기

## 요구사항 정리
- Create Session : Username & password 기반 Validate 필요 
- GetSession : TokenSession 기반 User 식별 필요
- Delete Session(TimeBased) : 7일이 지난 토큰은 전량 폐기
- Delete Session(Manual) : User 토큰을 차단 가능해야함

## 읽기 경계
- Create Session : Username 
- GetSession : TokenSession 
- Delete Session(TimeBased) : 토큰 생성 시간
- Delete Session(Manual) : User -> Tokens

## Data Modeling
- User : PK - user_name, Attribute: password 
  - 1번 토큰 생성 과정에서 유저 이름 기반 pwd 검증 가능
- Session : PK - session_value, Attribute : user_name, created_at, expired_at
  - session 기반 user 식별 가능
- GSI1 : PK: user, Attribute: Session
  - Manual Invocation에서 유저 대상 세션 무효화 시 적용

## TimeBased를 어떻게 처리할 것인가?
- 1안 : TTL 7일 설정 + 애플리케이션 검증
  - TTL 설정해도 최대 72시간 보존 가능하므로 애플리케이션에서 세션 접근 시 expired_at 한번 더 검증
  - 문제 
    - TTL로 삭제하는 것이 아니라 직접 접근해서 명시적 제거를 요구하는 것이라면?, 
    - 로그 추적용으로 7일 이후에도 세션데이터를 남겨놓길 원한다면?
  
- 2안 : Filter Expression
  - Session Store Scan하며 Fitler 처리 -> 근데 유저가 엄청 많으면 이거 힘듦..

- 3안 : Created_Date와 created_at으로 비즈니스적으로 어느정도 허용
  - GSI : PK : created_date(2026-07-20), SK : created_time
  - 매일 자정에 7일전 일자를 PK로 Scan 데이터 조회하여 expired_at이 지난 것들 조회 -> 개별 Delete하기
  - 문제는 1초 차이로 지워지지 않는 경우 다음 삭제텀인 자정까지 최대 24시간을 더 살 수 있음 -> 애플리케이션에서 조회하여 expired_at 지난 것들은 허용하지 않음

---

# Best Practice

## 3가지 질문
- 복합키 vs 단일키
- 관심사가 무엇인지
- 어떤 엔티티부터 모델링해야하는가?

- 간단한 엔티티일 수록 복합키 기반으로 grouping 하게 해줌
- 토큰은 무조건적인 Unique 필요
- Time based 삭제는 유저 액션에 의한 삭제가 아닌 비동기적으로 이루어지는 삭제
- User에 대한 삭제는 user기반으로 토큰을 그룹핑할 니즈가 있음

- 코어 엔티티부터 시작 User, Session
- User는 그룹핑 단위일 뿐이지 주요 관심사는 아니다

## Unique Token 요구사항
- PK설정 : Token Value를 PK로 삼으면 Unique가 보장됨
- condition expression : attirbute_not_exists를 통해 같은 세션이 없음을 보장하는 방식으로 넣을 수 있음

## Time Based Delete
- 1안 : pro active expiration : expire GSI를 index로 삼아 주기적으로 지우기
- 2안 : Lazy Expiration : 유저가 토큰을 넘겨받으면 expiration time 체킹하여 삭제
  - 유지보수해야할 작업이 없어지지만 죽은 토큰에 대한 스캔 비용 유지
  - 유저가 넘겨주지 않을 경우 계속 남아있음
- 3안 : TTL 옵션
  - 유지보수할 것은 줄이고 더 강력함
  - 48시간 지연이 있기 때문에 한번 애플리케이션 검증 or filter 검증이 필요

## Manual deletion
- GSI : PK를 Username으로 다른 attribute는 SessionToken으로
- Keys_Only를 통해서 pk만 옮기게 할 수 있음

## 다른 옴션들에는 뭐가 있을가?
- **KEYS_ONLY**: 인덱스 키 + base table 기본 키만 저장. 저장 공간·비용 최소, 다른 속성 필요 시 base table 재조회 필요.
- **INCLUDE**: KEYS_ONLY + 지정한 non-key 속성만 추가 투영(`NonKeyAttributes`). 저장 비용과 조회 편의성의 균형.
- **ALL**: 모든 속성 복제. 조회 시 base table 접근 불필요(가장 빠름), 저장·쓰기 비용 최대.

> 비용·저장량: `KEYS_ONLY < INCLUDE < ALL` / 조회 편의성은 반대 순서

