## 람다
람다는 함수를 변수에도 할당할 수 있고
메서드의 파라미터로 전달할 수 있고
메서드의 반환값으로도 사용항 수 있다.

메서드의 파라미터로 전달하려면 자료형 타입을 어떻게 해야할까?

**(자료형, 자료형) → 자료형**

위와 같이 기술한 형태와 같은 형식의 함수는 모두 파라미터로 받을 수 있다.

```kotlin
fun main() {
    // 고차 함수 형태로 넘기려면 일반 함수를 고차 함수로 변경해주는 연산자 :: 를 사용해야한다.
    b(::a) // b가 호출한 함수 a
}

fun a(str: String) {
    println("$str 함수 a")
}

fun b(function: (String)->Unit) {
    function("b가 호출한")
}
```
```text
b가 호출한 함수 a
```
파라미터로 넘길 함수를 굳이 이름까지 붙여 따로 만들 필요가 없다.
이때 **함수를 람다식으로 표현**할 수 있다.

### 람다식 작성법
파라미터로 받아온 문자열을 매칭해줄 변수 이름을 작성한다.
```kotlin
fun main() {
    b(::a) // b가 호출한 함수 a
   
    val c: (String)->Unit = { str: String -> // 자료형은 타입 추론으로 생략 가능 
        println("$str 함수 a")
    }
    b(c)
}

fun a(str: String) {
    println("$str 함수 a")
}

fun b(function: (String)->Unit) {
    function("b가 호출한")
}
```
```text
b가 호출한 함수 a
b가 호출한 함수 a
```

### 람다 사용 시 알아두면 좋은 규칙
```kotlin
val calculate: (Int, Int) -> Int = { a, b ->
  println(a)
  println(b)
  a + b // 마지막 구문인 a + b의 값을 Int로 반환함
} 
```
* 람다 함수도 여러 줄로 작성이 가능하다.
  * 여러 줄인 경우 마지막 구문의 결과 값을 반환한다.

```kotlin
val a: () -> Unit = { println("파라미터가 없다") }
```
* 파라미터가 없는 람다 함수는 실행할 구문들만 나열하면 된다.

```kotlin
val c: (String) -> Unit = { println("$it 람다함수") }
```
* 파라미터가 하나라면 파라미터 이름을 붙여줄 필요 없이 it 을 사용할 수 있다.

### 람다 호출 형태 정리
* `(T) -> Unit` 람다 
  * T를 람다의 인자로 받는 형태
  * 호출 시 객체를 명시적으로 전달해야 한다.
  * 람다 내부에서 객체는 `it` 또는 별도 파라미터 이름으로 접근한다.
```kotlin
val f: (User) -> Unit = { user ->
    user.name = "Belluga"
}

f(user)   // 호출 방식
```
함수 호출 관점에서 User를 인자로 넘겨 실행한다.

* `T.() -> Unit` 람다
  * T를 람다의 리시버(this)로 사용하는 형태
  * 호출 시 객체가 수신자(receiver) 가 된다
  * 람다 내부에서 this == T 이므로 멤버에 바로 접근 가능하다
```kotlin
val f: User.() -> Unit = {
    name = "Belluga"
}

user.f()   // 호출 방식
```
함수 호출 관점에서 User 객체에 대해 블록을 실행한다.

## 스코프 함수
객체를 임시로 this 또는 it으로 받아 특정 범위(scope) 안에서 여러 작업을 깔끔하게 작성할 수 있게 해주는 함수이다.
스코프 함수를 이용하면 main 함수와 별도의 scope에서 인스턴스의 변수와 함수를 사용하므로 코드가 깔끔해진다는 장점이 있다.

Kotlin 스코프 함수들은 전부 각각 다른 질문에 답하는 도구라고 보면 이해하기 쉽다.
- apply
  - 이 객체를 어떻게 셋팅할까?
- run
  - 이 컨텍스트에서 어떤 계산을 할까?
- with
  - 이 객체를 기준으로 무슨 작업을 할까?
- also
  - 이 객체를 쓰면서 부가적으로 뭘 하지?
- let
  - 이 값으로 무엇을 할까?

### apply
```kotlin
inline fun <T> T.apply(block: T.() -> Unit): T {
    block()
    return this
}
```
```kotlin
val user = User().apply {
    name = "Belluga"
    age = 30
}
```
모든 타입 T에 대해 apply 함수를 추가한다.

