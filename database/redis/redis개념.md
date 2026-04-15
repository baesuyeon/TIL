> 참고: [Redis 공식 문서](https://redis.io/)

## Redis란 ✅

레디스(Redis)는 Remote Dictionary Server의 약자로서,
인메모리(In-memory) 데이터 저장소로 키-값(Key-Value) 구조의 비정형 데이터를 저장하고 관리하기 위한 오픈 소스 비관계형 데이터베이스 관리 시스템(DBMS)이다.
일반적인 관계형 데이터베이스와는 달리 데이터를 디스크가 아닌 메모리(RAM)에 저장하고 처리하여 뛰어난 속도를 자랑한다.

Redis의 가장 큰 특징

* 매우 빠르기 때문에 실시간 응답성과 높은 처리량이 요구되는 애플리케이션에서 자주 사용된다.
* 빠르지만 데이터가 날아가지 않게 보완도 가능하다
    * 메모리에서 처리된 데이터를 주기적으로 디스크에 동기화해 데이터 유실을 방지할 수 있다.

## Redis 싱글 스레드 (Single-Thread)

Redis의 클라이언트 명령을 처리하는 부분은 이벤트 루프 기반의 싱글 스레드로 동작한다.
여러 클라이언트가 동시에 요청을 보내도 Redis 내부에서 요청을 한 줄로 세워 순서대로 하나씩 처리한다.
1번 요청이 완전히 끝날 때까지 2번 요청을 시작하지 않기 때문에 Redis 명령어는 원자적이다.
(여러 단계로 구성된 경우 명령어들 중간에 다른 명령어 요청이 끼어들 수는 있다 → 루아 스크립트를 사용할 수 있다)
중간에 오래 걸리는 연산이 있다면 나머지 요청이 밀리기 때문에 주의해야 한다.

싱글 스레드로 동작하는데 어떻게 많은 요청을 빠르게 처리할 수 있을까?

* 메모리 기반으로 동작
* 키-값 모델(해시 테이블) → 조회 시간복잡도 O(1)
* 논블로킹 I/O + I/O multiflexing (하나의 스레드로 여러 client의 IO 처리 가능)
    * 전통적인 블로킹 I/O에서는 클라이언트가 요청을 보냄 → 서버가 소켓에서 데이터를 읽으려고 함 → 데이터가 아직 안 오면 → 기다림(블로킹)
    * 즉 하나의 요청 기다리느라 다른 요청을 볼 수 없다.
    * 논블로킹 I/O의 핵심은 “기다리지 않는다”
      * 논블로킹 I/O만 쓰면 busy waiting을 해야한다. (CPU 자원 소모) 
      * I/O multiflexing: 여러 클라이언트 소켓을 한 곳에 등록해 두고 ‘지금 당장 처리 가능한 소켓만’ 골라서 알려달라고 OS에 요청하는 방식
    * 지금 바로 처리 가능한 요청만 처리한다.
* 싱글 스레드로 컨텍스트 스위치를 하지 않으며, 동시성 제어를 위한 잠금이 필요없다.

## Redis Client

- redis-cli utility
    - Redis에서 제공하는 공식 터미널 클라이언트 도구
    - 직접 명령어를 입력하고 결과를 확인하는 데 사용
    - CLI 환경에서 빠르게 확인, 디버깅, 실시간 조회할 때 유용

- Redis client for your preferred programming language
    - 어플리케이션 코드에서 Redis에 접근하기 위해 사용하는 라이브러리
    - 사용하는 언어에 따라 다양한 클라이언트가 존재 (Java/Kotlin에서는 Lettuce, Jedis가 있다.)

## redis-cli

* localhost:6379접속

```bash
redis-cli
```

호스트명과 포트번호를 생략하면 localhost의 6379로 접속된다.

* 원격접속

```bash
redis-cli -h #{호스트명} -p #{포트번호}
```

## Redis Client 비교, Lettuce vs Jedis

| 구분          | **Lettuce**                       | **Jedis**                         |
|-------------|-----------------------------------|-----------------------------------|
| 네트워크 모델     | 비동기 / Non-blocking (Netty 기반))    | 동기 / Blocking                     |
| 스레드 안전성     | Thread-Safe (여러 스레드에서 공유 가능)      | Thread-Safe X                     |
| 커넥션 관리      | 단일 커넥션으로 다수 요청 처리 가능              | 스레드마다 커넥션 필요 (Connection Pool 필수) |
| 지원 API      | Sync, Async, Reactive (Flux/Mono) | X                                 |
| Reactive 지원 | O (Mono / Flux)                   | X                                 |
| 성능 (고부하)    | 높음 (동시성 처리에 강함)                   | 상대적으로 낮음 (멀티스레드 부하 시)             |
| 사용 난이도      | 약간 높음                             | 단순함                               |
| Spring Boot | 2.0부터 기본 클라이언트로 채택                | 별도 설정 필요                          |

