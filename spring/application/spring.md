# Bean 생명주기 콜백 정리

초기화란?
객체가 생성되고 의존관계 주입까지 모두 끝난 뒤 실제로 사용 가능한 상태로 만드는 준비 작업

초기화 작업(init)은 의존관계 주입이 모두 완료된 이후에 실행되어야 한다.
하지만 개발자 입장에서는 의존관계 주입이 언제 끝났는지를 직접 알기 어렵다.
스프링은 이 문제를 해결하기 위해 콜백(Callback) 메커니즘을 지원한다.

* 초기화 콜백: 빈이 생성되고, 의존관계 주입이 완료되면 호출
* 소멸전 콜백: 빈이 소멸(컨테이너 종료)되기 직전에 호출

## 스프링 빈 라이프사이클

```text
스프링 컨테이너 생성 → 스프링 빈 생성 → 의존관계 주입 → 초기화 콜백 → 사용 → 소멸전 콜백 → 스프링 종료
```

💡 중요: 객체의 생성과 초기화를 분리하자 (단일 책임 원칙)

* 생성자: 필수 정보(파라미터)를 받아 메모리를 할당하고 객체를 생성하는 책임에 집중해야한다.
* 초기화: 생성된 값을 활용해 외부 커넥션을 연결하는 등 무거운 동작을 수행한다.

## `@Bean` 설정 정보에서 초기화 / 종료 메서드 지정

```java

@Configuration
public class AppConfig {

    @Bean(initMethod = "init", destroyMethod = "close")
    public DatabaseConnector databaseConnector() {
        DatabaseConnector connector = new DatabaseConnector();
        connector.setJdbcUrl("jdbc:mysql://localhost:3306/app");
        return connector;
    }
}
```

설정 정보에 `@Bean(initMethod = "init", destroyMethod = "close")` 처럼 초기화, 소멸 메서드를 지정할 수 있다.
메서드 이름을 자유롭게 지정할 수 있으며 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 종료 메서드를 적용할 수 있다.

## `@PostConstruct`, `@PreDestroy`

```java
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;

public class DatabaseConnector {

    private String jdbcUrl;

    public void setJdbcUrl(String jdbcUrl) {
        this.jdbcUrl = jdbcUrl;
    }

    @PostConstruct
    public void init() {
        System.out.println("DB 연결: " + jdbcUrl);
    }

    @PreDestroy
    public void close() {
        System.out.println("DB 연결 종료");
    }
}
```

@PostConstruct : 의존관계 주입 완료 후 자동 실행
@PreDestroy : 컨테이너 종료 직전 실행

코드가 간결하고 가독성이 좋다.
최신 스프링에서 **가장 권장하는 방법**이며 패키지가 `javax.annotation.PostConstruct` 이다.
스프링에 종속적인 기술이 아니라 JSR-250 라는 자바 표준으로 스프링이 아닌 다른 컨테이너에서도 동작한다.
단 외부 라이브러리에는 적용이 불가능하다. 외부 라이브러리를 초기화, 종료 해야 하면 @Bean 기능을 사용할 수 있다.

## @PostConstruct vs @EventListener(ApplicationReadyEvent)

초기화(init)는 빈이 쓸 준비가 되었는가?라면
ApplicationReadyEvent 는 애플리케이션이 이제 진짜 서비스 시작해도 되는가?

|       | `@PostConstruct`                          | `@EventListener(ApplicationReadyEvent)` |
|-------|-------------------------------------------|-----------------------------------------|
| 호출 시점 | 해당 **빈의 의존관계 주입 완료 직후**                   | **모든 빈**의 초기화가 끝나고 컨테이너가 완전히 생성된 후      |
| 의미    | 해당 빈이 이제 안전하게 사용 가능하다 (다른 빈의 상태는 보장하지 않음) | 애플리케이션 전체가 서비스 시작 가능하다                  |
| 트랜잭션  | 적용되기 어려움 (AOP 프록시 적용 시점 문제)               | 가능 (AOP가 완벽히 적용된 상태)                    |

# Bean 스코프
## 싱글톤
싱글톤 스코프의 빈을 조회하면 스프링 컨테이너는 항상 같은 인스턴스의 스프링 빈을 반환한다. <br >
여러 스레드에서 동시에 접근하므로 상태를 유지(Stateful)하게 설계하면 안된다.

## 프로토타입 
프로토타입 스코프를 스프링 컨테이너에 조회하면 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환한다.
개발자가 직접 `new`로 객체를 생성하는 것과 어떤 차이가 있을까?
* 의존관계 주입(DI)을 자동으로 처리하기 위해
  * 프로토타입 빈 내부에서도 다른 스프링 빈이 필요할 수 있다. 
* 빈 생명주기 콜백을 활용하기 위해
  * 객체를 직접 생성하면 라이프사이클 관리 기능을 사용할 수 없다. (객체 생성 직후 초기화 작업, @PostConstruct)
  * 종료 콜백(@PreDestroy)은 호출되지 않는다.

## 웹 관련 스코프 (Web Scopes)
* request: 웹 요청이 들어오고 나갈때 까지 유지되는 스코프이다.
* session: 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프이다.

