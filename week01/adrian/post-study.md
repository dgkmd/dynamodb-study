
- DynamoDB는 가능한 query pattern에 대해 scalable하지만, 가능한 query pattern이 한정되어 있다는 것 자체가 한계다.
- 그래서 사내에서는 특정 테이블에 대해 다양한 쿼리나 JOIN 등 복잡한 쿼리가 필요한 use case를 위해 memdb를 만든 걸로 보인다.
- memdb는 특정 DynamoDB 테이블 전체를 in-memory에 적재하고 쿼리를 수행한다. 단, 테이블의 SSOT는 underlying DynamoDB 테이블이다.