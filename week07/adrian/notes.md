# DeBrie 12장: N:M 전략

- 다대다(N:M) 관계를 표현하는 방법은 RDB의 경우 중간에 join 테이블을 두고, 쿼리시 JOIN을 통하여 특정 엔티티에 매핑되는 상대 엔티티들을 얻을 수 있다.
- DynamoDB에서는 join이 불가능하므로 위 방법은 불가능하다. 사실 N:M 관계 표현은 DDB에서 까다로운 주제다.

## N:M 표현 전략 4가지

1. Shallow duplication
- 각 엔티티 내에, 해당 엔티티에 매핑되는 상대 아이템 목록을 어트리뷰트로 저장하는 것이다.
- 매핑의 개수가 한정적이고 immutable한 경우 적합

2. Adjacency list
- 각 엔티티를 나타내는 PK 내에, SK로 매핑되는 엔티티 목록을 저장하는 것이다.
- 예: PK: MOVIE#<movie_name>, SK: ACTOR#<actor_name>
- PK와 SK를 뒤집은 GSI를 만드는 식으로 반대 방향 매핑도 획득 가능

3. Materialized Graph
- 각 엔티티를 node, 엔티티 간의 관계를 edge로 표현한 그래프를 DDB 테이블로 옮기는 것이다.
- 예: PK: person_id, SK: 해당 person의 속성
- 복잡해서 자주 쓰이지는 않음

4. Normalization & Multiple Requests
- RDB처럼 normalize하고, 여러 번의 request를 통해 쿼리하는 방식
- 엔티티의 변경이 잦은 경우 이게 최선의 방법일 수도 있음.