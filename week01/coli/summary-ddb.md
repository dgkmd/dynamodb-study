# Chapter2. Core Concepts in DDB


## 기초 단어들
- 테이블
  - 특징1. 테이블 하나가 여러 엔티티 표현 가능
    - RDB는 테이블이 하나의 엔티티만 대변하고 다른 개념은 조인 필요
    - DDB는 하나의 테이블에 여러 엔티티 타입을 포함할 수 있음
  - 특징2. schema
    - RDB는 스키마의 영향이 강력하고 접근패턴에 강제된다
    - DDB는 schemaless : 모든 컬럼에 필수인 것도 아님

- 아이템 : row
- 속성
  - scalar : 단일값 - string, number, binary, boolean, null
  - complex : 여러 값을 유연하게 나타냄 - lists, maps
  - sets
- PK
  - Unique + Not Null
  - 이미 있는 PK 값일경우 overwrite가 기본 설정
- Secondary Indexes
  - 다른 접근 패턴을 유연하게 확장 가능
  - 인덱스의 PK 설정 가능

## PK & Secondary Indexes

### PK
 - Simple PK : single element
 - Composite PK : two elements - partition key + sort key

### Secondary Indexes

- Local Secondary Index
  - Key Schema : base table과 같은 파티션 키를 써야함
  - Creation time : 테이블이 생성될 때 같이 선언해주어야 함
  - Consistency : 최종적 일관성(default) + 강한 일관성
  - Provision : base table 비용에 포함됨


- Global Secondary Index
    - Key Schema : 다른 어떤 속성도 파티션 및 정렬키가 될 수 있음
    - Creation time : 테이블이 생성된 후 선언 가능
    - Consistency : 최종적 일관성 고정
    - Provision : 별도로 계산됨

## The Importance of item collections
- item collection : 같은 파티션 키를 공유하는 아이템 집합
- 파티셔닝에 유용함
  - 여러 노드들에 파티션키를 대상으로 파티셔닝을 수행함
- 아이템 컬렉션은 데이터 모델링에 유용함
  - 쿼리 API는 다수의 아이템을 하나의 item collection으로 반환함
  

