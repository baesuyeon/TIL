> 참고: 김영한님 Spring Database

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
자동 커밋으로 설정하면 각각의 쿼리 실행 직후에 자동으로 커밋을 수행한다. <br >
`commit` , `rollback` 을 직접 호출하면서 트랜잭션 기능을 제대로 수행하려면 자동 커밋을 끄고 **수동 커밋**을 사용해야 한다. <br>
보통 자동 커밋 모드가 기본으로 설정된 경우가 많기 때문에 **수동 커밋 모드로 설정하는 것을 트랜잭션을 시작** 한다고 표현할 수 있다. <br>
수동 커밋 설정을 하면 이후에 꼭 commit , rollback 을 호출해야 한다. <br>
참고로 수동 커밋 모드나 자동 커밋 모드는 **한번 설정하면 해당 세션에서는 계속 유지**된다. <br>

## DB 락 - UPDATE : 데이터를 동시에 수정하는 것을 막는다
세션1이 트랜잭션을 시작하고 데이터를 수정하는 동안 아직 커밋을 수행하지 않았는데
세션2에서 동시에 같은 데이터를 수정하게 되면 정합성 문제가 발생한다.

문제를 방지하려면 세션이 트랜잭션을 시작하고 데이터를 수정하는 동안에는 커밋이나 롤백 전까지 **다른 세션에서 해당 데이터를 수정할 수 없게 막아야** 한다.

이를 막기 위해 DB는 락(Lock)을 사용한다.
```
set autocommit false;
update member set money=500 where id = 10;
```
InnoDB의 경우 UPDATE 실행 순간 자동으로 `Row-level Lock`을 건다. (X Lock (Exclusive Lock) 이 걸림) <br>
* 대상: id = 10 에 해당하는 row
* 락 해제 시점: COMMIT 또는 ROLLBACK 시

세션2는 락이 돌아올 때 까지 대기하게된다.
락을 무한정 대기하는 것은 아니고 락 대기 시간을 넘어가면 락 타임아웃 오류가 발생한다.

## DB 락 - READ
일반적인 조회는 락을 사용하지 않는다.
**락을 획득하지 않고 바로 데이터를 조회**할 수 있어서 세션1이 락을 획득하고 데이터를 변경하고 있어도, **세션2에서 데이터를 조회는 할 수 있다**.

```text
set autocommit false;
select * from member where member_id=10 for update;
```
데이터를 조회할 때도 락을 획득하고 싶을 때가 있다.
이럴 때는 **select for update** 구문을 사용하면 된다.
이렇게 하면 세션1이 **조회 시점에 락을 가져가버리기 때문에** 다른 세션에서 해당 데이터를 변경할 수 없다.
내가 조회하는 동안 다른 곳에서 변경하지마! (❗️조회는 허용된다)

## 애플리케이션에 트랜잭션 적용하기
커넥션을 통해 세션이 맺어지고 결과적으로 세션에서 트랜잭션이 시작되고 SQL이 실행되고 트랜잭션이 커밋되거나 롤백되기 때문에
트랜잭션을 시작하려면 커넥션이 필요하다. 트랜잭션을 사용하는 동안 같은 커넥션을 유지해야 같은 세션을 사용할 수 있다.

JDBC 트랜잭션 예제
```java
public void accountTransfer(String fromId, String toId, int money) throws SQLException {
    Connection con = dataSource.getConnection();
    try {
        con.setAutoCommit(false); // 트랜잭션 시작
        // 비즈니스 로직 수행
        Member fromMember = memberRepository.findById(con, fromId);
        Member toMember = memberRepository.findById(con, toId);

        memberRepository.update(con, fromId, fromMember.getMoney() - money);
        memberRepository.update(con, toId, toMember.getMoney() + money);
        con.commit();
    } catch(Exception e) {
        con.rollback();
        throw new IllegalStateException(e);
    } finally {
        if (con != null) {
            try {
                con.setAutoCommit(true); // 커넥션 풀로 돌려주기 전에 설정
                con.close();
            } catch (Exception e) {
                log.info("error", e);
            }
        }
    }
}
```
애플리케이션 로직에서 같은 커넥션을 유지하기 위해 **메서드 파라미터로 커넥션을 넘기고** 있다.
위 코드에서는 **비지니스 로직을 트랜잭션 코드가 감싸고** 있다.

jpa 트랜잭션 예제
```text
//엔티티 매니저 팩토리 생성
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성 
EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득

try {
  tx.begin(); //트랜잭션 시작 
  logic(em); //비즈니스 로직 
  tx.commit();//트랜잭션 커밋
} catch (Exception e) { 
  tx.rollback(); //트랜잭션 롤백
} finally {
    em.close(); //엔티티 매니저 종료
}
emf.close(); //엔티티 매니저 팩토리 종료
```

