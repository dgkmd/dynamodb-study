# 관계형 사고에서 멀어지기

## 전자상거래 예시

### 3가지 접근 패턴

DynamoDB 단일 테이블 설계(single-table design)로 위 세 가지 접근 패턴을 모두 만족하는 예시를 만들어봤어. 그대로 복사해서 쓰면 돼.

## `Orders` 테이블

| PK (orderId) | SK          | type    | itemName / 필드                        | qty | price   | status    |
|--------------|-------------|---------|----------------------------------------|-----|---------|-----------|
| `order#1001` | `c#detail`  | order   | orderedAt=2026-09-15, channel=web      |     |         | PAID      |
| `order#1001` | `i#2001`    | invoice | invoiceNo=INV-2001, issuedAt=2026-09-15|     | 45000   | ISSUED    |
| `order#1001` | `p#0001`    | product | 무선 마우스                             | 1   | 25000   |           |
| `order#1001` | `p#0002`    | product | USB-C 케이블                           | 2   | 10000   |           |
| `order#1002` | `c#detail`  | order   | orderedAt=2026-09-14, channel=app      |     |         | SHIPPED   |
| `order#1002` | `i#2002`    | invoice | invoiceNo=INV-2002, issuedAt=2026-09-14|     | 88000   | ISSUED    |
| `order#1002` | `p#0001`    | product | 기계식 키보드                          | 1   | 88000   |           |

### 접근 패턴 매핑

| 요구사항 | Query 조건 |
|---------|-----------|
| 주문의 제품들 모아보기 | `PK = order#1001 AND begins_with(SK, "p#")` |
| 주문의 송장 보기 | `PK = order#1001 AND begins_with(SK, "i#")` |
| 주문의 세부정보 보기 | `PK = order#1001 AND begins_with(SK, "c#")` |
| 주문 전체(제품+송장+세부) 한 번에 | `PK = order#1001` (SK 조건 없음) |

### 설계 포인트
- 같은 `orderId`(PK) 아래 성격이 다른 아이템들을 **SK 접두사(`c#`, `i#`, `p#`)로 구분**해서, 하나의 Query로 필요한 것만 골라옴.
- SK 접두사를 알파벳/의미 순으로 정렬되게 두면(`c#` → `i#` → `p#`), PK만으로 조회할 때 세부정보 → 송장 → 제품 순으로 정렬돼 나와서 다루기 편함
- 주문, 제품, 송장 간의 관계가 테이블에 그대로 유지되며 JOIN 연산도 필요 없어짐
- 비정규화를 통해 Join 없이도 Product 세부정보도 조회 가능함

### 함계 접근하는 데이터는 반드시 함께 저장되어야 함
- 비정규화로 데이터 중복? -> 트랜잭션 API를 통해 여러 곳을 한번에 업데이트 필요함
- 


