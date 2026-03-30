## 인터페이스 vs 자료구조
* 인터페이스
인터페이스는 무엇을 할 수 있는가를 정의하는 사양으로 우리가 구현하고자 하는 연산들을 정의한다.
구체적인 구현 방식은 정의하지 않는다.

예를 들어 Set 인터페이스는 다음과 같은 연산을 정의한다.
* 중복을 허용하지 않는다.
* 특정 원소를 찾을 수 있다.
* 원소를 추가/삭제할 수 있다.

* 자료구조
자료구조는 위 인터페이스를 어떻게 구현할것인가에 대한 선택이다.
어떤 자료구조를 선택할지는 애플리케이션의 요구사항과 우선순위에 따라 달라진다

### 코틀린 표준 라이브러리의 Set 구현체 LinkedHashSet
코틀린에서 `setOf()`나 `mutableSetOf()`를 호출할 때 내부적으로는 `LinkedHashSet` 구현체를 사용된다.

* LinkedHashSet (코틀린 기본값)
  * HashMap + Doubly Linked List
  * 특징: 입력 순서 유지 (Insertion Order)
  * 시간 복잡도: Find / Insert / Delete → 평균 O(1)

```text
[ Hash Table (Buckets) ]          [ Doubly Linked List (Order) ]
      (데이터 위치)                      (데이터 간의 앞/뒤 연결)

Bucket [0] -> [ Node: Banana ] <----┐      head
Bucket [1] -> empty                 │       ↓
Bucket [2] -> [ Node: Apple  ] <----┼── [ Apple ]  (before: null, after: Banana)
Bucket [3] -> [ Node: Cherry ] <----┘       ↓
                                        [ Banana ] (before: Apple, after: Cherry)
                                            ↓
                                        [ Cherry ] (before: Banana, after: null)
                                            ↓
                                           tail
```
데이터(Key)를 넣으면 내부적으로 Hash Function을 거쳐 고유한 인덱스에 저장한다.
각 노드가 앞뒤를 가리키는 포인터(before, after)를 갖는다.
(그림에는 생략되었지만 충돌 방지 체이닝을 위한 next 포인터도 포함한다)

장점
* 삽입한 순서대로 조회할 수 있다. (예측 가능)
 * 순회 속도가 해시 테이블 크기가 아닌 실제 데이터 개수에 비례하기 때문에 빠르다.

단점
* before, after 포인터를 추가로 저장해야한다. (HashSet보다 1.5~2배 메모리 사용)
* 삽입/삭제시 pointer를 끊고 다시 이어줘야 한다.

### HashSet
순서 유지보다 오직 성능과 메모리 효율이 중요할 때 사용한다.
* 내부 구조: Hash Table
* 특징: 순서 보장 없음
* 시간 복잡도: 평균 O(1)

```text
[ Hash Table (Buckets) ]           [ 메모리 상의 실제 노드 객체들 ]
      (데이터 위치)                   (순서 없이 해시 결과에 따라 흩어져 있음)

      인덱스 (Index)
Bucket [0] -> empty
Bucket [1] -> [ Node: Banana | hash: 1 ]
Bucket [2] -> empty
Bucket [3] -> [ Node: Apple  | hash: 3 ] -> [ Node: Cherry | hash: 3 ] // 해시 충돌 발생(next 포인터를 이용해 데이터를 체이닝)
Bucket [4] -> empty
Bucket [5] -> empty
   ...
```
장점
* before, after 포인터(참조 변수)가 없기 때문에 데이터 건수가 많을수록 메모리를 아낄 수 있다.
단점
* 데이터가 어떤 순서로 나올지 예측할 수 없다.
* 데이터를 처음부터 끝까지 다 훑으려면 전체 해시 테이블 크기만큼 시간이 걸린다.

### TreeSet
```text
[ Root Node ]
      (7)
     /   \
  (3)     (10)  <-- 부모보다 작으면 왼쪽, 크면 오른쪽
  / \
(1) (5)

[ 실제 메모리 상의 노드 객체 구조 ]
Node {
    val key: 5
    var left: Node?  -> null
    var right: Node? -> null
    var parent: Node? -> (3)번 노드
    var color: Boolean -> RED or BLACK
}
```
해시 함수를 전혀 사용하지 않고 오직 데이터 간의 크기 비교(Comparison)를 통해 위치를 결정한다.

* 내부 구조: Red-Black Tree
* 특징: 자동 정렬을 유지한다.
* 시간 복잡도: Find / Insert / Delete → O(log n)
* Min/Max 검색이 빠르고 범위 탐색에 강하다. 
* 정렬 상태 유지가 중요한 경우 선택한다. (값이 5~10 사이인 데이터를 모두 가져와야할 때)

HashSet, LinkedHashSet, TreeSet 모두 Thread-safe 하지 않다.
여러 스레드가 하나의 셋을 동시에 수정하려고 할 때 데이터 손실 등 예측 불가능한 동작이 발생할 수 있다.
Set 내부의 연산(해시 계산 → 버킷 탐색 → 삽입)은 여러 단계로 이루어져 있는데 이 단계들 사이에 다른 스레드가 끼어들 수 있기 때문이다.
예를 들어 두 스레드가 동시에 같은 버킷에 원소를 넣으면 한쪽 데이터가 덮어써져서 사라질 수 있다.
