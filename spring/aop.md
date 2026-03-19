## AOP
공통 관심 사항(cross-cutting concern)과 핵심 관심 사항(core concern)
AOP는 이러한 **관심사의 분리**를 위한 **프로그래밍 패러다임**이다.

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
모든 비즈니스 로직에서 실행 시간을 측정해야 한다고 가정해보자.
이 경우 모든 메서드마다 시간 측정 코드를 추가하는 것은 매우 번거롭다.
시간을 측정하는 기능은 핵심 관심 사항이 아니라 공통 관심 사항에 해당한다.

시간을 측정하는 로직과 **핵심 비즈니스의 로직이 섞이게 되면 유지보수가 어려워진다.**
또한 시간을 측정하는 로직(공통 로직)을 변경할 때 **모든 로직을 찾아가면서 변경**해야 한다.

이러한 문제를 해결하기 위해 AOP를 사용하여 관심사를 분리한다.
```text
Client
   ↓
Proxy (AOP 적용)
   ↓
Target (실제 객체)
```
Spring AOP는 프록시를 통해 구현된다.
프록시를 통해 공통 관심 사항을 분리하고 핵심 비즈니스 로직에 영향을 주지 않으면서 부가 기능을 적용한다.

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

Spring AOP는 프록시를 기반으로 동작하며 Spring Boot 2.x 이상부터는 인터페이스 유무와 관계없이 **CGLIB**을 기본 프록시 생성 방식으로 사용한다.
(참고로 AOP를 구현하는 방법에는 프록시 뿐만 아니라 컴파일 시점이나 클래스 로딩 시점에 바이트코드를 직접 조작(Weaving)해서 부가 기능을 추가하는 AspectJ 방식도 있다
다만 구현의 단순성과 사용 편의성을 위해 Spring에서는 프록시 기반 방식을 주로 사용한다)

### JDK Dynamic Proxy
JDK Dynamic Proxy는 **인터페이스 기반 프록시 생성 기술**로  
인터페이스만 주면 모든 메서드를 자동으로 구현한 프록시를 런타임에 만들어준다.

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
InvocationHandler는 프록시가 호출을 위임하는 핵심 인터페이스로 모든 메서드 호출은 `invoke()` 메서드를 통해 처리된다.

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

실행 흐름
```text
1. proxy.order()
2. → invoke(proxy, orderMethod, null)   // 예시에서 구현한 invoke 호출
3. → println("🔥 트랜잭션 시작")
4. → method.invoke(target)              // 실제 order() 실행
5. → println("✅ 트랜잭션 커밋")
```
InvocationHandler를 구현한 Handler는 반드시 `invoke()`를 구현해야 한다.
인터페이스와 함께 InvocationHandler 구현체를 제공해주면 프록시가 받은 모든 요청을 `invoke()` 메서드로 보내주면서
해당 메서드는 프록시 객체가 자동으로 호출하게된다. (인터페이스의 모든 메서드 호출이 여기로 들어온다)

**[JDK Dynamic Proxy 정리]**
* JDK Dynamic Proxy는 인터페이스 기반으로 동작한다.
* 프록시는 인터페이스의 모든 메서드 호출을 invoke()로 위임한다.
* InvocationHandler에서 공통 로직(트랜잭션, 로깅 등)을 처리할 수 있다.

**[JDK Dynamic Proxy의 문제점]**
```kotlin
val proxy = Proxy.newProxyInstance(...)
```
JDK Dynamic Proxy는 생성자가 없어 `new`로 객체 생성이 불가능하고 반드시 `Proxy.newProxyInstance()` static 메서드를 통해 런타임에 생성해야한다.
이로 인해 스프링이 일반적인 방식(리플렉션 기반)으로 객체를 생성할 수 없다.
스프링이 객체 생성 과정에 직접 개입하기 어려워 빈으로 등록하고 DI를 통해 관리하는데 제약이 있다.

이를 해결하기 위해 `FactoryBean` 인터페이스를 사용한다.
스프링은 FactoryBean 자체를 빈으로 등록하고
`getObject()` 메서드를 호출하여 반환된 객체(프록시)를 실제 빈으로 사용한다.

즉 FactoryBean을 통해 스프링이 생성 과정에 간접적으로 참여할 수 있게 되어
JDK Dynamic Proxy 객체도 빈으로 등록하고 DI할 수 있다.

```kotlin
class TxProxyFactoryBean<T : Any>(
    private var target: T,
    private var transactionManager: PlatformTransactionManager,
    private var pattern: String,
    private var serviceInterface: Class<T>
) : FactoryBean<T> {

    override fun getObject(): T {
        val handler = TransactionHandler(
            target = target,
            transactionManager = transactionManager,
            pattern = pattern
        )

        return Proxy.newProxyInstance(
            serviceInterface.classLoader,
            arrayOf(serviceInterface),
            handler
        ) as T
    }

    override fun getObjectType(): Class<*> {
        return serviceInterface
    }

    // 싱글톤 빈이 아니라는 뜻이 아니라 getObject()가 "항상 새로운 객체를 반환할 수 있다"는 의미
    override fun isSingleton(): Boolean {
        return false
    }
}
```
```kotlin
val factory = TxProxyFactoryBean(
    target = OrderServiceImpl(),
    transactionManager = txManager,
    pattern = "order*",
    serviceInterface = OrderService::class.java
)

val proxy: OrderService = factory.getObject()
```
1. TxProxyFactoryBean (FactoryBean) 을 빈으로 등록
2. 스프링이 getObject() 호출
3. 반환된 프록시 객체를 실제 빈처럼 사용

