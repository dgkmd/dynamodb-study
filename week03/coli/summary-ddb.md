
# Key Condition Expressions (Read)
- Query API에서 원하는 데이터를 정의할 때 사용
- 파티션 키 조건 + 선택적으로 정렬 키 조건(>, =, <) 쓰임 -> 속성에 대한 조건 X

```text
-- MovieRoles 테이블에서 파티션 키 Actor = 나탈리 포트만 조회 + 정렬키(Title -> N이상)
sult = dynamodb.query(
    TableName='MovieRoles',
    KeyConditionExpression="#a = :a AND #m < :title",
    ExpressionAttributeNames={
        "#a": "Actor",
        "#m": "Movie"
    },
    ExpressionAttributeValues={
        ":a": { "S": "Natalie Portman" ":title": { "S": "N" }
    }
)
```

# Filter Expressions (Read)

- Query, Scan 둘다 사용 가능
- 어떤 속성에도 적용이 가능하다는 특징(꼭, 키값을 대상으로만 하지 않아도 됨)
- Read Item(1MB 이내 검증) -> Filter 적용 -> Return Items => 일단 해당되는 Item들을 읽고 필터 적용
- Query & Scan은 최대 1MB 데이터를 반환 가능 그리고 이건 필터 조건을 적용하기 이전에 적용됨
```text
result = dynamodb.query(
    TableName='MovieRoles',
    KeyConditionExpression="#actor = :actor",
    FilterExpression="#genre = :genre" <- 필터 조건
    ExpressionAttributeNames={
        "#actor": "Actor",
        "#genre": "Genre"
    },
    ExpressionAttributeValues={
        ":actor": { "S": "Tom Hanks" ":genre": { "S": "Drama" }    
    }
)
```

- 1) 응답 페이로드 사이즈를 줄일 때
  - 단일 응답에서 최대 1MB 전송 가능 -> 전송되는 데이터를 줄여 응답시간 단축 가능
- 2) 애플리케이션 필터링이 더 쉬워짐
  - filter를 통해 일부를 버릴 경우 DDB에 보내는 요청에서 처리하기 보다 더 쉬운 경우 있음
- 3) TTL 만료 검증
  - 48시간 내에 만료시키므로 방지하기 위해 만료되었어야 하는 필터 표현식 작성 필요

---

# 3 .Projection

- 원하는 속성만 가져올 수 있음 - 너무 큰 행이 있을 때 이를 제외하고 조회
- Year처럼 예약어가 있다면 AttributeNames에서 지정해주어야 함
- list나 map의 특정 속성만 추출도 가능함
- filter처럼 일단 읽은 다음에 프로젝션을 진행 -> 1MB limit도 읽고 나서 프로젝션하기 전에 바로 적용

- ex) 테이블에서 CoverImage 행만 제외하고 반환하는 예시
```text
esult = dynamodb.query(
    TableName='MovieRoles',
    KeyConditionExpression="#actor = :actor",
    ProjectionExpression: "#actor, #movie, #role, #year, #genre"
    ExpressionAttributeNames={
        "#actor": "Actor",
        "#movie": "Movie",
        "#role": "Role",
        "#year": "Year",
        "#genre": "Genre"
    },
    ExpressionAttributeValues={
        ":actor": { "S": "Tom Hanks" }
    }
)
```

---

# 4. Condition Expressions

- item을 변경하려는 시도에 특정 상태를 보장하는데 사용 (PutItem, UpdateItem, DeleteItem)
  - PutItem 시 -> 이미 존재하는 item에 대해 덮어쓰기 방지
  - UpdateItem -> balance가 0처럼 유효성 검증
  - DeleteItem -> 삭제 권한 검증 (Owner인가 등등)

- 종류
  - attribute_exists()
  - attribute_not_exists()
  - attribute_type() : attribute가 특정 타입임을 보증
  - begins_with()
  - contains() : string이 특정 substring 포함하거나, set이 특정 요소 포함하거나
  - size() : lists, maps, sets 등의 사이즈 검증

- PK 뿐만 아니라 어떤 attribute 든지 활용 가능 -> item-based action에서 활용

## attribute_not_exists -> OverWrite & Unique 검증
- 같은 이름의 유저가 없음을 검증하고 Put
```text
result = dynamodb.put_item(
    TableName='Users',
    Item={
        "CreatedAt": { "S": datetime.datetime.now().isoformat()},
        "Username": { "S": "bountyhunter1"}, 
        "Name": { "S": "Boba Fett" }
    },
    ConditionExpression: "attribute_not_exists(#username)"
    ExpressionAttributeNames={
        "#username": "Username"
    }
)
```

## size -> Limiting in progress items
- background 워커에 큐 사이즈 조절할 때 - 10개까지만 가능
```text
result = dynamodb.update_item(
    TableName='WorkQueue',
    Key={
        "PK": { "S": "Tracker" }
    },
    ConditionExpression: "size(#inprogress) <= 10",
    UpdateExpression="Add #inprogress :id"
    ExpressionAttributeNames={
        "#inprogress": "InProgress"
    },
    ExpressionAttributeValues={
          ":id": { "SS": [ <jobId> ] }
    },
   
    
```

