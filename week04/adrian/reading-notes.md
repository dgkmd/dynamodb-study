# DeBrie 7/8장: 모델링, 단일테이블

## DDB의 denormalized 구조
- RDB의 특징: join, normalization
- 1NF, 2NF, 3NF 등의 normalization으로 데이터 중복이 없도록 테이블을 쪼개 저장하고, 쿼리시 JOIN으로 다시 이어붙여 사용
- normalization의 장점: 저장 용량 절약, data consistency 유지에 용이
- 하지만 그런 RDB의 단점: scalablility에 불리, compute를 비교적 많이 소모함
- DynamoDB는 데이터를 denormalized 형태로 저장함.
- 즉 중복 저장을 허용하고, dependency가 있는 필드라도 테이블 분리 없이 단일 테이블에 몰아서 저장
- 이런 DynamoDB의 가장 큰 장점은 scalable하다는 것

## DDB 설계
- 데이터 표현 (ERD)
- 액세스 패턴 파악. 접근 방식은 API 기준 또는 UI 기준 등

## 단일 테이블 디자인
- RDB와 다르게, 서로 다른 종류의 데이터를 하나의 테이블에 넣는 패턴 (primary key를 제외한 attribute들이 자유롭기 때문에 가능)
- 데이터 타입을 키의 prefix로 두는 방식이 일반적. (예: USER#123, ORDER#456 등)
- 단일테이블의 가장 큰 장점은 단일 요청으로 필요한 모든 데이터를 받아와 지연시간 등 성능 면에서 이점을 본다는 것이다.
- 단점은 러닝커브가 있고, 액세스 패턴에 유연하지 못하다는 것.
- 따라서 액세스 패턴이 자주 바뀌는 초기 앱은 싱글테이블 디자인을 꼭 쓸 필요가 없음. (책에서는 GraphQL과의 조합도 비추천함)

