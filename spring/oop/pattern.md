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

    var doOnPB: (PBProduct) -> R? = { null } // 기본 값을 정의해서 선택적으로 전달 가능
    var doOnImported: (ImportedProduct) -> R? = { null }
    var doOnLocal: (LocalProduct) -> R? = { null }

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