**[JDK Dynamic Proxy 방식의 한계]**
* 설정의 중복
  * 하나의 프록시 팩토리 빈은 하나의 타겟에만 대응하므로 부가 기능(예: 트랜잭션)을 적용할 타겟 클래스가 100개라면 100개의 ProxyFactoryBean 설정이 필요하다.
  * 적용 대상이 많아질수록 설정 코드가 반복되고 설정 코드가 반복되어 관리가 어려워진다.
* 타겟마다 FactoryBean과 Handler를 개별적으로 생성해야 한다
  * InvocationHandler가 target과 부가기능 같이 가지고 있다.

**[Spring ProxyFactoryBean]**
이러한 JDK Dynamic Proxy 방식의 한계를 해결하기 위해  
Spring은 `ProxyFactoryBean`을 제공한다.

ProxyFactoryBean은 프록시 생성 과정을 추상화하여 타겟 객체와 부가기능을 분리해서 설정할 수 있도록 한다.

### 부가기능(Advice)과 타겟의 분리
JDK Dynamic Proxy에서는 `InvocationHandler`가 타겟 객체와 부가기능을 함께 가지고 있었다.
반면 ProxyFactoryBean에서는 이를 다음과 같이 분리하여 부가기능을 독립적으로 재사용할 수 있다.
```text
ProxyFactoryBean
   ├── Target (비즈니스 로직) 
   └── Advice (부가기능)
```

[기존 방식]
```kotlin
class TransactionHandler(
    private val target: Any
) : InvocationHandler {

    override fun invoke(
        proxy: Any,
        method: Method,
        args: Array<out Any>?
    ): Any? {

        println("🔥 트랜잭션 시작")

        return try {
            val result = if (args != null) {
                method.invoke(target, *args)
            } else {
                method.invoke(target)
            }

            println("✅ 트랜잭션 커밋")
            result

        } catch (e: Exception) {
            println("❌ 트랜잭션 롤백")
            throw e
        }
    }
}
```
```kotlin
fun main() {
    val target = OrderServiceImpl()

    val proxy = Proxy.newProxyInstance(
        OrderService::class.java.classLoader,
        arrayOf(OrderService::class.java),
        TransactionHandler(target)
    ) as OrderService

    proxy.order()
    println("----")
    proxy.cancel()
}
```

[ProxyFactoryBean 방식]
```kotlin
class TransactionAdvice : MethodInterceptor {

    override fun invoke(invocation: MethodInvocation): Any? {
        println("🔥 트랜잭션 시작")

        return try {
            val result = invocation.proceed()

            println("✅ 트랜잭션 커밋")
            result

        } catch (e: Exception) {
            println("❌ 트랜잭션 롤백")
            throw e
        }
    }
}
```
```kotlin
fun main() {
    val pfBean = ProxyFactoryBean()

    pfBean.setTarget(OrderServiceImpl())
    pfBean.addAdvice(TransactionAdvice())

    val proxy = pfBean.getObject() as OrderService

    proxy.order()
    println("----")
    proxy.cancel()
}
```
기존 방식에서는 `method.invoke(target, *args)` 에서 우리가 직접 어떤 메서드인지 알고 target 넣고 실행까지 해야했다.
`invocation.proceed()` 방식에서 MethodInvocation은 (현재 호출된 메서드 + 타겟 + 실행 흐름)을 모두 가지고 있는 객체다.
다음 실행 대상(= 실제 타겟 메서드)을 실행하라는 의미

InvocationHandler를 구현했을 때와 달리 MethodInterceptor를 구현한 TransactionAdvice 에는 타겟 오브젝트가 등장하지 않아
MethodInterceptor를 싱글톤으로 두고 공유하여 사용할 수 있어 중복 문제를 일부 해결할 수 있다.

또한 하나의 프록시에 여러 개의 Advice를 등록해서 체인처럼 실행할 수 있다.
JDK Dynamic Proxy와 다르게 타겟이 구현하고 있는 인터페이스를 대신 찾기 때문에 인터페이스를 직접 제공해주지 않아도 된다.

```text
MethodInvocation
 ├── method        (어떤 메서드인지)
 ├── arguments     (파라미터)
 ├── target        (실제 객체)
 └── proceed()     (다음 실행)
```

일부 중복은 해결되지만 트랜잭션 적용 대상마다 ProxyFactoryBean을 설정해주어야 하는 문제는 남아있다.

[DefaultAdvisorAutoProxyCreator]
스프링이 제공하는 빈 후처리기 중 하나인 DefaultAdvisorAutoProxyCreator가 있다.