비즈니스 로직을 감싸는 트랜잭션 처리 코드의 형태가 구현 기술에 따라 달라진다
서비스 계층은 가급적 비즈니스 로직만 구현하고 특정 구현 기술에 직접 의존해서는 안된다. 
이렇게 하면 향후 구현 기술이 변경될 때 변경의 영향 범위를 최소화 할 수 있다.

문제점 정리
* 트랜잭션 문제
  * 트랜잭션 구현 기술이 비지니스 로직에 섞여있어 변경 시 영향 범위가 크다.
  * 트랜잭션 동기화 문제, 같은 트랜잭션을 유지하기 위해 커넥션을 파라미터로 넘거야 한다.
* 예외 누수
  * SQLException은 JDBC 전용 예외로 JPA로 변경하면 예외 타입도 변경되어 서비스 코드 수정이 불가피하다.
* 코드 반복 문제
  *  유사한 코드의 반복이 너무 많다. (try-catch-finally, 커넥션을 열기/닫기, commit/rollback, 리소스 정리)

## 트랜잭션 추상화
트랜잭션 추상화 인터페이스
```text
package org.springframework.transaction;

public interface PlatformTransactionManager extends TransactionManager {
  TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
          throws TransactionException;

  void commit(TransactionStatus status) throws TransactionException;
  void rollback(TransactionStatus status) throws TransactionException;
}
```
트랜잭션의 개념은 동일하지만 기술마다 구현 방법이 다르다. 
따라서 서비스 계층이 특정 기술(jdbc, jpa)에 직접 의존하게 된다.

트랜잭션을 다루는 방식을 추상화한 PlatformTransactionManager 인터페이스를 기반으로 각각의 기술에 맞는 구현체를 만들면 된다.
* `getTransaction()`: 트랜잭션을 시작한다.
* `commit()`: 트랜잭션을 커밋한다.
* `rollback()`: 트랜잭션을 롤백한다.

* JdbcTxManager : JDBC 트랜잭션 기능을 제공하는 구현체
* JpaTxManager : JPA 트랜잭션 기능을 제공하는 구현체

```text
public class OrderService {

    private final txManager PlatformTransactionManager;

    public void order() {
        txManager.begin();
        try {
            // 비즈니스 로직
            txManager.commit();
        } catch (Exception e) {
            txManager.rollback();
            throw e;
        }
    }
}
```
서비스 코드는 txManager에만 의존하고 사용하는 기술이 jdbc인지, jpa인지 전혀 알 필요가 없다.

## 트랜잭션 동기화
**트랜잭션 매니저는 내부에서 트랜잭션 동기화 매니저를 사용한다.**
트랜잭션 동기화 매니저는 쓰레드 로컬(ThreadLocal)을 사용해서 커넥션을 동기화해준다.

동작 방식
클라이언트 요청으로 서비스 로직을 시작한다.
1. 트랜잭션을 시작하려면 커넥션이 필요하다. 서비스 계층에서 `transactionManager.getTransaction()`을 호출해 트랜잭션을 시작한다.
2. 트랜잭션을 시작하려면 데이터베이스 커넥션이 필요하기 때문에 트랜잭션 매니저는 내부에서 데이터소스를 통해 커넥션을 생성한다.
3. 커넥션을 수동 커밋 모드로 변경해서 실제 데이터베이스 트랜잭션을 시작한다.
4. 트랜잭션 매니저는 트랜잭션이 시작된 커넥션을 트랜잭션 동기화 매니저에 보관한다.
5. 트랜잭션 동기화 매니저는 쓰레드 로컬에 커넥션을 보관한다. 따라서 멀티 스레드 환경에 안전하게 커넥션을 보관할 수 있다.
   * 쓰레드 로컬을 사용하면 각각의 쓰레드마다 별도의 저장소가 부여되어 다른 스레드가 같은 커넥션을 사용하는 문제점이 발생하지 않는다.
6. 리포지토리는 트랜잭션 동기화 매니저에 보관된 커넥션을 꺼내서 사용한다.(`DataSourceUtils.getConnection()`) 따라서 파라미터로 커넥션을 전달하지 않아도 된다.
7. 트랜잭션이 종료되면 트랜잭션 매니저는 트랜잭션 동기화 매니저에 보관된 커넥션을 통해 트랜잭션을 종료하고 커넥션도 닫는다.
   * 쓰레드 로컬을 사용하면 각각의 쓰레드마다 별도의 저장소가 부여되어 다른 스레드가 같은 커넥션을 사용하는 문제점이 발생하지 않는다.