### Lettuce
```text
애플리케이션 Thread 1 ─┐
애플리케이션 Thread 2 ─┼─▶ EventLoop (Netty) ─▶ Redis
애플리케이션 Thread 3 ─┘
```

여러 애플리케이션 스레드에서 발생한 Redis 요청을 
Lettuce 내부의 Netty I/O 스레드(EventLoop)가 모아서 네트워크 단에서 처리한다.

스레드들이 Netty라는 단일 통로(EventLoop)에 요청을 던져두고 바로 자기 할 일을 하러 간다. 
커넥션 하나로도 수많은 스레드의 요청을 큐(Queue)에 쌓아 순차적으로 처리할 수 있어 효율적이다.

* 애플리케이션 스레드는 Redis 응답을 기다리지 않는다.
* 요청은 EventLoop에 전달되고 바로 반환된다.
* 실제 소켓 I/O는 EventLoop가 담당한다.

### Jedis
```text
애플리케이션 Thread 1 → Connection 1 → Redis
애플리케이션 Thread 2 → Connection 2 → Redis
애플리케이션 Thread 3 → Connection 3 → Redis
```
Jedis는 전통적인 블로킹 I/O 방식이다.

1대1 전담 창구 방식으로 요청을 보내면 응답이 올 때까지 스레드가 기다려야(Blocking)한다. 
그래서 동시에 여러 요청을 처리하려면 그만큼 커넥션(창구)을 많이 늘려야 한다.

* Redis 명령을 보내면
* 응답이 올 때까지 해당 스레드가 차단(Blocking)된다.
* 하나의 커넥션은 동시에 여러 스레드가 사용할 수 없다

동시 요청이 늘어나면 애플리케이션 스레드 수와 Redis 커넥션 수가 함께 증가한다.

### 왜 Lettuce를 더 많이 쓸까?
(1) 비동기 기반의 고성능

Lettuce는 Netty 기반으로 비동기 방식으로 동작한다.

> 하나의 커넥션만으로도 여러 애플리케이션 스레드가 Redis 명령을 동시에 보낼 수 있다.

이 말의 의미는 여러 스레드가 하나의 TCP 커넥션에 요청을 등록하고 바로 반환할 수 있다는 뜻이다.
```text
애플리케이션 Thread 1 → 요청 등록 → 바로 반환
애플리케이션 Thread 2 → 요청 등록 → 바로 반환
애플리케이션 Thread 3 → 요청 등록 → 바로 반환
```
실제 전송과 응답 처리는 EventLoop가 담당한다.

반면 Jedis는

```text
Jedis
Thread-1 → Redis GET → (응답 올 때까지 대기)
Thread-2 → Redis GET → (대기)
```
하나의 커넥션을 동시에 사용할 수 없고 요청마다 스레드가 응답을 기다리며 Blocking된다.

(2) 효율적인 리소스 사용

Jedis는 Thread Safe 하지 않다.
* 하나의 Jedis 커넥션을 여러 스레드가 동시에 사용하면 안된다.
* JedisPool을 통해 커넥션을 미리 여러개 만들어두고 돌려써야 한다

이로 인해 
* 커넥션 수 증가 → 메모리 사용량 증가
* 커넥션 풀 크기 튜닝 필요 → 운영 복잡도 증가

(3) Reactive 프로그래밍 지원
Spring WebFlux와 같은 Reactive 환경에서는 Lettuce가 유일한 선택지이다.
* Lettuce: Async / Reactive API 지원
* Jedis: 동기 기반, Reactive 스트림 미지원

## 캐싱 전략 - Look-Aside

읽기 요청이 많은 경우 가장 일반적으로 사용하는 전략이다.

1. 캐시에서 데이터 조회
2. 캐시에 없으면 DB에서 데이터를 읽어온 뒤
3. Redis에 저장

장점
* 캐시 장애시에도 DB에서 데이터 조회 가능하다.

