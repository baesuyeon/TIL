> 출처 <br />
> - https://d2.naver.com/helloworld/6080222 <br />
> - https://engineering.linecorp.com/ko/blog/reactive-streams-with-armeria-1


# 서블릿 컨테이너와 스프링 ✅
* 서블릿 컨테이너(WAS): HTTP 요청을 받고 응답하는 기본 인프라 (Tomcat, Jetty), WAS의 주요 엔진
* 스프링 프레임워크: 서블릿 컨테이너 안에서 비즈니스 로직을 처리하는 엔진

1. 사용자가 HTTP 요청을 보낸다.
2. 서블릿 컨테이너(Tomcat)가 요청을 스프링이 등록한 단 하나의 서블릿인 **DispatcherServlet**에게 전달한다.
3. 스프링이 적절한 컨트롤러를 찾아 로직을 실행한다.

❗DispatcherServlet의 관리 주체가 서블릿 컨테이너 → 스프링 컨테이너로 변경되었다.

* 전통적인 방식 (과거 WAR 배포)

서블릿 컨테이너가 주도권을 가졌다.

1. Tomcat이 먼저 실행
2. 설정 파일(web.xml) 로드: 톰캣이 DispatcherServlet 클래스 경로가 적혀있는 web.xml 파일을 읽는다.
3. 서블릿 생성: 톰캣이 리플렉션 기술을 사용해 DispatcherServlet 객체를 **직접 생성(New)** 하고 관리한다.
4. 스프링 구동: 생성된 DispatcherServlet이 내부적으로 스프링 컨테이너(ApplicationContext)를 만든다. (내부 `init()` 과정에서 ApplicationContext를 생성)

* 스프링 부트 방식 (현재 JAR 배포)

DispatcherServlet의 생성과 등록 주체는 스프링으로 이동하였다.
(서블릿 생명주기(init/service/destroy)는 여전히 서블릿 컨테이너가 담당)

1. Main 메서드 실행: 자바 애플리케이션(SpringApplication.run())이 먼저 실행
2. 스프링 컨테이너 생성: 스프링이 먼저 자기 자신(IoC 컨테이너)을 만든다.
3. DispatcherServlet 빈 생성: 스프링 부트가 자동으로 DispatcherServlet을 스프링 빈(@Bean)으로 등록한다.
4. 내장 서버 실행 및 등록: 스프링이 내부적으로 톰캣(내장 서버)을 실행시키면서 방금 자기가 만든 DispatcherServlet 빈을 톰캣에게 "이 서블릿을 써라"라고 직접 등록한다.

💡왜 이렇게 바뀌었을까?
스프링 부트가 이 방식을 택한 이유는 **"제어의 역전(IoC)을 웹 서버 영역까지 확장"** 하기 위해서이다.

* 설정의 단일화: 서버 설정(포트 번호 등)과 애플리케이션 설정을 모두 스프링 설정 파일(application.properties) 하나로 관리할 수 있다.
* 유연한 서블릿 등록: 서블릿 컨테이너의 복잡한 설정 없이도, 스프링 빈을 등록하듯 서블릿이나 필터를 코드만으로 자유롭게 추가할 수 있다.
* 편리한 배포: 별도의 톰캣 설치 없이 `java -jar` 한 줄로 어디서든 실행이 가능해졌다.


## 서블릿(Servlet)
```java
public interface Servlet {
    void service(ServletRequest req, ServletResponse res);
}
```
서블릿은 자바 진영에서 HTTP 요청/응답을 처리하기 위한 표준 인터페이스(기술)이다.

**서블릿의 라이프사이클**
서블릿은 `init()`, `service()`, `destroy()` 메서드 순으로 수행된다.
`init()` 메서드는 서블릿이 처음 호출되었을 때 단 한번만 수행된다.
`service()` 메서드는 사용자가 웹페이지를 호출할 때마다 수행된다.
`destroy()` 메서드는 해당 서블릿 인스턴스가 WAS에서 사라질 때 단 한번만 호출된다.

## DispatcherServlet
모든 HTTP 요청을 가장 먼저 받는 스프링 MVC의 “프론트 컨트롤러 서블릿”
DispatcherServlet 하나가 모든 요청을 먼저 받아 요청을 적절한 컨트롤러로 위임한다.