## contains -> Admin에 포함된 유저만 구독 변경 가능할 때
```text
result = dynamodb.update_item(
    TableName='BillingDetails',
    Key={
        "PK": { "S": 'Amazon' }
    },
    ConditionExpression="contains(#a, :user)",
    UpdateExpression="Set #st :type",
    ExpressionAttributeNames={
        "#a": "Admins",
        "#st": "SubscriptionType"
    },
    ExpressionAttributeValues={
        ":user": { "S": 'Jeff Bezos' },
        ":type": { "S": 'Pro' }
    }
)
```

---

# Update Expressions
- 연산들
  - set : add or overwrite attribute
  - remove : 속성 or 요소 지우기
  - add : number 속성을 더하거나 set에 요소 산입
  - delete : set에서 요소 지우기

## REMOVE — 속성 자체 또는 리스트 요소를 지움

- 항목(item)에서 속성(attribute)을 통째로 없애거나, 리스트(List)의 특정 인덱스 요소를 제거할 때 사용
- 예시: `SET ... REMOVE Age, Colors[2]`
    - `Age` → 이 속성 자체가 항목에서 사라짐
    - `Colors[2]` → 리스트의 인덱스 2 요소가 제거되고 뒤 요소들이 앞으로 당겨짐
- 대상: **속성 전체** 또는 **리스트의 원소**

## DELETE — 오직 Set 타입에서 요소를 지움

- **Set(집합) 타입 속성에서만** 사용 가능, 집합 안의 특정 원소(들)를 빼낼 때 사용
- 예시: `DELETE Colors :c`
    - `:c`가 `{"Red", "Blue"}` 같은 Set이면, `Colors` 집합에서 `Red`와 `Blue`를 빼냄
    - 속성 자체는 남아 있고 내용물만 줄어듦

## Case1. Updateing or setting an attribute
- Users 테이블에 pictureurl 세팅하기
```text
result = dynamodb.update_item(
    TableName='Users',
    Key={
        "Username": { "S": "python_fan" }
    },
    UpdateExpression="SET #picture :url",
    ExpressionAttributeNames={
        "#picture": "ProfilePictureUrl"
    },
    ExpressionAttributeValues={
        ":url": { "S": <https://....> 
    }
)
```

## Case2. Delete Attribute
- 희소 인덱스 패턴에서도 자주 쓰임
```text
result = dynamodb.update_item(
    TableName='Users',
    Key={
        "Username": { "S": "python_fan" }
    },
    UpdateExpression="REMOVE #picture",
    ExpressionAttributeNames={
        "#picture": "ProfilePictureUrl"
    }
)

```

## Case3. Incrementing numeric value
- number 속성의 증감을 위해 사용
- ContactUsPage의 view 수를 증가
- 읽고 증가시키는게 아니라 바로 증가시킬 수 있음

```text
result = dynamodb.update_item(
    TableName='PageViews',
    Key={
        "Page": { "S": "ContactUsPage" }
    }
    UpdateExpression="SET #views = #views + :inc",
    ExpressionAttributeNames={
        "#views": "PageViews"
    },
    ExpressionAttributeValues={
        ":inc": { "N": "1" }
    }
)
```

## Case4. Ading a nested property
- PhoneNumbers라는 map의 MobileNumber를 키로 value를 저장
- SET Map.KEY VALUE 로 세팅
```text
result = dynamodb.update_item(
    TableName='Users',
    Key={
        "Username": { "S": "python_fan" }
    }
    UpdateExpression="SET #phone.#mobile :cell",
    ExpressionAttributeNames={
        "#phone": "PhoneNumbers",
        "#mobile": "MobileNumber"
    },
    ExpressionAttributeValues={
        ":cell": { "S": "+1-555-555-5555" }
    }
)
```

## Case5. Adding and removing from a set
- ADD Set Value -> 멱등한 연산
- 한번에 여러개의 요소를 추가할 수도 있음
```text
result = dynamodb.update_item(
    TableName="SaasApp",
    Key={ "PK": { "S": "Admins#<orgId>" },
    UpdateExpression="ADD #a :user",
    ExpressionAttributeNames={
        "#a": "Admins"
    },
    ExpressionAttributeValues={
        ":user": { "SS": ["an_admin_user", "another_user"] 
    }
)
```

- REMOVE SET VALUE
```text
result = dynamodb.update_item(
    TableName="SaasApp",
    Key={ "PK": { "S": "Admins#<orgId>" },
    UpdateExpression="REMOVE #a :user",
    ExpressionAttributeNames={
        "#a": "Admins"
    },
    ExpressionAttributeValues={
        ":user": { "SS": ["an_admin_user"] 
    }
)
```