단점
* 캐시 장애 시 DB에 부하가 집중될 수 있다.
* 신규 데이터 초기에는 cache miss가 많이 발생할 수 있다.
  → 이를 완화하기 위해 cache warming을 사용할 수 있다. (미리 db에서 캐시로 데이터를 밀어넣어주는 작업)

## 캐싱 전략 - 쓰기 전략
Write: update, insert, delete를 포함한 모든 변경 작업

| 전략                | Write 시 동작           | 핵심 특징           |
| ----------------- | -------------------- | --------------- |
| **Write-Through** | DB + Cache 모두 업데이트   | 정합성 ↑, 쓰기 비용 ↑  |
| **Write-Behind**  | Cache 먼저, DB는 비동기    | 성능 ↑, 정합성 ↓     |
| **Write-Around**  | DB만 저장 + Cache evict | 단순함, read 부하 가능 |

* **Write-through (강한 정합성)**

데이터가 생성/수정될 때마다 DB와 캐시를 동시에 업데이트 한다.

주의할 점: write 성능 저하 (매번 두 번의 쓰기 작업)
언제쓰면 좋을까? 항상 최신 값이 캐시에 있어야 하는 경우

* **Write-Behind (Write-Back)**
```text
WRITE → Cache
        DB (비동기)
```
주의할 점: DB 반영이 지연되거나 캐시 장애 시 데이터 유실될 위험이 있다.
언제쓰면 좋을까? 약간의 데이터 유실이 허용되는 경우, 쓰기 빈도가 매우 높은 경우, 집계성 데이터

* **Write-Around**
```text
WRITE:
  1. DB 저장
  2. 관련 캐시 삭제 (invalidate)

READ:
  1. 캐시 조회
  2. 없으면 DB 조회 후 캐시 적재
```
주의할 점: write 직후 첫 read는 cache miss. 트래픽이 몰리면 순간적으로 DB 부하가 증가 할 수 있다.
언제쓰면 좋을까? 읽기가 많고, 쓰기는 상대적으로 적은 데이터

## Redis 주요 설정
* **MAXMEMORY-POLICY**

메모리가 가득 찼을 때 키를 어떻게 관리할지 결정한다.

* noeviction(default): 키를 삭제하지 않고 쓰기 실패한다.
* volatile-lru: 가장 최근에 사용하지 않았던 key부터 삭제 (expire 설정된 key 값 대상)
* allkeys-lru: 가장 최근에 사용하지 않았던 key부터 삭제 (모든 key 값 대상)

## 캐시 TTL
TTL이 없는 데이터는 메모리에 계속 쌓이게 되어 메모리 고갈(OOM) 현상을 야기할 수 있다.
많은 경우 데이터는 특정 시간 동안만 유효하다. (1시간 동안 유효한 인증번호 등)
TTL을 설정하면 이러한 복잡한 로직을 Redis에 위임하여 애플리케이션 코드를 단순하고 명료하게 유지할 수 있다.

캐시 TTL을 너무 짧게 설정하는 경우
key가 만료되는 순간 많은 서버에서 해당 key를 보고 있던 상황이라면
수많은 서버가 동시에 DB를 찔러 DB가 마비될 수 있다.
* TTL 시간을 넉넉하게 늘린다. (DB를 찌르는 빈도가 낮아진다)
* TTL에 난수(Jitter)를 추가하여 만료 시점을 분산시켜 DB 부하를 골고루 나눈다. (ex: 60분 + (0~5분 사이의 랜덤 값))

## Redis 주의할 점
(1) **하나의 키에 너무 큰 데이터를 저장하지 말 것 (Big Key 문제)**

**하나의 키에 지나치게 큰 데이터를 저장하면** 여러 문제가 발생할 수 있다.

일반적으로

* **1MB를 넘는 문자열 값**이나
* **수만개 이상의 요소를 가진 컬렉션**은

**사이즈가 큰 키(Big Key)** 로 간주한다.
이러한 Big Key는 **네트워크 사용량을 급격히 증가**시킨다.
Redis는 데이터를 네트워크를 통해 주고받기 때문에 크기가 1MB인 키를 초당 1,000번 조회한다면 초당 약 **1GB의 네트워크 트래픽**이 발생하게 된다.
Redis에서는 **자주 접근되는 데이터를 작게 쪼개어 저장하는 것**이 성능과 안정성 면에서 훨씬 안전하다.

(2) 특정 키에만 요청이 과도하게 몰리지 않도록 할 것 (Hot Key 문제)