* 클라이언트 → WAS(Tomcat 등) 요청 
* WAS 내부의 서블릿 컨테이너가 URL 매핑 확인 
* DispatcherServlet으로 매핑된 경우 → DispatcherServlet 호출
  * 서블릿 컨테이너는 HttpServletRequest, HttpServletResponse 객체를 생성
  * DispatcherServlet은 최초 요청시 또는 애플리케이션 시작시 생성되며 생성시 `init()` 이 호출되어 스프링 컨텍스트와 각종 객체들을 초기화
    * 스프링 컨테이너(ApplicationContext)를 뒤져서 컨트롤러 목록, 뷰 리졸버, 핸들러 매핑 등을 찾아 자신의 내부 리스트에 등록
  * 서블릿 컨테이너는 요청을 처리할 스레드를 생성 
  * 해당 스레드에서 DispatcherServlet의 service() 메서드가 호출 
  * DispatcherServlet은 요청을 분석해 적절한 컨트롤러에 위임
* 응답 후 Request, Response 객체는 소멸되고 스레드는 스레드 풀로 반환

⚠️ 아주 중요한 포인트 <br >
서블릿 = 스레드 ❌ : 서블릿은 스레드가 아니다.
* 서블릿은 싱글톤 객체 
* WAS가 요청마다 스레드를 할당

# 서버 엔진과 스프링 WebFlux
* WebFlux의 탄생 배경

전통적인 서블릿 방식은 **요청마다 스레드를 할당**하므로 동시 접속자가 많아지면 스레드가 부족해지거나 메모리 점유율이 급증하는 문제가 있었다. 
이를 해결하기 위해 **"적은 스레드로 많은 동시 요청을 처리하자"** 는 목적으로 만들어졌다.
I/O 대기 시간이 많은 서비스(마이크로서비스 간 통신 등)에서 효율이 극대화된다.

* 논블로킹 HTTP 서버(Netty/Jetty/Undertow)
  * HTTP 요청 수신
  * 이벤트 루프(Event Loop) 기반으로 논블로킹 I/O 처리
    * 서버 엔진의 논블로킹 I/O의 대상
      * 네트워크 IO(네트워크를 통해 데이터를 읽고(Read) 쓰는(Write) 모든 과정), 소켓, TCP 연결, HTTP 패킷
  * 서블릿 표준에 의존하지 않음
* Spring WebFlux
  * 논블로킹 서버 위에서 비즈니스 로직을 처리하는 리액티브 웹 프레임워크
  * Reactive Streams 표준을 기반으로 비동기 로직 처리
  * WebFlux의 비동기 로직 처리의 대상
    * 서비스 로직, DB/외부 API 결과
  * 요청 라우팅, 컨트롤러 실행, HTTP 응답 구성을 담당

논블로킹 HTTP 서버(Netty/Jetty/Undertow): 서블릿 표준에 얽매이지 않고, 이벤트 루프(Event Loop) 기반으로 네트워크 I/O를 처리하는 고성능 엔진 <br>
스프링 WebFlux: 리액티브 스트림(Reactive Streams) 표준을 바탕으로 비동기 로직을 처리하는 프레임워크

* 사용자가 HTTP 요청을 보낸다.
* 서버 엔진(Netty)이 요청을 받고 이를 스프링이 제공하는 DispatcherHandler(HttpHandler)에게 전달한다.
* DispatcherHandler가 요청을 받아 적절한 컨트롤러(Handler)를 찾아 로직을 실행한다.
* 모든 과정은 **비동기(Mono/Flux)** 로 처리되어 결과가 나올 때까지 스레드가 대기하지 않고 다른 요청을 처리하러 떠난다.

## DispatcherHandler
모든 HTTP 요청을 가장 먼저 받는 스프링 WebFlux의 핵심 관문이다. (프론트 핸들러) 
DispatcherServlet과 유사한 역할을 하지만 서블릿 인터페이스를 상속받지 않는 일반 클래스이다.

클라이언트가 요청을 보내면 서버 엔진(Netty 등)이 요청을 수신한다.
HttpHandler가 서버 엔진별 독자적인 요청 형식을 스프링의 ServerHttpRequest로 변환한다.
DispatcherHandler가 이를 넘겨받아 HandlerMapping을 통해 처리할 대상을 찾는다.

HandlerAdapter가 실제 핸들러(Controller)를 호출한다.
결과값은 항상 **비동기 래퍼 클래스(Mono/Flux)** 로 반환되며, 데이터가 준비되는 시점에 응답이 완성된다.

## 스레드 모델의 차이
* Spring MVC (Thread Per Request) <br>
요청마다 스레드 풀에서 스레드를 하나씩 꺼내서 할당한다. <br>
DB나 외부 API 응답이 올 때까지 스레드는 차단(Blocking) 상태로 대기한다. (동시에 요청이 많아지면 스레드가 부족해짐)

