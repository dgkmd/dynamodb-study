
## DynamoDB의 operation 세 가지
- Single item (singleton)
  - 대상 item의 정확한 primary key를 지정하여 read/write 작업
  - write operation은 main table에만 가능
- Query
  - composite key 테이블에서 하나의 partition key 내의 item들을 쿼리할 때 사용
  - 특히 sort key에 대한 범위 쿼리 (range query) 가능
- Scan
  - 테이블을 전수 스캔하는 작업
  - 일반적으로 자주 쓰이는 operation은 아님


## DynamoDB의 세 가지 데이터 타입
- Scalar (binary, string, numeric, boolean, null)
- Document ({list, map} of {scalar, document})
- Set (unordered and unique collection of scalars)
(지난 주 학습 내용에도 어느 정도 포함되어 있던 내용)

## DynamoDB 데이터 제약 (data limitations)
- Throughput 관련: on-demand 모드와 provisioned 모드 존재.
- 특정 reserved words를 attribute name으로 쓸 수는 있으나, 쿼리 시 placeholder 등의 workaround 필요
- 아이템 하나의 크기는 400 KB 이하여야 함
- Query나 scan의 경우, 데이터가 많으면 paginate됨. 한 페이지의 최대 크기는 1MB.