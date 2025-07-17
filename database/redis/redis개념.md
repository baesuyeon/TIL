> 참고: [Redis 공식 문서](https://redis.io/)

## Redis란

레디스(Redis)는 Remote Dictionary Server의 약자로서, "키-값" 구조의 비정형 데이터를 저장하고 관리하기 위한 오픈 소스 비관계형 데이터베이스 관리 시스템(DBMS)이다.


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

*  원격접속
```bash
redis-cli -h #{호스트명} -p #{포트번호}
```

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