* Spring WebFlux (Event Loop)
보통 CPU 코어 개수만큼의 아주 적은 스레드만 생성한다. <br>
요청을 받은 스레드는 작업을 던져두고 즉시 다음 요청을 받으러 간다. (Non-blocking)
작업이 완료되면 이벤트 루프가 이를 감지하여 최종 응답을 보낸다. (모든 작업이 Non-blocking일 때만 성립한다)

예시) 
상담원이 주문만 받고
“완성되면 알림 드릴게요” 하고 다음 손님 받음
커피가 완성되면 시스템이 알아서 손님에게 알림 
(이벤트 루프는 “완료되면 실행해달라”고 등록된 작업 목록을 계속 보고 있다가 완료 이벤트를 감지하면 
미리 등록해둔 후속 작업을 실행 후 응답을 조립 및 HTTP Response 전송 후 요청 종료)
상담원(스레드)이 놀지 않음, 기다림이 많은 상황에서 효율적

Spring MVC와 가장 큰 차이점 <br >
Spring MVC처럼 요청마다 스레드를 할당하는 모델이 아니라,
소수의 Event Loop 스레드가 다수의 요청을 동시에 관리하는 구조이다.
네트워크 I/O는 논블로킹 방식으로 처리되어 특정 요청의 I/O 대기 시간이 스레드를 점유하지 않는다.

## 컨텍스트 스위칭
CPU는 한번에 딱 하나의 스레드만 실행할 수 있다.
하지만 우리 눈에는 수백개의 스레드가 동시에 돌아가는 것처럼 보인다. CPU가 스레드들을 아주 빠르게 번갈아 가며 처리하기 때문이다.
CPU가 하던 일을 저장하고 새로운 일을 불러오는 과정을 컨텍스트 스위칭이라고 하는데
Spring MVC에서는 요청 하나당 스레드 하나를 할당하기 때문에 동시 접속자가 늘어날수록 생성된 스레드들 사이의 컨텍스트 스위칭 비용이 증가하여 CPU 자원이 낭비될 수 있다.
Spring WebFlux는 스레드 개수를 아주 적게(CPU 코어 수와 비슷)유지 하기 때문에 CPU가 스레드를 갈아치울 일이 훨씬 적어 컨텍스트 스위칭 오버에드를 줄인다.


## WebFlux 생명주기와 구성 요소
WebFlux는 서블릿 API를 사용하지 않으므로 init(), service(), destroy() 같은 서블릿 생명주기를 따르지 않는다. <br >
WebFlux에서는 HTTP 요청을 처리하는 흐름이 Reactor(Mono, Flux)의 리액티브 스트림 생명주기를 기반으로 동작한다.

* OnSubscribe: 구독자가 스트림을 구독할 때 발생 (요청 결과를 받겠다고 등록하는 단계)
  * 손님이 “커피 나오면 알려주세요” 하고 번호표 받음 
* OnNext: 데이터가 하나씩 방출될 때 발생
* OnComplete / OnError: 성공적으로 완료되거나 에러가 발생하여 스트림이 종료되었음을 알리는 신호
  * 커피 다 나왔어요(OnComplete), 재료가 떨어졌어요 (OnError)

## WebFlux 전체 요청 흐름

1. 사용자가 HTTP 요청을 보낸다.
2. 논블로킹 HTTP 서버(Netty)가 요청을 수신한다.
3. Netty의 **Event Loop 스레드**가 요청을 스프링이 제공하는 WebFlux HttpHandler에게 전달한다.
4. WebFlux의 프론트 컨트롤러인 **DispatcherHandler**가 요청을 받아 HandlerMapping을 통해 처리할 핸들러를 찾는다.
5. HandlerAdapter가 실제 핸들러(컨트롤러)를 호출한다.
6. 핸들러는 결과를 Mono / Flux 형태로 반환한다.
6. 데이터가 준비되는 시점에 논블로킹 방식으로 HTTP 응답이 전송된다.

모든 과정은 비동기적으로 연결된 리액티브 파이프라인 위에서 수행되며 처리 결과를 기다리기 위해 스레드가 점유되지 않는다.

### Event Loop 스레드의 역할

```text
EventLoop-1 → 요청 A, B, C
EventLoop-2 → 요청 D, E
DispatcherHandler (WebFlux의 프론트 컨트롤러)
DispatcherHandler는 WebFlux에서 모든 요청을 가장 먼저 받는 프론트 컨트롤러 역할의 스프링 빈이다. (어떤 컨트롤러를 부를지를 결정)
```

