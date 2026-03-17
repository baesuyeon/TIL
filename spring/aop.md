## AOP
공통 관심 사항(cross-cutting concern) vs 핵심 관심 사항(core concern)
AOP는 **관심사의 분리**를 위한 **프로그래밍 패러다임**이다.

```kotlin
fun createOrder() {
    log.info("start")
    val start = System.currentTimeMillis()

    // 비즈니스 로직
    orderRepository.save(order)

    val end = System.currentTimeMillis()
    log.info("time = ${end - start}")
}
```
모든 비즈니스 로직에 소요시간을 측정해야할 때
모든 메서드마다 시간 측정 코드를 추가하는 것은 너무나도 번거롭다.
시간을 측정하는 기능은 핵심 관심 사항이 아니라 공통 관심 사항이다.

시간을 측정하는 로직과 **핵심 비즈니스의 로직이 섞이게 되면 유지보수가 어려워진다.**
또한 시간을 측정하는 로직을 변경할 때 **모든 로직을 찾아가면서 변경**해야 한다.

```text
Client
   ↓
Proxy (AOP 적용)
   ↓
Target (실제 객체)
```
Spring AOP는 프록시를 통해 구현된다.

## 프록시를 직접 만드는 방식
```kotlin
interface OrderService {
    fun order(): String
    fun cancel(): String
}

class OrderServiceImpl : OrderService {
    override fun order(): String {
        println("👉 주문 처리")
        return "order complete"
    }

    override fun cancel(): String {
        println("👉 주문 취소")
        return "cancel complete"
    }
}
```

직접 만든 프록시
```kotlin
class OrderServiceProxy(
    private val target: OrderService
) : OrderService {

    override fun order(): String {
        println("🔥 트랜잭션 시작")

        val result = target.order()

        println("✅ 트랜잭션 커밋")

        return result.uppercase()
    }

    override fun cancel(): String {
        println("🔥 트랜잭션 시작")

        val result = target.cancel()

        println("✅ 트랜잭션 커밋")

        return result.uppercase()
    }
}
```
직접 프록시를 만드는 경우 인터페이스의 메서드가 많아지면 코드를 작성하는 것이 상당히 부담스러워진다.
위 예시에서는 2개지만 10개, 20개가 되는 경우 프록시 클래스에서 모든 메서드를 다 override 해야한다.
모든 메서드에 트랙잭션 기능을 부여하는 코드가 중복될 수 있다.

## 프록시를 자동으로 만들어주는 기술
* JDK Dynamic Proxy
* CGLIB

Spring AOP는 프록시를 통해 구현되며 Spring Boot 2.x 이상부터는 인터페이스 유무와 상관없이 **CGLIB**을 기본 프록시 생성 방식으로 사용한다.
(AOP를 구현하는 방법에는 프록시 뿐만 아니라 컴파일 시점이나 클래스 로딩 시점에 바이트코드를 직접 수정(Weaving)해서 부가 기능을 추가하는 AspectJ 방식도 있다
구현의 편의성을 위해 프록시 방식이 채택되었다)

### JDK Dynamic Proxy
인터페이스만 주면 모든 메서드를 자동으로 가로채는 프록시를 런타임에 만들어준다. (리플렉션 기능 사용)

```kotlin
interface OrderService {
    fun order()
    fun cancel()
}

class OrderServiceImpl : OrderService {
    override fun order() {
        println("주문 처리")
    }

    override fun cancel() {
        println("주문 취소")
    }
}
```

```kotlin
class TransactionHandler(
    private val target: Any
) : InvocationHandler {

    override fun invoke( // 프록시가 호출하는 진입점
        proxy: Any,
        method: Method,
        args: Array<out Any>?
    ): Any? {

        println("🔥 트랜잭션 시작")

        try {
            val result = if (args != null) {
                method.invoke(target, *args)
            } else {
                method.invoke(target)
            }

            println("✅ 트랜잭션 커밋")
            return result

        } catch (e: Exception) {
            println("❌ 트랜잭션 롤백")
            throw e
        }
    }
}
```
```text
1. proxy.order()
2. → invoke(proxy, orderMethod, null)   // 예시에서 구현한 invoke 호출
3. → println("🔥 트랜잭션 시작")
4. → method.invoke(target)              // 진짜 order() 실행
5. → println("✅ 트랜잭션 커밋")
```
InvocationHandler를 구현한 Handler는 반드시 `invoke()`를 구현해야 한다.
인터페이스와 함께 InvocationHandler 구현체를 제공해주면 프록시가 받은 모든 요청을 `invoke()` 메서드로 보내주면서
해당 메서드는 프록시 객체가 자동으로 호출하게된다. (인터페이스의 모든 메서드 호출이 여기로 들어온다)

```kotlin
fun main() {
    val target = OrderServiceImpl()

    val proxy = Proxy.newProxyInstance( // 인터페이스만 주면 모든 메서드를 구현한 구현체를 자동 생성해준다.
        OrderService::class.java.classLoader, // 동적으로 생성되는 다이나믹 프록시 클래스의 로딩에 사용할 클래스 로더
        arrayOf(OrderService::class.java), // 구현할 인터페이스
        TransactionHandler(target) // 부가 기능을 담은 invocationHandler
    ) as OrderService

    proxy.order()
    println("----")
    proxy.cancel()
}
```
```text
🔥 트랜잭션 시작
주문 처리
✅ 트랜잭션 커밋
----
🔥 트랜잭션 시작
주문 취소
✅ 트랜잭션 커밋
```
실행 결과는 위와 같다.

* JDK Dynamic Proxy의 문제점