리시버가 T인 람다
즉 블록 안에서 this가 T 객체이다. (User가 “인자”가 아니라 람다의 “this”가 된다)

- 목적: **객체를 설정/초기화**
- 반환값: **객체 자신**
- 아래 질문에 대한 답
  > 이 객체를 어떻게 셋팅할까?
  
```kotlin
inline fun <T> T.apply(block: (T) -> Unit): T {
    block()
    return this
}
```
만약 apply 함수가 위와 같았다면 아래와 같이 사용해야 했을 것이다. (-)
```kotlin
user.apply { u ->
    u.name = "Belluga"
}
```

### run
```kotlin
inline fun <T, R> T.run(block: T.() -> R): R {
    return block()
}
```
```kotlin
val isCheap = book.run {
    price < 10000
}
```
```kotlin
val label = Book("Book",10000).run {
    discount()
    "$name -$price원"
}
```
- 목적: 객체를 이용해 어떤 값을 계산
- 반환값: 마지막 표현식
- 아래 질문에 대한 답
  > 이 객체를 이용해서 뭘 얻을까?

### with
```kotlin
inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    return receiver.block()
}
```
run 과 동일하지만 인스턴스를 참조 연산자 대신 파라미터로 받는다는 차이점만 존재한다.
`a.run { .. }` → 확장 함수
`with(a) { .. }` → 일반 함수

### also/let
```kotlin
inline fun <T> T.also(block: (T) -> Unit): T {
    block(this)
    return this
}
```
```kotlin
inline fun <T, R> T.let(block: (T) -> R): R {
    return block(this)
}
```
also는 apply와 비슷하다 (처리가 끝나면 인스턴스를 반환)
let은 run과 비슷하다 (처리가 끝나면 최종값을 반환)

apply와 run이 참조 연산자 없이 인스턴스의 변수와 함수를 사용할 수 있었다면
also와 let은 파라미터를 통해 인스턴스를 넘긴것 처럼 `it`을 통해 인스턴스를 사용할 수 있다.

Why? 같은 이름의 변수나 함수가 **‘scope 바깥에 중복되어 있는 경우에 혼란을 방지하기 위함이다’**
also/let은 this가 아닌 it을 사용함으로써 스코프 안밖의 이름 충돌을 명시적으로 드러낼 수 있다.

```kotlin
fun main() {
    val price = 5000
    
    val a = Book("Book", 10000).apply {
        name = "[초특가] " + name
        discount()
    }
    a.run { 
      println("상품명: ${name}, 가격: ${price}") // 5000
    }
    
    a.let {
        println("상품명: ${it.name}, 가격: ${it.price}") // 8000
    }
}

data class Book(var name: String, var price: Int) {
    fun discount() {
        price -= 2000
    }
}
```
run 함수가 인스턴스 내의 price 속성보다 main 함수의 price 변수를 우선시한다.
이때 run을 대체하는 let을 사용할 수 있다.

## Kotlin DSL
DSL(Domain Specific Language)이란 특정 영역(domain)에서 최적화되어 사람이 읽기 쉬운 형태로 표현한 언어이다.

### Kotlin DSL의 핵심 철학
DSL은 구현부터 시작하지 않는다.
먼저 어떻게 쓰고 싶은지를 정한다.
```kotlin
page {
    header {
        title("Home")
    }
    body {
        section {
            text("Welcome")
        }
    }
}
```
사용자가 쓰게 될 코드 모양을 먼저 설계하고
그 코드를 가능하게 하는 API를 역으로 만든다.

### Kotlin DSL의 핵심 구성 요소
* 리시버 람다 (`T.() -> Unit`)
  * 블록 내부에서 this가 특정 타입을 가리키게 한다.
```kotlin
section {
    text("Hello") // this.text()
}
```

* 중첩 가능한 객체 구조
  * DSL 블록은 대부분 트리 구조로
  * 부모 → 자식으로 객체가 쌓인다
```text
Page
 └─ Body
     └─ Section
```

* 의미 있는 함수/타입 이름
  * 문법보다 의미가 먼저 읽히도록 설계한다.
  * doSomething() 보다 header {}, section {} 같은 이름을 사용한다.

