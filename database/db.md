## JDBC (인터페이스)
- 데이터베이스마다 커넥션을 연결하는 방법, SQL을 전달하는 방법, 결과를 응답받는 방법이 모두 다르다.
  - 데이터베이스를 변경하면 데이터베이스 사용 코드를 변경해야 하고, 데이터베이스마다 코드를 학습해야 한다.
  
→ 이 문제를 해결하기 위해 `JDBC`라는 자바 표준 데이터베이스 인터페이스 등장했다.
- JDBC: 데이터베이스에 접속할 수 있도록 하는 자바 API
- 애플리케이션 로직은 추상화된 JDBC 표준 인터페이스에만 의존한다.

## JDBC Driver (구현체)
DB 벤더(회사)에서 자신의 DB에 맞도록 구현해서 라이브러리로 제공하는데 이것을 JDBC 드라이버라고 한다.
예) MySQL JDBC 드라이버, Oracle JDBC 드라이버

## JDBC 대표 인터페이스
```text
// Connection 구현체를 반환하는 함수
Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);
```
데이터베이스에 연결하려면 JDBC가 제공하는 `DriverManager.getConnection(..)` 를 사용하면 된다.
`DriverManager`는 라이브러리에 등록된 Driver 구현체들에게 URL을 넘겨서 연결 가능 여부를 판단하고 가장 먼저 연결을 생성할 수 있는 Driver를 사용해 커넥션을 반환해준다.

## 순수 jDBC 코드 예시
```java
public Member findById(String memberId) throws SQLException {
    String sql = "select * from member where member_id = ?";

    Connection con = null;
    PreparedStatement psmt = null;
    ResultSet rs = null;

    try {
        con = getConnection(); // DriverManager.getConnection(URL, USERNAME, ..)
        psmt = con.prepareStatement(sql);
        psmt.setString(1, memberId);
        
        rs = psmt.executeQuery();
        if(rs.next()) {
            Member member = new Member();
            member.setMemberId(rs.getString("member_id"));
            member.setMoney(rs.getInt("money"));
            return member;
        } else {
            throw new NoSuchElementException("member not found memberID=" + memberId);
        }
    } catch (SQLException e) {
        log.error("db error", e);
        throw e;
    } finally {
        if(rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if(psmt != null) {
            try {
                psmt.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }

        if(con != null) {
            try {
                con.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }
    }
}
```

## JDBC를 편리하게 사용할 수 있는 기술
아래 기술들도 내부적으로는 모두 JDBC를 사용한다.

- SQL Mapper
  - +) SQL 응답 결과를 객체로 편리하게 반환해준다.
  - +) JDBC의 반복 코드를 제거해준다.
  - -) 개발자가 직접 SQL을 작성해야 한다.
  - 대표 기술: `스프링 JdbcTemplate`, `MyBatis`

- ORM 기술
  - 객체를 관계형 데이터베이스 테이블과 **맵핑**해주는 기술
  - ORM 기술이 개발자 대신에 SQL을 동적으로 만들어 실행해준다.

## 커넥션 풀과 데이터소스
DB 드라이버는 DB와 `TCP/IP` 로 커넥션을 연결한다.(3 way handshake, 시간 비용 ↑)
애플리케이션을 시작하는 시점에 커넥션 풀은 필요한 만큼 커넥션을 미리 확보해서 풀에 보관한다.(default 10개)
커넥션 풀에 들어있는 커넥션은 **TCP/IP 로 DB와 커넥션이 연결되어 있는 상태**이기 때문에 언제든지 즉시 SQL을 DB에 전달할 수 있다.
DB에 무한정 연결이 생성되는 것을 막아주어서 DB를 보호해주는 장점은 덤!
스프링 부트 2.0 부터는 기본 커넥션 풀로 hikariCP 오픈소스 커넥션 풀을 제공한다.

## 데이터 소스 (인터페이스)
커넥션을 얻는 방법은 DriverManager를 직접 사용하거나, 커넥션 풀을 사용하는 등 다양한 방법이 존재한다.

* DriverManager : 항상 새로운 커넥션 획득
* 커넥션 풀 : **커넥션 재사용** (con.close(): 커넥션 풀의 경우 커넥션을 닫는게 아니라 풀로 반환한다)

DataSource는 커넥션을 획득하는 방법을 추상화한 인터페이스로 자바는 javax.sql.DataSource 라는 인터페이스를 제공한다.
애플리케이션은 구체적인 커넥션 생성 방식에 의존하지 않고 DataSource를 통해 Connection을 획득할 수 있다.