**Hot Key**란 키의 크기와 상관없이 **짧은 시간 동안 특정 키에 요청이 집중되는 상황**을 의미한다.
모든 요청이 **같은 실행 큐**(싱글 스레드 이벤트 루프)를 공유하기 때문에 다른 키를 조회하는 요청도 뒤에서 계속 대기할 수 있다.
```text
[요청 A] → hot:key
[요청 B] → hot:key
[요청 C] → hot:key
[요청 D] → 다른:key  ← 이 요청도 같이 기다림
```
* 서버 내부 캐시를 함께 사용하거나
* 만료 시간을 분산시키는 등의 방식으로 (DB 동시 접근 회피)

**특정 키에 트래픽이 집중되지 않도록할 수 있다.** (키를 분산하는 것이 좋다)

## Keys and values

Redis 데이터베이스에 저장하는 모든 데이터 객체에는 고유한 키가 있다.
키는 해당 객체를 검색하거나 객체를 수정하기 위해 Redis 명령에 전달하는 "문자열"이다.
특정 키와 연결된 데이터 객체를 값 이라고 하며, 이 둘을 합쳐서 키-값 쌍 이라고 한다.

### Key

프로그래밍 언어의 변수 이름과 달리 Redis 키는 형식에 제한이 거의 없으므로 공백이나 구두점 문자가 포함된 키도 가능하다.

하지만 콜론(":") 문자를 사용하여 키를 여러 섹션으로 나누는 규칙이 있다. (ex: "person:1", "person:2", "office:London", "office:NewYork:1").
"user:1000"처럼 "object-type:id"처럼 사용하는 것이 좋다.
이 규칙을 사용하면 키를 여러 범주로 간편하게 모을 수 있다.
"comment:4321:reply.to" 또는 "comment:4321:reply-to"처럼 여러 단어로 구성된 필드에는 점이나 대시를 사용하는 경우가 많다.

[주의 할 점]

- 너무 긴 키는 피하는 것이 좋다. 키가 지나치게 길면 메모리 낭비가 발생할 수 있으므로 필요하다면 해싱하는 방법으로 축약하는 것이 좋다.
- 너무 짧은 키도 피하는 것이 좋다. 예를들어 "u1000flw"와 같은 축약형 대신 "user:1000:followers"처럼 의미가 명확한 구조화된 키를 사용하는 것이 바람직하다.
  이로 인해 발생하는 메모리 증가는 매우 작고, 가독성과 유지보수 측면에서 훨씬 유리하다.
- 허용되는 최대 키 크기는 512MB이다.

### Redis 주요 명령어

메모리 확인

```text
MEMORY USAGE <key>
```
Redis에서 특정 키 하나가 실제로 차지하고 있는 메모리 크기를 바이트 단위로 확인할 수 있는 명령어다.
`MEMORY USAGE` 는 단순히 value 크기만 보는 것이 아니라 아래 항목을 모두 포함해서 Redis가 실제로 할당한 메모리 크기를 알려준다.
* key 문자열 자체
* value 데이터
* Redis 내부 자료구조 오버헤드

```text
127.0.0.1:6379> memory usage 'key:all'
(integer) 2632
```
key:all 이라는 Redis 키가 약 2,632 bytes (≈ 2.6KB) 의 메모리를 사용하고 있다는 뜻이다.

## Redis Data Types
* Strings
  * 가장 기본적인 타입
  * 문자열, 숫자, JSON 직렬화 데이터 모두 저장 가능
  * 카운터, 토큰, 간단한 캐시 값에 자주 사용
* Bitmaps
  * 비트 단위 연산이 가능해 사용자 출석 체크나 온라인 여부 확인 시 메모리를 극단적으로 아낄 수 있다.
* Lists
  * 순서가 있는 문자열 리스트
  * 양쪽 끝에서 push / pop 가능
  * 큐(queue)나 스택(stack)처럼 사용하기 좋다
  * [A, B, C]
  * LPUSHX(리스트 앞쪽) / RPUSHX(리스트 뒷쪽): key가 존재할 때만 push됨
* Hashes
  * 하나의 key 안에 여러 필드를 저장할 수 있다.  
```text
key
 ├─ field1 : value1
 ├─ field2 : value2
 └─ field3 : value3
``` 
* Sets
  * 중복 없는 문자열 집합
  * 순서 없음
  * {A, B, C}
* Sorted Sets(ZSets)
  * Set + 정렬 가능
  * 각 멤버는 score라는 숫자 값을 가진다.
  * {"LG":1, "KT":2, "SSG":3, "NC":4, "KIA":5}
  * score는 랭킹, 점수 기반 정렬, 시간 순 정렬 등으로 사용할 수 있다.