### DSL 설계
* 블록 하나 = 리시버 람다 하나
```kotlin
fun page(block: Page.() -> Unit)
fun header(block: Header.() -> Unit)
fun body(block: Body.() -> Unit)
```
각 `{}` 블록은 보통 다음 형태로 매핑된다.

* 상태를 가지는 객체 설계
```kotlin
class Page {
    val headers = mutableListOf<Header>()

    fun header(block: Header.() -> Unit) {
        headers += Header().apply(block)
    }
}
```
* 객체 생성
* 설정(block 실행)
* 결과를 컬렉션에 

```kotlin
class Body {
    val sections = mutableListOf<Section>()

    fun section(block: Section.() -> Unit) {
        sections += Section().apply(block)
    }
}
```
이 패턴이 반복되며 DSL 전체 구조가 완성된다.

### 다른 DSL 예제
```kotlin
val mail = email {
    to = "test@test.com"
    title = "Hello"
}
```

```kotlin
class Email {
    var to: String = ""
    var title: String = ""
}

fun email(block: Email.() -> Unit): Email {
    return Email().apply(block)
}
```

## 제네릭
**제네릭이란?**

함수나 클래스를 선언할 때 고정적인 자료형 대신
실제 사용 시점에 결정되는 타입 파라미터를 받아 사용하는 방법이다.

- 코드의 재사용성을 높일 수 있다.
- 타입 안정성을 컴파일 시점에 보장할 수 있다.

### 타입 파라미터
타입 파라미터는 관례적으로 `T` 를 사용한다.
여러 개가 필요한 경우 `<T, U, V>` 형태로 선언할 수 있다.

### 제네릭 타입이 결정되는 시점
```kotlin
// 함수
fun <T> genericFunc(param: T): T

// 클래스
class GenericClass<T>(var prop: T)
```
`T` 는 실제 타입이 아니라 컴파일 시 실제 타입으로 치환되는 자리 표시자다.

```kotlin
fun <Int> genericFunc(param: Int): Int

class GenericClass<Int>(var pref: Int)
```
타입 파라미터에 실제 타입이 할당되면
제네릭을 사용하는 모든 코드는 해당 타입으로 컴파일된다.
→ 내부적으로는 Int 전용 함수 / 클래스처럼 동작한다.

### 타입 제한 (Upper Bound)
`<T: SuperClass>`
제네릭 타입을 특정 수퍼 클래스(또는 인터페이스)를 상속한 타입으로만 제한할 수 있다.

```kotlin
fun <T : SuperClass> genericFunc(param: T): T
```
T 는 반드시 SuperClass 를 상속한 타입이어야 한다.

### 타입 추론
* 함수 제네릭의 타입 추론
```kotlin
fun <T> genericFunc(param: T): T

genericFunc(1)
```
호출 시점에 타입이 자동 결정된다.

* 클래스 제네릭의 타입 추론
```kotlin
class GenericClass<T>(var pref: T)

GenericClass<Int>() // 타입 파라미터를 명시

GenericClass(1) // 생성자 인자를 통해 타입을 추론
```

## out / in (공변성, 반공변성)
자바, 코틀린은 무공변(불공변)으로
Dog가 Animal의 자식이더라도 List<Dog>는 List<Animal>에 대입할 수 없다. (완전히 남남이라고 선언)

왜그렇게 만들었을까?
런타임에 발생할 수 있는 사고를 막기 위해서이다.
```kotlin
val dogList: MutableList<Dog> = mutableListOf(Dog())

val animalList: MutableList<Animal> = dogList // 원래는 여기서 컴파일 에러가 나지만 허용된다고 가정해보자

animalList.add(Cat()) // animalList의 실제 정체는 dogList인데 고양이가 들어가는 사고가 발생한다.
val dog: Dog = animalList[1] // 꺼내보니 고양이가 들어있다. (ClassCastException)
```
하위 타입 컬렉션을 상위 타입 컬렉션으로 보는 순간
상위 타입의 다른 하위 객체가 들어갈 수 있다.
→ 강아지 상자를 동물 상자로 보는 건 위험하다.