```text
public interface DataSource {
    Connection getConnection() throws SQLException;
}
```
애플리케이션 로직은 DataSource 인터페이스에만 의존하도록 작성할 수 있으며
실제 구현체는 환경에 따라 교체할 수 있다.

## 트랜잭션의 이해
트랜잭션은 **데이터베이스의 상태를 변화시키는 하나의 논리적 작업 단위**이다.
트랜잭션에 포함된 여러 작업은 모두 성공하거나 모두 실패해야 하며 이를 보장하기 위해 데이터베이스는 ACID 특성을 제공한다.

| 특징 | 핵심 내용                                              | 계좌 이체 예시 (A ➡ B 1만원 송금)                                                 |
| :--- |:---------------------------------------------------|:------------------------------------------------------------------------|
| **원자성**<br>(Atomicity) | **All or Nothing**<br>전부 성공하거나, 전부 실패해야 한다.        | A의 잔고는 줄었는데 B의 잔고가 늘지 않는 일은 없어야 한다. 중간에 실패하면 A의 돈도 그대로여야 한다.            |
| **일관성**<br>(Consistency) | **규칙 준수**<br>DB에서 정한 도메인 규칙을 항상 만족해야 한다. (비즈니스 규칙) | '잔고는 0원 미만이 될 수 없다'는 규칙이 있다면, 돈이 부족한 A가 이체를 시도할 때 트랜잭션은 실패해야한다.         |
| **격리성**<br>(Isolation) | **간섭 배제**<br>동시에 실행되는 트랜잭션끼리 서로 영향을 주지 않아야 한다.     | A가 B에게 이체하는 동안, C가 A의 잔고를 조회해도 '이체 중인(아직 빠져나가지 않은) 중간결과' 금액을 볼 수 없어야한다. |
| **지속성**<br>(Durability) | **영구 저장**<br>성공한 트랜잭션은 시스템 오류가 나도 보존되어야한다.         | '이체 완료' 메시지가 떴다면, 그 직후에 은행 서버 전원이 꺼져도 나중에 켰을 때 이체 내역은 남아 있어야 한다.        |

클라이언트는 데이터베이스 서버에 연결을 요청하고 `커넥션`을 맺게 된다. 이때 데이터베이스 서버는 내부에 `세션`이라는 것을 만든다. 
그리고 앞으로 해당 **커넥션을 통한 모든 요청은 이 세션을 통해서 실행**하게 된다.
쉽게 이야기해서 개발자가 클라이언트를 통해 SQL을 전달하면 현재 커넥션에 연결된 세션이 SQL을 실행한다.
세션은 트랜잭션을 시작하고, 커밋 또는 롤백을 통해 트랜잭션을 종료한다. 그리고 이후에 새로운 트랜잭션을 다시 시작할 수 있다.
사용자가 커넥션을 닫거나, 또는 DBA(DB 관리자)가 세션을 강제로 종료하면 세션은 종료된다.

단 커넥션 풀에서는 아래 경우 세션이 종료된다.
* 커넥션 풀이 커넥션을 물리적으로 제거할 때
* 애플리케이션 종료 시
* DB 서버 또는 DBA가 세션을 강제로 종료
세션이 살아 있다는 것은 이전 사용자의 흔적이 남아 있을 수 있다는 뜻이기 때문에
커넥션 풀은 반납 시 반드시 초기화를 해야한다. (isolation level 원복, readOnly 원복 등)

데이터 변경 쿼리를 실행하고 데이터베이스에 그 결과를 반영하려면 커밋 명령어인 commit 을 호출하고, 결과를 반영하고 싶지 않으면 롤백 명령어인 rollback 을 호출하면 된다.
커밋을 호출하기 전까지는 임시로 데이터를 저장하는 것이다.
해당 트랜잭션을 시작한 세션(사용자)에게만 변경 데이터가 보이고 다른 세션(사용자)에게는 변경 데이터가 보이지 않는다.

Why?
세션2에서 세션1이 아직 커밋하지 않은 변경 데이터가 보이다면 세션1이 롤백 했을 때 데이터 정합성에 큰 문제가 발생한다.

## 트랜잭션 자동 커밋, 수동 커밋
```text
set autocommit true; // 자동 커밋 모드 설정
insert into member(member_id, money) values ('data1',10000); // 자동 커밋 
```
자동 커밋으로 설정하면 각각의 쿼리 실행 직후에 자동으로 커밋을 호출한다.