Event Loop 스레드는 단순히 요청을 전달하는 역할에 그치지 않는다.
* I/O 감시: 데이터베이스나 외부 API로부터 응답이 오는지 네트워크 소켓을 지속적으로 체크한다.
* 콜백 실행: 데이터가 준비되었다는 신호(이벤트)가 오면 하던 다음 단계의 로직(리액티브 연산, 응답 전송 등)을 이어서 수행한다.

### WebFlux 주의점 ⚠️
Event Loop 스레드를 절대 블로킹하면 안된다.
Event Loop 스레드를 블로킹하면 그 스레드가 담당하던 모든 연결의 요청 수신과 응답 전송이 동시에 멈추게된다.
블로킹 I/O(DB, JPA 등)는 별도 블로킹 전용 스레드 풀로 위임해야 한다.

### WebFlux의 가치
WebFlux의 진짜 가치는 DB보다 외부 I/O(API, 메시지, 스트리밍)에 있다.
**JDBC(Java Database Connectivity)** 라는 표준 자체가 Blocking 방식으로 
관계형 DB(Oracle, MySQL 등) 드라이버는 쿼리를 던지면 결과를 받을 때까지 스레드를 잡아둔다.
(WebFlux를 써도 밑바닥(DB 드라이버)이 Blocking이면 결국 스레드는 대기하게 된다. (최근에는 R2DBC라는 Non-blocking 드라이버가 나왔지만, 아직 JDBC만큼 성숙하지는 않았다.)
또한 DB는 쿼리 결과 전체를 한번에 다 받으려고 한다.
WebFlux는 데이터가 하나 올 때마다 바로바로 소비자에게 던져주는 '구독(Subscribe)' 모델에 최적화되어 있으며 이것이 WebFlux가 따르는 Reactive Streams의 핵심이다.

API의 경우 이미 고성능 Non-blocking 지원이(WebClient, gRPC)가 아주 잘 발달되어 있어
요청부터 응답까지를 블로킹 없이 연결하는 엔드투엔드 논블로킹 처리가 쉽고 효과적이다.

따라서 MSA 간 통신에서 WebFlux가 훨씬 더 큰 가치를 인정받고 있다.
무조건 MVC보다 좋다기보다 I/O 비중이 높고 동시성이 중요한 서비스에 적합하다.

## WebFlux 요청 흐름
Spring WebFlux는 더이상 WAS가 필요하지 않아 Netty를 사용하며
Project Reactor를 통해서 Reactive Programming을 지원한다.

Project Reactor는 Reactive Streams의 구현체이다.
Reactive Streams는 단순히 JVM 기반에서 Async Non-Blocking 처리를 위한 스펙을 명세한 것이다.

```text
Client
 → Netty (Event Loop), 커피 주문을 넘기자마자 바로 다음 손님을 받는다.
 → HttpHandler, HTTP 요청/응답을 ServerHttpRequest ServerHttpResponse로 랩핑
 → WebFilter Chain, 공통 처리, 인증/인가, 로깅, 헤더 처리, 공통 전/후처리
 → WebHandler, =DispatcherHandler 요청을 어떤 핸들러(@Controller, HandlerFunction) 가 처리할지 결정
 → HandlerFunction / @Controller
 → Mono / Flux
 → Spring subscribe
 → Response write
```

Spring WebFlux는 기존 Spring MVC와 동일하게 애너테이션 기반으로 설정할 수 있다. 
(@RequestMapping, @ResponseBody, @PathVariable 등 애너테이션이 동일하게 동작한다)
결정적인 차이점은 바로 반환 타입이다.

Spring MVC에서는 String, List와 같은 일반적인 Plain Object를 반환할 수 있지만
Spring WebFlux에서는 이러한 객체를 직접 반환할 수 없고 반드시 Publisher 타입으로 감싸서 반환해야 한다.

Mono와 Flux는 모두 Project Reactor의 Publisher Object이다.
컨트롤러가 Publisher를 반환해야만 Subscriber가 처리 가능한 만큼만 데이터를 요청하는 Backpressure(request(n), cancel) 메커니즘을 적용할 수 있다.

Spring WebFlux에서는 컨트롤러가 Publisher를 반환하면 
HTTP 응답을 실제로 쓰는 시점에 프레임워크가 이를 subscribe한다.
이 과정에서 Netty는 HTTP 응답 버퍼 상태를 기준으로 request(n)을 조절하며 이를 통해 네트워크 레벨의 Backpressure를 구현한다.

```text
Mono<T>  : 0 또는 1개의 데이터 (단일 결과값을 비동기로 받는지)
Flux<T>  : 0 ~ N개의 데이터 (데이터가 스트림 형태로 흐름)
```

Mono<List<T>> vs Flux<T>
둘다 여러 개의 요소가 포함될 수 있음을 의미하지만 용도가 조금 다르다.
Mono<List<T>>: 여러개의 요소가 한번에 반환됨을 의미
Flux<T>: 여러개의 요소가 Stream 형태로 반환됨

## Reactive Streams 란?
기존 옵저버 패턴에서는 publisher가 subscriber의 상태를 고려하지 않고 데이터를 전달하는데만 충실했다.
만약 publisher가 1초 동안 100개의 메시지를 보내는데 subscriber는 1초에 10개밖에 처리하지 못한다면 
큐(queue)를 이용해서 대기 중인 이벤트를 저장해야 한다.

하지만 서버의 메모리는 한정적이기 때문에 
* 고정 길이 버퍼를 사용할 때
  * 버퍼가 가득 차면 신규 메시지를 거절하거나 재요청해야 하고
  * 이 과정에서 추가적인 네트워크 통신과 CPU 연산 비용이 발생한다.

* 가변 길이 버퍼를 사용할 때
  * 버퍼 크기에 제한이 없어 
  * out of memory 에러가 발생할 수 있다.

Backpressure는 이러한 문제를 해결하기 위한 메커니즘으로
Publisher가 데이터를 일방적으로 전달하는 대신 Subscriber가 처리 가능한 만큼만 요청하도록 한다.
데이터의 크기를 Subscriber가 결정한다.

## Reactive Streams API
```text
public interface Publisher<T> {
   public void subscribe(Subscriber<? super T> s);
}
 
public interface Subscriber<T> {
   public void onSubscribe(Subscription s);
   public void onNext(T t);
   public void onError(Throwable t);
   public void onComplete();
}
 
public interface Subscription {
   public void request(long n);
   public void cancel();
}
```
* Publisher에는 Subscriber의 구독을 받기 위한 subscribe API 하나만 존재한다.
* Subscriber에는 
  * 전달 받은 데이터를 처리하기 위한 `onNext`
  * 에러를 처리하는 `onError`
  * 작업 완료 시 사용하는 `onComplete`
  * 그리고 매개 변수로 Subscription을 받는 `onSubscribe` API가 있다.
    * Subscription은 n개의 데이터를 요청하기 위한 `request`와 
    * 구독을 취소하기 위한 `cancel` API가 있다.

Reactive Streams에서 위 API를 사용하는 흐름
* Subscriber가 subscribe()를 호출하여 Publisher에게 구독을 요청한다.
* Publisher는 onSubscribe()를 호출하여 Subscriber에게 Subscription을 전달한다.
* 이제 Subscription은 Subscriber와 Publisher 간 통신의 매개체가 된다. Subscriber는 Publisher에게 직접 데이터 요청을 하지 않고 Subscription을 통해서만 통신한다.
* Subscription의 Subscription.request(n)을 호출하여 Publisher에게 자신이 처리 가능한 만큼의 데이터 개수를 요청한다.(ex request(3) or 구독 취소 cancel)
* Publisher는 요청받은 개수만큼 onNext()를 호출해 데이터를 전달한다.
* 모든 데이터 처리가 완료되면 onComplete(), 처리 중 오류가 발생하면 onError() 시그널을 전달한다.

Reactive Streams에서는 Subscriber가 request(n)을 호출하기 전까지 Publisher는 어떤 데이터도 전송할 수 없다.
즉 onNext()는 request(n) 이전에 호출될 수 없다.

Backpressure는 Subscriber가 처리 가능한 만큼만 데이터를 요청하는 메커니즘이다.
맛집에서 음식을 막 내보내는게 아니라 손님이 "이제 한 접시 더 주세요"라고 말할 때만 내보내는 것이다.

Reactive Streams의 구현체 살펴보기
* 순수 연산: RxJava, Reactor Core
* 저장소의 데이터를 Reactive Streams로 조회: ReactiveMongo
* 웹 프로그래밍과 연결된 Reactive Streams: Armeria, Spring WebFlux

### Cold / Hot Publisher
* Cold Publisher
  * 구독(subscribe)이 발생할 때마다 데이터 생산이 새로 시작되는 Publisher이다.
  * 각 구독자는 독립적인 데이터 흐름을 가진다.
  * 예: HTTP 요청(WebClient), DB 조회

* Hot Publisher
  * 이미 생성 및 전송되고 있는 데이터를 구독자가 중간부터 받아보는 Publisher이다.
  * 예: 실시간 이벤트 스트림, 센서 데이터