```kotlin
val animalList: MutableList<Animal> = mutableListOf(Cat())
addDog(animalList) // 원래는 여기서 컴파일 에러가 나지만 허용된다고 가정해보자

// animalList = [Cat, Dog]
val dog: Dog = animalList[0] // 실제 정체는 Cat인데 Dog로 꺼내지는 사고가 발생한다. (ClassCastException)

fun addDog(list: MutableList<Dog>) {
  list.add(Dog())
}
```
Animal 리스트에 Dog를 추가하는 건 안전하지만
그 리스트를 Dog 리스트로 간주하는 순간 문제가 발생한다.
→ 동물 상자를 강아지 상자로 보는 것도 위험하다.

Kotlin에서는 아무런 정보가 없다면 모두 막아야 안전하지만
개발자가 **‘이 방향은 절대 안 쓸게’** 라고 약속하면 그 약속을 믿고 일부 대입을 허용한다.

### out — 내보내기 전용, 읽기 전용, 반환 전용
* out이 쓰이는 곳

| 위치                     | 의미                                     |
| ---------------------- |----------------------------------------|
| `interface Foo<out T>` | 클래스 내부에서 `T`를 반환(생산)으로만 사용             |
| `MutableList<out Animal>`     | 해당 위치(ex 함수 파라미터)에서 읽기 전용으로 사용 (쓰기 금지) |

* out이 선언되는 위치
```kotlin
interface Producer<out T> {
    fun produce(): T
}
```
* ️반환은 가능하다.
* 파라미터로는 받을 수 없다.
* 하위 타입 리스트를 상위 타입 리스트로 다뤄도 안전하다.
* List<Dog> → List<Animal> 대입 허용

대표적인 예: List
```kotlin
interface List<out T>
```
* get(index): T 있음 (T를 반환)
* add(T) 없음

```kotlin
val dogs: List<Dog> = listOf(Dog())
val animals: List<Animal> = dogs // 가능해진다.
val animal: Animal = animals[0] // 문제가 없다
```

```kotlin
fun printAnimals(list: MutableList<out Animal>) { // list를 읽기 전용으로만 사용하겠다는 의미
  val a: Animal = list[0]   // OK
  // list.add(Dog()) ❌
}
```
원래 MutableList<Animal>은 MutableList<Dog>을 받을 수 없다.
MutableList에 Cat이 들어갈 수 있다.

`printAnimals` 위 함수는 아래 파라미터를 모두 받을 수 있다.
* List<Dog>
* List<Cat>
* List<Animal>

참고)
```kotlin
// ❌ 이런 건 불가능, 이런 문법은 없다.
fun <out T> foo(): T
```

### in — 받기 전용, 쓰기 전용
* in이 쓰이는 곳
| 위치                       | 의미                                      |
| ------------------------ | --------------------------------------- |
| `interface Foo<in T>`    | 클래스 내부에서 `T`를 파라미터(소비)로만 사용             |
| `MutableList<in Animal>` | 해당 위치(ex 함수 파라미터)에서 쓰기 전용으로 사용 (읽기 제한됨) |

* in이 선언되는 위치
```kotlin
interface Consumer<in T> {
    fun consume(value: T)
}
```
* ️파라미터로는 받을 수 있다.
* 반환 타입으로는 사용할 수 없다.
* 상위 타입 리스트를 하위 타입 리스트처럼 다룰 수 있다.
* MutableList<Animal> → MutableList<in Dog> 대입 허용

대표적인 예: 
```kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}
``` 
compareTo는 T를 파라미터로 받기만 한다. (T를 소비)

```kotlin
class Dog : Comparable<Animal> {
    override fun compareTo(other: Animal): Int {
        return 0
    }
}
```
Dog는 Animal과 비교가 가능하다.
Comparable<in T> 이기 때문에 Comparable<Animal>을 Comparable<Dog>처럼 취급이 가능하다.
상위 타입을 소비하는 것이 안전하다.

```kotlin
fun addDog(list: MutableList<in Dog>) { // list를 넣기 전용으로만 사용하겠다는 의미
    list.add(Dog())
    // val dog: Dog = list[0] ❌ 불가
}
```
element를 넣는 용도로만 쓰겠다.
addDog는 아래를 모두 받을 수 있다.
* MutableList<Dog>
* MutableList<Animal>
* MutableList<Any>

왜 읽기가 제한될까? list에 Cat이 들어있을 수도 있다. (animals가 addDog 함수에 전달될 수도 있다)
`val animals: MutableList<Animal> = mutableListOf(Cat())`
