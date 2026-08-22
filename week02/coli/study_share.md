# TransactionWriteItems API
- 최대 100개 쓰기 작업 한번에
- 동일 리전, 동일 AWS rPwjd
- 트랜잭션 내 항목 총크기 4MB

## 동일 트랜잭션 내 동일한 항목을 여러 작업 대상으로 지정 못함
- Put, Update, Delete, ConditionCheck
- Why? 여러 작업을 순차로 실행하는 것이 아니라 함께 병렬 실행 -> 무엇을 먼저 실행할지 모름
- 트랜잭션 완료 -> 변경사항이 GSI, 스트림, 백업 전파

## TransactionGetItems는 언제쓸까
- 트랜잭션에서 수정된 항목의 원자 스냅샷을 보장하려면 TransactionGetItems로 모든 관련 항목을 함께 읽음
- 데이터에 대한 일관된 보기를 제공하므로 완료 트랜잭션의 모든 변경사항을 보거나 전혀 볼 수 없음

- GetItem/BatchGetItem은 각각 독립적인 읽기라 일관성 보장이 안됨

```text
내가 항목 A를 읽음        ← 트랜잭션 T의 변경 전 값
↑ 이 사이에 트랜잭션 T가 커밋됨
내가 항목 B를 읽음        ← 트랜잭션 T의 변경 후 값
```
- 결과적으로 A는 옛날 값, B는 새 값 — 트랜잭션이 반쯤 걸쳐진 찢어진(torn) 상태를 보게 됨
- TransactGetItems는 읽는 도중에 다른 트랜잭션이 중간에 끼어들 수 없어서 일관된 읽기가 가능 


## 멱등성
- CRT : 클라이언트 토큰으로 멱등성 확인 가능(10분 유효)
- 동일한 CRT여도 다른 요청 파라미터인 경우 IdempotentParameterMismatch 예외 반환
- 연결시간 초과 및 연결 문제로 중복 호출될 경우 애플리케이션 오류 방지 가능
- ReturnConsumeCapacity -> 소모된 WCU 값 반환, 후속 요청은 항목을 읽는데 사용된 RCU 값 반환

## 쓰기 오류 처리
- 조건식 중 하나의 조건이 충족되지 않는 경우
- 동일한 TransactWriteItems 작업 내 여러 작업에서 동일한 항목을 대상으로 지정
- 트랜잭션을 완료하기 위한 프로비저닝 용량이 부족한 경우
- 두개의 쓰기 트랜잭션 작업이 같은 항목을 동시에 건드리면 나중것은 실패함
- 트랜잭션에 의한 변경으로 인해 항목 크기가 지나치게 커지거나(400KB 초과), 로컬 보조 인덱스(LSI)가 너무 커지거나, 유사한 검증 오류가 발생하는 경우

---

# TransactionGetItems
- 최대 100개의 작업을 그룹화하는 동기식 읽기 작업
- 총 4MB를 초기화할 수 없음

## 읽기 오류 처리
- 트랜잭션 쓰기 작업 중에 여러 항목을 읽으면 일부는 새값, 일부는 옛날 값을 볼 수 있음
```text
트랜잭션 쓰기: 항목 1, 2, 3을 수정 중  ← 아직 진행 중
트랜잭션 읽기: 항목 3, 4를 읽으려 함
                    ↑
              항목 3이 겹침!
```
- TransactionGetItems가 같은 항목을 건드리는 TransactionWrtieItems와 겹치면 실패함
- 트랜잭션을 완료하기 위한 프로비저닝 용량이 부족한 경우
- 사용자 오류(예: 잘못된 데이터 형식)가 발생하는 경우

## 트랜잭션 겹침 오류
| 선행 (진행 중) | 후행 (새 요청)                                           | 취소 여부 | 이유 |
|---|-----------------------------------------------------|:---:|---|
| TransactWriteItems | TransactWriteItems, PutItem, UpdateItem, DeleteItem | ✅ 취소 | 같은 항목을 동시에 수정 → 원자성 훼손 방지 |
| TransactWriteItems | TransactGetItems                                    | ✅ 취소 | 수정 중인 항목을 읽음 → 찢어진 스냅샷 방지 |
| TransactGetItems | TransactWriteItems, PutItem, UpdateItem, DeleteItem |                                  | ✅ 취소 | 원자적 읽기 중 값 변경 → 읽기 일관성 보호 |
| TransactGetItems | TransactGetItems                                    | ❌ 정상 | 읽기끼리는 값을 바꾸지 않음 → 충돌 없음 |

---

# Transaction Isolation

## Serializable
- 여러 연산이 동시에 실행이 되어도 마치 그 결과가 하나가 완전히 끝난 다음에 다음것이 시작된 거서럼 보장

### 직렬성이 보장되는 3가지 조합
- 트랜잭션 연산 & 일반 쓰기(PutITem, UpdateItem, DeleteItem)
- 트랜잭션 연산 & 일반 읽기(GetItem)
- TransactWriteItem <-> TransactGetItems


## Read-Committed
- 커밋된 값만 본다 : 성공하지 못한 트랜잭션의 중간 상태는 절대 보지 않음
- 읽은 직후의 수정은 막지 못한다 : 읽은 순간 커밋된 값이었지만 그 값이 유지된다는 보장은 없음
- BatchGetItem, Query, Scan의 격리수준이 read-committed
```text
Query가 항목들을 순서대로 훑는 중:
  항목 1 읽음 (옛날 값)
  항목 2 읽음 (옛날 값)
     ↑ 이 사이에 트랜잭션 쓰기가 항목 3을 커밋
  항목 3 읽음 (새 값) ←  같은 Query인데 여기서부터 새 값!
  항목 4 읽음 (새 값)
```

---
# Transaction 용량
- 항목된 Prepare -> Commit으로 일반 읽기/쓰기를 2번 작업 => RCU,WCU도 2배 작업
- 트랜잭션이 실패해도 RCU/WCU는 소비됨

---

# Reference
- https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/transaction-example.html