* HyperLogLogs
  * 중복되지 않는 값의 개수를 카운트할 때 사용한다.
  * 정확한 값이 아닌 근사치를 제공한다.
  * 데이터의 수와 상관없이 메모리 사용량은 항상 약 12KB로 고정된다.

## Sorted Set
중복되지 않는 문자열(member)을 점수(score)에 따라 정렬한 집합이다.
score가 같으면 member 문자열의 사전 순으로 정렬한다.

```text
key = leaderboard

score    member
----------------
100      userA
150      userC
200      userB
```

주요 명령어

| 명령어                            | 설명                              | 시간 복잡도        |
|--------------------------------|---------------------------------|---------------|
| ZADD key score member          | 새로운 멤버와 점수를 추가                  | O(log N)      |
| ZINCRBY key increment member   | 특정 멤버의 점수를 증가/감소                | O(log N)      |
| ZRANGE key start stop          | 점수 순위 기반 범위 조회 (낮은 순)           | O(log N + M)  |
| ZREVRANGE key start stop       | 점수 순위 기반 범위 조회 (높은 순)           | O(log N + M)  |
| ZRANK key member               | 특정 멤버의 순위 조회                    | O(log N)      |
| ZSCORE key member              | 특정 멤버의 점수 조회                    | O(1)          |
| ZREM key member                | 특정 멤버 삭제                        | O(log N)      |
| ZREMRANGEBYRANK key start stop | start ~ stop 범위에 있는 member 삭제   |  O(log N + M) |

ZADD 옵션

| 옵션   | 의미                |
| ---- | ----------------- |
| NX   | member 없을 때만 추가   |
| XX   | member 있을 때만 업데이트 |
| GT   | score 증가만 허용      |
| LT   | score 감소만 허용      |
| CH   | 변경된 member 수 반환   |
| INCR | score 증가 연산       |

실제 사용 사례

(1) 실시간 투표 및 랭킹 시스템
* Top 10 유저 등수/점수 확인
* 특정 유저의 등수/점수 확인

```text
ZADD key score member
ZADD leaders:exp 0 391(userId)
                   127
                   268
                   637
                   722
                   971
                   662
                   37
                   175
                   21
```
게임 리더보드 초기화, 초기 점수는 0으로 가정

```text
ZINCRBY key increment member
ZINCRBY leaders:exp 300 37(userId)
```
플레이어가 점수 획득

```text
│ score    member
│ ----------------
│ 100      userA
│ 150      userC
│ 200      userB
▼
```
ZRANGE

```text
▲ score    member
│ ----------------
│ 100      userA
│ 150      userC
│ 200      userB
```
```text
ZREVRANGE leaders:exp 0 9 WITHSCORES
```
ZREVRANGE 를 사용해 상위 Top 10을 조회할 수 있다.

```text
ZRANK leaders:exp 37 // 낮은 순 → 높은 순
ZREVRANK leaders:exp 37 // 높은 순 → 낮은 순
```
ZREVRANK를 사용해 userId 37의 등수를 확인할 수도 있다.

```text
ZSCORE leaders:exp 37
```
특정 유저의 점수를 조회할 수도 있다.

```text
ZINCRBY key increment member
ZINCRBY votes:event_1 1 candidate_A
```
실시간 투표의 경우 투표할 때마다 점수를 1 올릴 수 있다.

(2) 선착순 대기열
이벤트 페이지에 접속한 순서대로 대기 번호를 부여하고 순차적으로 입장시켜야하는 경우

```text
ZADD waiting_line {timestamp} {user_id}
```
score로 현재 시간을 사용하면 들어온 순서대로 정렬된다.

(3) 최근 본 상품, 최근 검색어
유저별로 최근에 본 상품 5개만 유지하고 싶은 경우
타임스탬프를 점수로 사용하여 데이터를 넣고 ZREMRANGEBYRANK를 사용해 5위 밖의 데이터는 자동으로 지워버릴 수 있다.

## Counting 전략
* 정확한 카운팅 (String, Hash, ZSet)
`INCR`, `INCRBY`, `ZINCRBY`와 같은 커맨드를 사용한다. 
오차없는 정확한 데이터지만 데이터 개수와 메모리 사용량이 함께 정비례해서 증가한다.

