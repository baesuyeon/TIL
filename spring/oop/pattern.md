* 참고: [얄코 OODP](https://yalco.notion.site/)

## 디자인 패턴

### 파사드(Facade) 패턴
```text
                                              ┌───────────────┐
                                              │  Subsystem A  │
                                              │  Subsystem B  │
 [  Client  ]  ─────▶  [   Facade   ]  ─────▶ │  Subsystem C  │
                                              │  Subsystem D  │
                                              └───────────────┘
```
Facade란?
* 사용하기 편리한 단순한 클래스이다.
* Facade가 내부에서 여러 서브시스템을 조합하여 호출한다.
* 복잡한 서브시스템을 감싸서 사용하기 쉬운 진입점을 제공한다.

Client는 Facade만 알고있기 때문에
서브시스템끼리 서로 복잡하게 얽혀 있어도 Client는 간단하게 사용할 수 있다.

## 전략(Strategy) 패턴
```text
[ Strategy 1 ] ─┐
[ Strategy 2 ] ─┤
[ Strategy 3 ] ─┼────▶  [   Context   ]  ─────▶  [  Client  ]
[ Strategy 4 ] ─┘
```
특정 작업을 하는 방식(전략)들을 여러개 두고
필요에 따라 갈아끼울 수 있도록 하는 패턴이다.
여러 전략 구현체 중 하나를 Context에 주입하여 사용하며
Client는 전략을 직접 사용하지 않고 Context를 통해 기능을 사용한다.

```kotlin
interface PaymentStrategy {
    fun pay(amount: Int)
}

class CashPaymentStrategy : PaymentStrategy {  }
class CreditCardPayment : PaymentStrategy {  }
class PointPaymentStrategy : PaymentStrategy {  }

class ShoppingCartContext(val strategy: PaymentStrategy) {
    fun checkout(amount: Int): Int {
        return strategy.pay(amount)
    }
}
```
계산 전략
* 현금 결제
* 카드 결제
* 포인트 결제

## 템플릿 메서드(Template Method) 패턴
에피타이저 → 수프 → 샐러드 → 생선 요리 → 메인 요리 → 디저트 → 커피/티

```kotlin
abstract class FrenchCourseTemplate {

    // 템플릿 메서드: 전체 흐름을 고정
    fun serveCourse() {
        serveAppetizer()
        serveSoup()
        serveSalad()
        serveFish()
        serveMain()
        serveDessert()
        serveDrink()
    }

    protected abstract fun serveAppetizer()
    protected abstract fun serveSoup()
    protected abstract fun serveSalad()
    protected abstract fun serveFish()
    protected abstract fun serveMain()
    protected abstract fun serveDessert()
    protected abstract fun serveDrink()
}
```

템플릿 메서드 패턴은 정해진 순서의 처리 흐름(알고리즘 뼈대)을 상위 클래스에서 정의하고
각 단계를 하위 클래스에서 자유롭게 구현하도록 하는 패턴이다.
전체 흐름은 항상 동일한 순서로 실행되지만,
각 단계의 세부 구현은 상황에 맞게 변경될 수 있을 때 유용하다.
→ “전체 흐름은 고정하고 세부 구현만 바꾸는 패턴”

실무와 가까운 예제
```kotlin
abstract class ProductTemplateConverter<R> {

  fun convert(products: List<Product>): List<R> =
    products.mapNotNull { product ->
      if (!shouldConvertRemoved() && product.isRemoved) return@mapNotNull null
      if (!shouldConvertHidden() && product.isHidden) return@mapNotNull null
      if (!shouldConvertBlocked() && product.isBlocked) return@mapNotNull null

      when (product) {
        is PBProduct -> convertPBProduct(product)
        is ImportedProduct -> convertImportedProduct(product)
        is LocalProduct -> convertLocalProduct(product)
      }
    }
  
  abstract fun convertPBProduct(product: PBProduct): R?
  abstract fun convertImportedProduct(product: ImportedProduct): R?
  abstract fun convertLocalProduct(product: LocalProduct): R?
  
  open fun shouldConvertRemoved(): Boolean = false
  open fun shouldConvertHidden(): Boolean = false
  open fun shouldConvertBlocked(): Boolean = false
}
```

```kotlin
class ProductDtoV1TemplateConverter : ProductTemplateConverter<ProductDtoV1>() {

    override fun convertPBProduct(product: PBProduct): ProductDtoV1 =
        ProductDtoV1(id = product.id)

    override fun convertImportedProduct(product: ImportedProduct): ProductDtoV1 =
        ProductDtoV1(id = product.id)

    override fun convertLocalProduct(product: LocalProduct): ProductDtoV1 =
        ProductDtoV1(id = product.id)

    override fun shouldConvertRemoved(): Boolean = false
    override fun shouldConvertHidden(): Boolean = false
    override fun shouldConvertBlocked(): Boolean = false
}

val convertedProducts: List<ProductDtoV1> = ProductDtoV1TemplateConverter.convert(products)
```

```kotlin
class ProductDtoV2TemplateConverter : ProductTemplateConverter<ProductDtoV2>() {

    override fun convertPBProduct(product: PBProduct): ProductDtoV2 =
        ProductDtoV2(id = product.id)

    override fun convertImportedProduct(product: ImportedProduct): ProductDtoV2 =
        ProductDtoV2(id = product.id)

    override fun convertLocalProduct(product: LocalProduct): ProductDtoV2 =
        ProductDtoV2(id = product.id)

    override fun shouldConvertRemoved(): Boolean = false
    override fun shouldConvertHidden(): Boolean = false
    override fun shouldConvertBlocked(): Boolean = false
}

val convertedProducts: List<ProductDtoV2> = ProductDtoV2TemplateConverter.convert(products)
```

convert() 메서드는 알고리즘의 뼈대(흐름)를 정의하고
convertPBProduct, convertImportedProduct, convertLocalProduct, shouldConvertRemoved 등은 변경 가능한 세부 단계를 담당한다.
즉 전체 구조는 고정하고, 구현만 교체하는 템플릿 메서드 패턴이다.

같은 구조를 상속 없이, 람다(함수)를 주입하는 방식으로 표현할 수도 있다.

```kotlin
class ProductConverter<R>(
  private val doOnPBProduct: (PBProduct) -> R?,
  private val doOnImportedProduct: (ImportedProduct) -> R?,
  private val doOnLocalProduct: (LocalProduct) -> R?,
  private val convertRemoved: Boolean,
  private val convertHidden: Boolean,
  private val convertBlocked: Boolean,
) {

  fun convert(products: List<Product>): List<R> =
    products.mapNotNull { product ->
      if (!convertRemoved && product.isRemoved) return@mapNotNull null
      if (!convertHidden && product.isHidden) return@mapNotNull null
      if (!convertBlocked && product.isBlocked) return@mapNotNull null

      when (product) {
        is PBProduct -> doOnPBProduct(product)
        is ImportedProduct -> doOnImportedProduct(product)
        is LocalProduct -> doOnLocalProduct(product)
      }
    }
}
```

```kotlin
class ProductConverterBuilder<R> {

    var doOnPBProduct: (PBProduct) -> R? = { null } // 기본 값을 정의해서 선택적으로 전달 가능
    var doOnImportedProduct: (ImportedProduct) -> R? = { null }
    var doOnLocalProduct: (LocalProduct) -> R? = { null }

    var convertRemoved: Boolean = false
    var convertHidden: Boolean = false
    var convertBlocked: Boolean = false

    fun build(): ProductConverter<R> =
        ProductConverter(
            doOnPB = doOnPB,
            doOnImported = doOnImported,
            doOnLocal = doOnLocal,
            convertRemoved = convertRemoved,
            convertHidden = convertHidden,
            convertBlocked = convertBlocked,
        )
}

fun <R> productConverter(block: ProductConverterBuilder<R>.() -> Unit): ProductConverter<R> =
    ProductConverterBuilder<R>().apply(block).build()
```

```kotlin
val productDtoV1Converter = productConverter<ProductDtoV1> {
    doOnPBProduct = { product -> ProductDtoV1(id = product.id) }
    doOnImportedProduct = { product -> ProductDtoV1(id = product.id) }
    doOnLocalProduct = { product -> ProductDtoV1(id = product.id) }

    convertRemoved = true
    convertHidden = false
    convertBlocked = false
}

val convertedProducts: List<ProductDtoV1> = productDtoV1Converter.convert(products)
```

```kotlin
val productDtoV2Converter = productConverter<ProductDtoV2> {
    // PB 상품을 개념적으로 지원하지 않는 컨버터
    // doOnPBProduct = { product -> ProductDtoV2(id = product.id) }
    doOnImportedProduct = { product -> ProductDtoV2(id = product.id) } 
    doOnLocalProduct = { product -> ProductDtoV2(id = product.id) }

    convertRemoved = true
    convertHidden = true
    convertBlocked = true
}

val convertedProducts: List<ProductDtoV2> = productDtoV2Converter.convert(products)
```

전통적인 템플릿 메서드 패턴을 함수형 스타일로 풀어낸 구조이다.
템플릿 메서드 패턴은 상속 기반이고 람다 방식은 조합 기반이다. 
**요즘은 조합을 더 선호하는 흐름이다.**

## 상속보다는 조합
상속(is-a)
```text
BaseClass
   ▲
   │
ChildClass
```
* 상속의 문제점 
  * 부모와 자식의 강한 결합
    * 부모 클래스를 변경하면 자식 클래스들에 영향이 간다.
  * 상속은 컴파일 타임에 구조가 결정되어 런타임에 변경이 어렵다.
  * 계층 구조가 깊어지면 파악이 어려워진다.

조합(has-a)
```text
Class
 ├─ ComponentA
 ├─ ComponentB
 └─ ComponentC
```
기능을 상속으로 물려받기보다 객체를 가지고 조합해서 동작을 구성하라는 설계 원칙이다.

* 조합의 장점
  * 원하는 구현체를 끼워넣을 수 있다.
  * 구현체는 런타임에 교체가 가능하다.
  * 객체의 내부 구현이 철저히 숨겨진다.

상속 코드
```kotlin
class ProductDtoV3TemplateConverter : ProductTemplateConverter<ProductDtoV3>() {
    override fun convertPBProduct() { /* 수정 */ }
    override fun convertImportedProduct() { /* 기존 로직 복붙 */ }
    override fun convertLocalProduct() { /* 기존 로직 복붙 */ }
}
```

조합 코드
```kotlin
val v3 = productConverter {
    doOnPBProduct = { /* 수정 */ }
    doOnImportedProduct = v1.doOnImportedProduct
    doOnLocalProduct = v1.doOnLocalProduct
}
```
PB 상품만 다르게 처리하고 싶은 요구사항이 있는 경우
상속 로직에서는 구조상 나머지도 다시 구현해야하지만
조합 로직에서는 부분만 교체할 수 있다. (변경의 유연성)

## 싱글톤(Singleton) 패턴
특정 클래스의 인스턴스가 단 하나만 존재해야 할 때 필요한 패턴이다.
(인스턴스를 새로 생성한다는 것은 자원(메모리, 시간)을 소모하는 것이다)

```java
public class Theme {
    private static Theme instance;
    private String themeColor;

    private Theme() {
        this.themeColor = "light"; // Default theme
    }

    public static Theme getInstance() {
        if (instance == null) {
            instance = new Theme();
        }
        return instance;
    }

    public String getThemeColor() {
        return themeColor;
    }
    public void setThemeColor(String themeColor) {
        this.themeColor = themeColor;
    }
}
```

```text
Theme.getInstance().setThemeColor("dark");
```
테마는 딱 하나만 존재해야하는 개념이다.
생성자를 사용할 수 없기 때문에 외부에서는 Theme 클래스의 인스턴스를 만들 수 없다.

## 상태(State) 패턴
객체의 내부 상태에 따라 동일한 메서드의 행동이 달라질 때 사용하는 패턴이다.
`play()`, `stop()` 같은 메서드가 현태 상태에 따라 다른 행동을 해야할 때 적합하다.

다음과 같은 상황에서 상태 패턴을 고려할 수 있다.
* 상태 값에 따라 if/else, switch, when 분기가 계속 늘어날 때
* 상태 전이 규칙이 복잡해질 때
* “이 상태에서 이 행동이 가능한가?”를 코드로 추적하기 어려워질 때
* 상태별로 검증, 정책이 다를 때

상태 패턴을 적용하지 않은 코드
```java
public class VideoPlayer {
    private String state;

    public VideoPlayer() {
        this.state = "Stopped";
    }

    public void play() {
        if (state.equals("Stopped")) {
            System.out.println("Starting the video.");
            state = "Playing";
        } else if (state.equals("Playing")) {
            System.out.println("Video is already playing.");
        } else if (state.equals("Paused")) {
            System.out.println("Resuming the video.");
            state = "Playing";
        }
    }

    public void stop() {
        if (state.equals("Playing")) {
            System.out.println("Pausing the video.");
            state = "Paused";
        } else if (state.equals("Paused")) {
            System.out.println("Stopping the video.");
            state = "Stopped";
        } else if (state.equals("Stopped")) {
            System.out.println("Video is already stopped.");
        }
    }

    public static void main(String[] args) {
        VideoPlayer player = new VideoPlayer();
        
        player.play();   // "Starting the video."
        player.play();   // "Video is already playing."
        player.stop();   // "Pausing the video."
        player.play();   // "Resuming the video."
        player.stop();   // "Pausing the video."
        player.stop();   // "Stopping the video."
        player.stop();   // "Video is already stopped."
    }
}
```
상태 패턴을 적용하지 않은 코드의 문제점
* 상태가 늘어날수록 if/else 분기가 늘어난다.
* 상태 별 로직이 여러 메서드에 흩어진다.
* Paused 상태에서 play 하면 뭐하지?를 코드 전체에서 찾아야 한다.

상태 패턴은 상태를 값(enum, 문자열)으로 관리하지 않는다.
상태 자체를 객체로 만들고 행동을 그 안에 넣는다.

상태 패턴을 적용한 코드
```java
public interface State {
    void play(VideoPlayer player);
    void stop(VideoPlayer player);
}
```
모든 상태가 가져야 할 행동의 공통 인터페이스

```java
public class StoppedState implements State {
    @Override
    public void play(VideoPlayer player) {
        System.out.println("Starting the video.");
        player.setState(new PlayingState());
    }

    @Override
    public void stop(VideoPlayer player) {
        System.out.println("Video is already stopped.");
    }
}

public class PlayingState implements State {
    @Override
    public void play(VideoPlayer player) {
        System.out.println("Video is already playing.");
    }

    @Override
    public void stop(VideoPlayer player) {
        System.out.println("Pausing the video.");
        player.setState(new PausedState());
    }
}

public class PausedState implements State {
    @Override
    public void play(VideoPlayer player) {
        System.out.println("Resuming the video.");
        player.setState(new PlayingState());
    }

    @Override
    public void stop(VideoPlayer player) {
        System.out.println("Stopping the video.");
        player.setState(new StoppedState());
    }
}
```
Stopped, Playing, Paused 상태를 문자열이나 enum이 아니라 클래스로 표현한다.
그리고 상태별 행동을 그 클래스 안에 넣는다.

상태 패턴을 적용하면 상태별 로직이 한 곳에 모인다.
각 상태 클래스가 “이 상태에서 가능한 행동”과 “다음 상태”를 스스로 알고 있다.
하나의 상태에서 다른 상태로 전환할 때 대상 상태의 새 객체를 생성자로 만들어 넣어준다.

```java
public class VideoPlayer {
    private State state;

    public VideoPlayer() {
        // 초기 상태
        this.state = new StoppedState();
    }

    public void setState(State state) {
        this.state = state;
    }

    public void play() {
        state.play(this);
    }

    public void stop() {
        state.stop(this);
    }
}public interface State {
    void play(VideoPlayer player);
    void stop(VideoPlayer player);
}
```

```java
public class Main {
    public static void main(String[] args) {
        VideoPlayer player = new VideoPlayer();
        
        player.play();   // "Starting the video."
        player.play();   // "Video is already playing."
        player.stop();   // "Pausing the video."
        player.play();   // "Resuming the video."
        player.stop();   // "Pausing the video."
        player.stop();   // "Stopping the video."
        player.stop();   // "Video is already stopped."
    }
}
```

## 옵저버(Observer) 패턴
옵저버(관찰자) 패턴은 한 객체의 상태가 변경되면
그 변화를 구독하고 있는 다른 객체들(Observer)에게 자동으로 알림을 보내는 패턴이다.

왜 옵저버 패턴이 필요할까?
옵저버 패턴이 없으면 보통 이런 구조가 된다

> “혹시 방 나왔어요?” <br>
“지금은요?” <br>
“지금은요?” <br>
“지금은요?” <br>

클라이언트가 주기적으로 물어보는 Polling 방식은 불필요한 요청이 계속 발생하여
네트워크, 자원이 낭비되며 응답 타이밍이 늦어질 수 있어 비효율적이다.

공인중개사 예제로 이해하기
* subscribe()      : 방 나오면 알려주세요
* notifyObservers(): 방 나왔어요!
* unsubscribe()    : 이제 괜찮아요. 안 알려주셔도 돼요

옵저버 패턴 구현하기
```java
public interface Subject {
    void subscribe(Observer observer);
    void unsubscribe(Observer observer);
    void notifyObservers(Event event);
}
```

```java
public interface Observer {
    void update(Event event);
}
```
인터페이스 이름은 `Listener`라는 네이밍을 사용하기도 한다.
함수 이름은 `on(행위)`으로 시작하는 네이밍을 사용하기도 한다. (onEventOccurred, onUpdated)

```java
public class RealEstateOffice implements Subject {

    private final List<Observer> observers = new ArrayList<>();

    @Override
    public void subscribe(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void unsubscribe(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers(Event event) {
        for (Observer observer : observers) {
            observer.update(event);
        }
    }

    // 방이 새로 나왔을 때 호출되는 메서드
    public void newRoomArrived(String roomInfo) {
        // 로직 수행

        Event event = new Event(
                "NEW_ROOM",
                "📢 공인중개사: 방 나왔어요! 👉 " + roomInfo
        );

        notifyObservers(event);
    }
}
```
옵저버 패턴을 사용하면 좋은 경우
* 구독자 수가 제한적
* 이벤트 발생 빈도가 적당
* 옵저버 로직이 가볍고 빠름
* 비동기 처리 가능

옵저버 패턴을 사용하면 좋지 않은 경우
* 구독자 수 수백~수천개가 되는 경우
  * 옵저버 패턴은 O(N) 구조로 모든 구독자를 순회하며 알림 메서드를 호출하기 때문에 구독자가 많아질수록 CPU, 메모리 자원이 소모된다.
* 이벤트 발생 빈도 높음
  * 구독자 수도 많은데 발생 빈도도 높다면 병목 현상이 발생할 수 있음
* 옵저버 안에서 DB/외부 호출
  * 네트워크 지연이나 데이터베이스 락(Lock) 등으로 인해 지연이 발생할 수 있다.
* 동기 호출 구조
  * 1번 구독자가 처리를 완료해야 2번 구독자에게 알림을 보낼 수 있는 경우 1번 구독자에서 시간이 오래 걸리면 뒤에 있는 모든 구독자는 대기하게 된다.
→ 이 경우는 메시지 큐 구조를 대신 고려할 수 있다.

## 프록시(Proxy) 패턴
프록시는 클라이언트와 실제 객체 사이에 존재하며
클라이언트는 실제 객체 대신 프록시 객체에 작업을 요청한다.
프록시 객체는 해당 요청에 대해 부가기능을 수행한다.

적용 사례
* 접근 권한 체크(Spring Security)
  * 클라이언트의 인증 상태나 권한을 확인하고 권한이 있을 경우에만 실제 객체의 메서드를 호출
* 캐싱(@Cacheable)
  * 프록시가 캐시 데이터를 가지고 있다면 실제 객체를 호출하지 않고 캐시된 데이터를 바로 반환
* 로깅
  * 클라이언트의 요청이 실제 객체의 특정 메서드를 호출하기 전후에 해당 요청 정보(메서드 이름, 인자, 실행 시간 등)를 로그로 기록
* 트랜잭션(@Transactional)

```java
// 클라이언트는 인터페이스에 의존하기 때문에 자신이 프록시와 통신하는지 실제 객체와 통신하는지 알 필요가 없다(알 수 없다)
public interface Service {
    void doSomething();
}

public class RealService implements Service {
    public void doSomething() {  }
}

// 프록시 객체는 실제 객체를 참조한다.
public class ProxyService implements Service {
    private final Service target;

    public void doSomething() {
        // before
        target.doSomething();
        // after
    }
}
```

### JDK Proxy vs CGLIB Proxy
프록시 패턴을 개발자가 직접 구현하지 않고 런타임에
프록시를 자동으로 일관되게 적용하기 위해 나온 기술이다.

JDK Proxy: 인터페이스 기반 프록시
```text
[ Client ]
    |
    v
[ Service 인터페이스 ]
    |
    v
[ JDK Proxy (가짜 구현체) ]
    | --> invocationHandler.invoke()
    v
[ RealService ]
```
JDK Proxy는 모든 메서드 호출을 InvocationHandler.invoke()로 위임한 뒤
그 안에서 실제 RealService 메서드를 실행한다.

JDK Proxy는 인터페이스가 반드시 필요하다는 단점이 있다.
프록시 때문에 인터페이스를 만드는 경우가 생기게된다.

CGLIB Proxy: 클래스 상속 기반 프록시
```text
[ Client ]
    |
    v
[ Proxy extends RealService (가짜 자식 클래스(CGLIB)) ]
    |
    v
[ RealService ]
```
CGLIB Proxy는 실제 클래스를 상속하여 프록시를 생성하는 방식이기 때문에 인터페이스가 없어도 적용할 수 있다.
그러나 Kotlin의 클래스는 기본적으로 final이어서 상속이 불가능하므로 CGLIB 프록시를 사용하려면 open 키워드를 사용하거나 
allOpen 플러그인을 통해 자동으로 상속 가능하게 설정해야 한다. (@Component, @Service, @Transactional등 애노테이션 붙은 클래스들을 자동으로 open으로 바꿔줌)

## 어댑터(Adapter) 패턴