## 트랜잭션 템플릿
```text
// 트랜잭션 시작
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

try {
  //비즈니스 로직
  bizLogic(fromId, toId, money); 
  transactionManager.commit(status); // 성공시 커밋
} catch (Exception e) { 
  transactionManager.rollback(status); //실패시 롤백
  throw new IllegalStateException(e);
}
```
트랜잭션을 사용하는 로직을 살펴보면 위와 같은 패턴이 반복된다.
템플릿 콜백 패턴을 활용하면 이런 반복 문제를 깔끔하게 해결할 수 있다.
스프링은 TransactionTemplate 라는 템플릿 클래스를 제공한다.

TransactionTemplate
```text
public class TransactionTemplate {

    private PlatformTransactionManager transactionManager;
    
    public <T> T execute(TransactionCallback<T> action) {..}
    
    void executeWithoutResult(Consumer<TransactionStatus> action) {..}
}
```

```text
public class MemberServiceV3_2 {

    private final TransactionTemplate txTemplate;
    private final MemberRepositoryV3 memberRepository;

    public MemberServiceV3_2(PlatformTransactionManager transactionManager, MemberRepositoryV3 memberRepository) {
        this.txTemplate = new TransactionTemplate(transactionManager);
        this.memberRepository = memberRepository;
    }

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        txTemplate.executeWithoutResult((status) -> {
            try {
                bizlogic(fromId, toId, money);
            } catch (SQLException e) {
                throw new IllegalStateException(e);
            }
        });
    }

    private void bizlogic(String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(fromId);
        Member toMember = memberRepository.findById(toId);

        memberRepository.update(fromId, fromMember.getMoney() - money);
        memberRepository.update(toId, toMember.getMoney() + money);
    }
}
```
트랜잭션 템플릿의 기본 동작은 다음과 같다.
* 비즈니스 로직이 정상 수행되면 커밋한다.
* 언체크 예외가 발생하면 롤백한다. 체크 예외인 경우 커밋한다.

트랜잭션 템플릿 덕분에, 트랜잭션을 사용할 때 반복하는 코드를 제거할 수 있었다.
하지만 서비스 계층엔 순수한 비즈니스 로직만 남긴다는 목표는 아직 달성하지 못했다.

## 트랜잭션 AOP
@Transactional 을 사용하면 스프링이 AOP를 사용해서 트랜잭션을 편리하게 처리해준다

프록시를 사용하면 트랜잭션을 처리하는 객체와 비즈니스 로직을 처리하는 서비스 객체를 명확하게 분리할 수 있다.
개발자는 트랜잭션 처리가 필요한 곳에 `@Transactional` 애노테이션만 붙여주면 된다.

`@Transactional` 이 없으면 자동으로 Auto Commit 모드가 적용된다.
쿼리 하나하나 수행될 때마다 Commit 되기 때문에 중간에 예외라도 발생하면 큰일이다.

## @Transactional의 기본 롤백 규칙
@Transactional은 기본적으로 RuntimeException과 Error에 대해서만 자동 롤백한다.
체크 예외(Checked Exception) 가 발생하면 자동 롤백되지 않는다.

```text
@Transactional
public void service() throws SQLException {
    // SQLException 발생 → 기본 설정에서는 롤백 ❌
}
```

왜 스프링은 이렇게 설계했을까?
자바에서 체크 예외는 원래 “복구 가능성이 있는 예외”를 의미한다.
“개발자가 잡아서 처리할 수도 있는 예외면 자동 롤백까지 해버리진 말자” => 기본 값
하지만 DB 예외는 복구 불가능한 경우가 대부분이다.

```text
@Repository(표식) → AOP 프록시 생성 → 예외 변환 Advice 적용
```

```text
JPA → PersistenceException 발생(이미 언체크 예외) → 스프링이 DataAccessException으로 변환(공통 예외 계층)
```

```text
JdbcTemplate → 내부에서 SQLException 을 잡아서 → DataAccessException으로 변환
```
실무에서 문제가 생기지 않는 이유는 SQLException을 그대로 쓰지 않고 JdbcTemplate, @Repository, JPA 등을 사용하는데
이들은 내부에서 SQLException (checked) → DataAccessException (RuntimeException) 으로 자동 변환한다.

SQLException을 쓰면 어떻게 해야 하나?
@Transactional 어노테이션에서 rollbackFor 옵션을 제공한다.
@Transactional(rollbackFor = SQLException.class)

