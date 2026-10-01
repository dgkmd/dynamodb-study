## my 설계
### main table
- PK: SessionToken
### GSI
- PK: Username
- SK: SessionToken

## 풀면서, 그리고 예시 답안 보고 나서 든 생각들
### 1
먼저 TTL을 사용하지 않는다고 가정하고 (책 용어로) pro-active expiration을 가정한 설계를 고민해봤지만 좋은 방법을 떠올릴 수 없었다. 일단 ExpiresAt을 sort 한 index가 있어야 테이블을 전수스캔하지 않고 expire된 토큰만 골라 삭제할 수 있는데, DDB에서는 sorted key로 하려면 SK에 넣어야 되니까... 그러면 PK를 뭘로 잡든 그 PK 값들을 다 알아야 쿼리 자체가 가능하다.

역시 TTL이 답인 것 같다.
### 2
나는 GSI도 primary key가 unique해야 한다고 생각하고 SK에 SessionToken을 끼워넣었는데, 예시 답안에서 GSI를 simple key(PK=Username)로 한 걸 보고 정보를 찾아보니 GSI의 primary key는 unique할 필요가 없는 거였다. 

그게 simple key든 composite key든!