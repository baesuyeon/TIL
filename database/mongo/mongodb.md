### MongoDB 인덱스
MongoDB는 고정된 스키마는 없지만 원하는 필드를 인덱스로 지정하여 검색결과를 빠르게 조회할 수 있다.

MongoDB 인덱스는 B+Tree와 유사한 자료구조를 사용하여
정렬된 키를 바탕으로 range scan이 가능해 범위 조건 쿼리에도 효율적이다.

인덱스 리프 노드에는 인덱스 키와 Record Pointer (문서 위치 정보)가 있다 (MySQL InnoDB와 다름)
MongoDB에서는 모든 컬렉션에 대해 _id 필드에 대한 유니크 인덱스가 자동으로 생성된다.

### 범위연산
인덱스가 있다면 범위 연산도 효율적이다.

```
db.orders.find({ price: { $gte: 100000, $lte: 300000 } })
```
* 인덱스에서 price >= 100000의 시작 지점을 찾고
* 그 지점부터 price <= 300000까지 리프 노드를 순차 스캔

### MongoDB 커버링 인덱스
MongoDB에도 커버링 인덱스 개념이 있다.
쿼리가 인덱스에 포함된 필드만으로 처리되면 문서 본문을 읽지 않는다.

다음 조건을 모두 만족하면 커버링 인덱스이다.
* WHERE 조건에 사용한 필드가 인덱스에 있고
* SELECT 하는 필드도 인덱스에 있을 때
  * _id 포함 여부도 봐야함

인덱스 생성
```text
db.orders.createIndex({ userId: 1, price: 1 })
```

커버링 인덱스가 되는 쿼리
```text
db.orders.find(
  { userId: "u1", price: { $gte: 100000 } },
{   userId: 1, price: 1, _id: 0 }
)
```
* 조건 필드: userId, price → 인덱스에 있다.
* 조회 필드: userId, price → 인덱스에 있다.
→ 문서 본문(document) 디스크 접근 = 0

실행 계획
```text
db.orders.find(
  { userId: "u1" },
  { userId: 1, price: 1, _id: 0 }
).explain("executionStats")
```
IXSCAN → 컬렉션을 스캔(COLLSCAN)한 게 아니라 인덱스를 순회해서 조건을 만족하는 엔트리만 탐색했다는 뜻
totalDocsExamined: 0 → 본문을 하나도 열지 않았다
→ 문서 접근 없이 인덱스만 사용

커버링 인덱스가 안되는 쿼리
```text
db.orders.find(
  { userId: "u1" },
  { userId: 1 }
)
```
MongoDB에서는 기본적으로 _id 조회가 항상 포함된다.
_id는 해당 인덱스에 없기 때문에 문서 본문을 한번 더 읽는다.

### 복합 인덱스
a, b 필드로 이루어진 복합 인덱스가 있다면 a에 대한 단일 인덱스는 제거해도 된다.

```text
{ userId: 1, createdAt: -1 }
```
복합 인덱스에서 키의 순서는 매우 중요하다.
위 경우 인덱스 리프 노드 정렬이 아래와 같이 userId → createdAt 순서가 된다.
```text
(u1, t3)
(u1, t2)
(u1, t1)
(u2, t5)
(u2, t4)
```

이 인덱스로 가능한 것
* userId = ?
* userId = ? AND createdAt >= ?
* userId = ? ORDER BY createdAt DESC

불가능/비효율적인 것
* createdAt = ? (userId 조건 없음)
* ORDER BY createdAt 단독

## BSON이란?
JSON과 유사한 구조를 가진 MongoDB의 내부 바이너리 데이터 포맷이다.
JSON처럼 key-value 구조지만 텍스트가 아니라 바이너리로 문서를 저장해서 (필드명|타입|길이|값)
문자열을 파싱할 필요 없이 필요한 필드만 빠르게 접근할 수 있따.