* 근사 카운팅 (HyperLogLogs)
```text
PFADD key member
PFADD key "a"
PFADD key "b"

PFCOUNT key // 123884584
PFMERGE key1, key2, key3 // merge된 결과
```
* 모든 string 값을 유니크하게 구분하여 개수를 추정할 수 있다.
* 대용량 데이터를 카운팅 할 때 적절하다 (오차 0.81%)
* set과 비슷하지만 저장되는 용량은 매우 작다(저장되는 데이터 개수에 상관없이 12KB 고정)
* 저장된 데이터는 다시 조회할 수 없다.
* 적합한 예
  * 웹사이트 방문한 유니크 IP 수
  * 유니크 검색어 수
  * 유니크 사용자 수 집계

## 레디스 동시성 문제?
동시에 요청이 2개 들어오는 경우
```text
A → redis 조회 → 없음
B → redis 조회 → 없음
```
```text
A → SET RUNNING → 애플리케이션 비지니스 로직 처리
B → SET RUNNING → 애플리케이션 비지니스 로직 처리
```
조회와 업데이트 단계가 분리되어있어 중복 업데이트가 발생할 수 있다.
해결방법(SETNX)

SETNX는 Redis 내부에서 원자적(atomic) 연산으로 실행된다.
그래서 동시에 요청이 와도 단 하나만 성공한다.

```text
SET key value NX(key 없을 때만 생성) EX(TTL) 3
```

request A
```text
SET key RUNNING NX → 성공
→ RUNNING 상태
→ 외부 API 호출
```

request B
```text
SET key RUNNING NX → 실패
→ retry
```

## 캐시를 사용할 때 대응해야 하는 주요 문제 상황
* 캐시 스탬피드 (Cache Stampede)
여러 개의 키에 동시에 캐시 미스가 발생하면 데이터베이스에 부하가 집중 될 수 있다.
특히 캐시 만료 시간이 동일하게 설정되어 있을 경우(예: 매일 자정), 특정 시점에 트래픽이 폭증할 수 있다.
이로 인해 데이터베이스에 과부하가 발생하고, 최악의 경우 서비스 장애로 이어질 수 있다.

해결 방법
1. 지터(Jitter, 짧은 지연 시간) 적용 
캐시 TTL(Time To Live)에 랜덤 값을 추가하여 만료 시점을 분산시킨다.
2. 선제적 캐시 갱신 (Cache Warming)
만료 전에 미리 캐시를 갱신하여 미스 발생 자체를 줄인다.

* 캐시 관통 (Cache Penetration)
캐시에 존재하지 않는 키에 대한 요청이 반복적으로 들어와 불필요하게 매번 데이터베이스를 조회하게 되는 현상이다.
데이터베이스에서 읽었는데도 캐싱 되지 않는 상황을 캐시 관통이라고 한다.
주로 존재하지 않는 데이터를 반복 조회할 때 발생한다.

해결 방법
1. Null 값 캐싱
DB 조회 결과가 없더라도 “없음”을 캐시에 저장하여 재조회를 방지한다.
2. Bloom Filter
애초에 존재하지 않는 키 요청을 필터링한다.

* 캐시 시스템 장애
캐시 시스템에 장애가 발생한 경우 복구될 때까지 데이터베이스에 부하가 발생할 수 있다.
반드시 동작해야 되는 핵심 기능을 제외하고, 편의를 위한 부가 기능은 일시적으로 운영을 중단하는것이 나을 수 있다.
데이터베이스 부하를 감안하더라도 꼭 동작해야할 기능인지 개발자가 미리 고민할 필요가있다.

* Hotkey 만료
많은 요청이 집중되는 키를 Hotkey라고 한다. 
핫키가 만료되는 순간 동시에 많은 요청이 DB를 조회하게 되는 문제가 발생할 수 있다.
가능하다면 캐시의 만료 기한을 없애거나, 백그라운드에서 주기적으로 새 값을 적용해서 캐시가 만료되지 않게 하는 것이 좋습니다.

해결 방법
1. 수동 갱신
캐시의 만료 기한을 없애거나 업데이트 시에만 갱신한다.
2. 백그라운드 갱신 (Refresh Ahead)
만료 전에 비동기적으로 캐시를 갱신한다.
3. 분산 락을 적용한다.
하나의 요청만 DB를 조회하도록 제한한다.

## Redis 분산 락(Distributed Lock) Redlock
https://redis.io/docs/latest/develop/clients/patterns/distributed-locks/

## Redis Message Broker
레디스는 캐시 저장소로 사용된다고 인식되고 있지만 메시지 브로커로 활용될 수 도 있다.
