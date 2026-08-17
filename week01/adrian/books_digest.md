## The DynamoDB Book (DeBrie), chapter 2

개념/용어 정의 챕터

- DynamoDB에서는 테이블 내 각 **item**(=record)이 다른 어트리뷰트들을 가질 수 있음 (RDB에서는 한 테이블의 모든 row가 같은 column들을 (schema) 가지는 것과 대조적)
- DynamoDB의 아이템 내 각 필드를 의미하는 **attribute**는 scalar(string, number, binary, null), complex(list, map), set type이 있음.
- 각 테이블은 primary key를 가짐. (DynamoDB는 schemaless이지만 primary key는 테이블 레벨에서 정의해 주어야 함) Primary key는 **simple** (*partition* key 하나) 또는 **composite** (*partition* key 하나와 그 아래 *sort* key 하나) 형태로 존재할 수 있음.
- DynamoDB에는 **secondary index** 개념이 있음. Local secondary index는 원래 table과 동일한 partition key 아래에 추가로 다른 sort key를 사용할 수 있게 한 것. **Global secondary index(GSI)**는 원래 table과 아예 다른 primary key로 테이블을 재배열한 것. GSI가 더 널리 쓰임.
- Composite primary key 테이블(또는 GSI)의 경우, 동일한 partition key 내의 item들을 item collection이라고 부름. DynamoDB의 액세스 패턴이 item collection 단위로 되도록 설계하게 됨.

## Amazon DynamoDB (Dhingra et al.), chapter 1~2

Chapter 1. DynamoDB 소개와 역사적 맥락

1. DynamoDB는 NoSQL DB 중 하나다.
2. RDB와 NoSQL DB의 차이
    전통적인 RDBMS는 데이터에 엄격한 스키마를 부여하고 잘 normalize해서 저장하며 테이블 간의 관계도 잘 정의한다. 즉 '강하게 구조화된 데이터베이스'라고 할 수 있다. 그래서 데이터의 duplication이 최소화되고 쿼리도 유연하게 구성할 수 있다(JOIN 등).
    반면 NoSQL DBMS는 엄격한 스키마를 DB 레벨에서 규정하지 않고, duplication도 허용한다. 그래서 스토리지 면에서는 덜 효율적이며 쿼리 패턴도 한정된 경우가 많다. 하지만 가능한 쿼리 패턴에 대해서 horizontally scalable하며 compute efficient 하도록 설계되어 있어서, 초거대규모 DB 운영 면에서 매우 유리하다.
3. DynamoDB의 특징
    - NoSQL 중에서 key-value 타입 DB
    - OLTP에 최적화되도록 설계. OLAP(집계/스캔) use case에는 불리하다(-> RDB 사용).
    - GSI(Global Secondary Index)를 통해 새로운 액세스 패턴에도 대응하도록 확장 가능

Chapter 2. Console과 SDK

1. AWS management console
2. AWS SDK
3. AWS Lambda

(Step-by-step guide 위주의 챕터라 내용 정리 생략)