# 서블릿 필터, 인터셉터, AOP 비교
**공통 관심사(cross-cutting concern)** 의 분리
서블릿 필터, 인터셉터, AOP 기능들은 모두 어떤 행동을 하기 전에 먼저 실행하거나, 실행한 후에 추가적인 행동을 할 때 사용되는 기능들이다.

<img width="588" height="239" alt="Image" src="https://github.com/user-attachments/assets/958c3809-ea58-4a3b-a30a-794f90f3edfe" />

## 서블릿 필터
웹 애플리케이션의 최전방 수문장
공통 관심사는 스프링의 AOP로도 해결할 수 있지만, 웹과 관련된 공통 관심사는 지금부터 설명할 서블릿 필터 또는 스프링 인터셉터를 사용하는 것이 좋다.

| 항목 | Tomcat(Servlet API) | Netty(Spring WebFlux) |
| --- | --- | --- |
| 대응 개념 | Servlet Filter | WebFilter |

서블릿 필터는 스프링 컨테이너가 아닌 **서블릿 컨테이너(Tomcat 등)** 에 의해 관리되는 기술이다. 스프링에 요청이 도달하기 전/후에 가장 먼저 실행된다.
Spring Boot에서는 스프링 코드로 등록해서 톰캣에 연결한다.

* 실행 주체 = 톰캣
* 등록,설정 = 스프링 애플리케이션 코드

Spring Boot는 **웹 컨테이너(Tomcat)를 코드로 설정**해주는 프레임워크이다.
Spring → 톰캣 : "이 필터를 톰캣에 등록해줘"

* 핵심 특징
  * Request/Response 교체 가능
    * `chain.doFilter(request, response)` 호출 시 다른 객체를 넘길 수 있는 유일한 계층이다.
    * 보안 취약점 필터링 (XSS 방어)
      * 사용자가 입력한 값에 `<script>` 같은 위험한 코드가 섞여 있을 때 필터에서 이를 감지하여 `&lt;script&gt;` 로 치환된 값을 내뱉는 Wrapper로 바꿔치기해서 컨트롤러로 보내면 컨트롤러는 안전한 값만 받게된다.
    * 바디 내용 캐싱 (Logging)
      * HTTP의 Body(스트림)는 한번 읽으면 사라진다.
      * 필터에서 로그를 남기려고 바디를 읽어버리면 컨트롤러는 빈 바디를 받게되는 문제가 있다.
      * `ContentCachingRequestWrapper`로 원본을 감싸서 `chain.doFilter(wrapper, response)`를 호출해서 컨트롤러로 보낸다.
        * `ContentCachingRequestWrapper`는 읽을 때 복사본을 만드는 방식
      * 컨트롤러에서 `@RequestBody` 등을 통해 바디를 읽을 때 Wrapper가 데이터를 내부 메모리에 캐싱(복사)한다.
      * 컨트롤러 작업이 끝나고 다시 필터로 제어권이 돌아왔을 때 Wrapper에 복사해둔 데이터(캐시)를 꺼내 로그를 찍는다.

즉 전역적 처리, 웹 보안(XSS 방어), 인코딩 변환, 이미지 압축 등 스프링과 무관한 처리에 적합하다.

**필터 흐름**
```text
HTTP 요청 → WAS → 필터 → 디스패처 서블릿 → 컨트롤러

모든 고객의 요청 로그를 남기는 요구사항이 있다면 필터를 사용하면 된다. 
참고로 필터는 특정 URL 패턴에 적용할 수 있다. `/*` 이라고 하면 모든 요청에 필터가 적용된다.

필터 제한
```text
HTTP 요청 → WAS → 필터 → 서블릿 → 컨트롤러 // 로그인 사용자
HTTP 요청 → WAS → 필터(적절하지 않은 요청이라 판단, 서블릿 호출X) // 비 로그인 사용자
```

필터에서 적절하지 않은 요청이라고 판단하면 거기에서 끝을 낼 수도 있다. 
그래서 로그인 여부를 체크하기에 딱 좋다.

필터 체인
```text
HTTP 요청 → WAS → 필터1 → 필터2 → 필터3 → 서블릿 → 컨트롤러
```

필터 인터페이스

```java
public interface Filter {

    public default void init(FilterConfig filterConfig) throws ServletException {
    }

    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    public default void destroy() {}
}
```

필터 인터페이스를 구현하고 등록하면 **서블릿 컨테이너**가 필터를 싱글톤 객체로 생성하고, 관리한다.

- init() : 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출된다.
- doFilter() : url-pattern에 맞는 모든 HTTP 요청이 디스패처 서블릿으로 전달되기 전에 웹 컨테이너에 의해 실행되는 메소드이다.
- destroy(): 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.

- **`hain.doFilter(request, response);`**
    - 이 부분이 가장 중요하다. **다음 필터가 있으면 필터를 호출**하고, **필터가 없으면 서블릿을 호출**한다. 만약 이 로직을 호출하지 않으면 다음 단계로 진행되지 않는다.

필터에는 다음에 설명할 스프링 인터셉터는 제공하지 않는, 아주 강력한 기능이 있는데

chain.doFilter(request, response); 를 호출해서 다음 필터 또는 서블릿을 호출할 때 

**request ,response 를 다른 객체로 바꿀 수 있다.**

❗️WebFlux는 서블릿이 없지만 필터 역할은 여전히 필요하기 때문에
WebFlux는 서블릿 표준을 따르지 않지만 WebFilter를 통해 동일한 역할을 수행한다. 
WebFlux에서는 인터셉터 개념이 없으므로 WebFilter가 모든 가로채기 역할을 통합해서 관리한다.

# 인터셉터

<img width="785" height="304" alt="Image" src="https://github.com/user-attachments/assets/8c03a0d4-ccd9-4b7e-acf6-276a7e8f1b82" />

서블릿 필터가 서블릿이 제공하는 기술이라면, 스프링 인터셉터는 스프링 MVC가 제공하는 기술이다. 

둘다 웹과 관련된 공통 관심 사항을 처리하지만, 적용되는 순서와 범위, 그리고 사용방법이 다르다.

❗️ 특별히 필터를 꼭 사용해야 하는 상황이 아니라면 인터셉터를 사용하는 것이 더 편리하다.

Dispatcher Servlet이 Controller를 호출하기 전, 후에 인터셉터가 끼어들어 요청과 응답을 참고하거나 가공할 수 있다.

스프링 인터셉터 흐름
```text
HTTP 요청 → WAS → 서블릿 필터 → 서블릿(디스패처 서블릿) → 스프링 인터셉터 → 컨트롤러
```

스프링 인터셉터 인터페이스
스프링의 인터셉터를 사용하려면 HandlerInterceptor 인터페이스를 구현하면 된다.
```java
public interface HandlerInterceptor {

    // 컨트롤러가 호출되기 전에 실행된다.
    default boolean preHandle(
			HttpServletRequest request, HttpServletResponse response,
      Object handler
    ) throws Exception {}
    
    // 컨트롤러가 호출된 후에 실행된다.
    default void postHandle(
        HttpServletRequest request, HttpServletResponse response,
        Object handler, @Nullable ModelAndView modelAndView
    ) throws Exception {}

default void afterCompletion(
HttpServletRequest request, HttpServletResponse response,
Object handler, @Nullable Exception ex
) Exception {}
}
```
인터셉터는 컨트롤러 호출 전( preHandle ), 호출 후( postHandle ), 요청 완료 이후( afterCompletion )와 같이 단계적으로 잘 세분화 되어 있다. 

- 서블릿 필터의 경우 단순히 request , response 만 제공했지만, 인터셉터는 어떤 컨트롤러( handler )가 호출되는지 호출 정보도 받을 수 있다.

`postHandle` : 컨트롤러에서 예외가 발생하면 postHandle 은 호출되지 않는다.
`afterCompletion` : afterCompletion 은 항상 호출된다. 이 경우 예외( ex )를 파라미터로 받아서 어떤

예외가 발생했는지 로그로 출력할 수 있다.

**afterCompletion은 예외가 발생해도 호출된다.**

예외가 발생하면 postHandle() 는 호출되지 않으므로 예외와 무관하게 공통 처리를 하려면
`afterCompletion()` 을 사용해야 한다.

```java
@Slf4j
public class LogInterceptor implements HandlerInterceptor {

    public static final String LOG_ID = "logId";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();

        request.setAttribute(LOG_ID, uuid);

        // @RequestMapping인 경우 HandlerMethod handler가 사용된다.
        if (handler instanceof HandlerMethod) {
            // 호출할 컨트롤러 메서드의 모든 정보가 포함되어 있다.
            HandlerMethod hm = (HandlerMethod) handler;
        }

        log.info("REQUEST [{}][{}][{}]", uuid, requestURI, handler);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle [{}]", modelAndView);
    }

    @Override
    public void afterCompletion(
            HttpServletRequest request,
            HttpServletResponse response,
            Object handler, Exception ex
    ) throws Exception {
        String requestURI = request.getRequestURI();
        String logId = (String) request.getAttribute(LOG_ID);

        log.info("REQUEST [{}][{}][{}]", logId, requestURI, handler);

        if(ex != null) {
            log.error("afterCompletion error!!!", ex);
        }
    }
}
```
return true
: true 면 정상 호출이다. 다음 인터셉터나 컨트롤러가 호출된다.

## **인터셉터에서 body를 읽으면 왜 컨트롤러에서 못읽을까?**

HTTP request body는 ‘스트림’이라서 한 번 읽으면 끝이다.

**그래서 인터셉터에서 body를 읽으면 안 된다**

**인터셉터의 책임**

- 인증
- 권한
- 메트릭
- URI / 헤더 검사

❌ body 파싱

이건 **설계적으로 금지된 영역**에 가깝다.

로그를 찍고 싶다면 필터를 사용해야 한다.

# 서블릿 필터 vs 인터셉터
서블릿 필터와 인터셉터 실행 흐름 보기

<img width="799" height="529" alt="Image" src="https://github.com/user-attachments/assets/10adc092-c11a-4671-a5ce-5d4952538244" />

출처: https://sivakumar-hybris-dev.medium.com/spring-framework-recap-mvc-module-864d5f89b224

원래 postHandle은 ModelAndView 객체를 받아서 뷰에 전달할 데이터를 추가로 가공하는 용도이기 때문에
JSON을 반환할 때는 postHandle가 사실상 쓸모가 거의 없어지고 afterCompletion는 JSON이 클라이언트에게 완전히 전송된 후 호출된다.

```text
HTTP 요청
   ↓
Tomcat (Servlet Container)
   ↓
Servlet Filter (전역적인 웹 보안, 로깅)
   ↓
DispatcherServlet
   ↓
HandlerMapping (어떤 컨트롤러로 갈지 결정)
   ↓
HandlerExecutionChain (Handler + Interceptors)
   ↓ -------------------------
   ↓  [Interceptor: preHandle]  ← 가로채기 1 (로그인 체크 등)
   ↓ -------------------------
HandlerAdapter (컨트롤러 실행을 도와주는 도구)
   ↓
Controller
   ↓ -------------------------
   ↓  [Interceptor: postHandle] ← 가로채기 2 (모델 값 가공), 아직 View가 생성되기 이전에 호출
   ↓ -------------------------
DispatcherServlet (뷰 렌더링)
   ↓ -------------------------
   ↓  [Interceptor: afterCompletion] ← 가로채기 3 (리소스 정리)
   ↓ -------------------------
Servlet Filter (나가는 길)
   ↓
HTTP 응답
```
Servlet 기반(Spring MVC) 요청 흐름

```text
HTTP Request
   ↓
Netty (Event Loop 기반 서버)
   ↓
HttpHandler
   ↓
WebHandler
   ↓
WebFilterChain (WebFilter들의 집합)
   ↓ -------------------------
   ↓  [WebFilter 1, 2, 3...]    ← 여기서 MVC의 Filter+Interceptor 역할 수행
   ↓ -------------------------
DispatcherHandler (MVC의 DispatcherServlet 역할)
   ↓
HandlerMapping
   ↓
HandlerAdapter
   ↓
Controller (비동기(Non-blocking)로 실행)
   ↓
HTTP Response (Mono/Flux 반환)
```
WebFlux 요청 흐름 (좀 더 자세히)
Interceptor라는 개념이 따로 없다.
대신 **WebFilter**가 필터와 인터셉터의 역할을 모두 수행하며 실행 체인(WebFilterChain) 속에서 흐름을 제어한다.

필터는 기본적으로 스프링과 무관하게 전역적으로 처리해야 하는 작업들을 처리할 수 있다.

- 보안 검사
- 이미지, 데이터 압축
- 문자열 인코딩
- 스프링 앞단 서블릿 영역에서 관리되기 때문에 필터에서 예외 처리가 필요하다.

인터셉터에서는 클라이언트의 요청과 관련되어 전역적으로 처리해야 하는 작업들을 처리할 수 있다.

- 세부적으로 적용해야 하는 인증이나 인가
- 필터와 다르게 request, response를 조작할 수 없다.


공통 관심사는 스프링의 AOP로도 해결할 수 있지만 웹과 관련된 공통 관심사는 지금부터 설명할 서블릿 필터 또는 스프링 인터셉터를 사용하는 것이 좋다.
**웹과 관련된 공통 관심사**를 처리할 때는 HTTP의 헤더나 URL의 정보들이 필요한데, 
서블릿 필터나 스프링 인터셉터는 `HttpServletRequest` 를 제공한다.

서블릿 필터의 경우 단순히 request , response 만 제공했지만, 인터셉터는 어떤 컨트롤러( handler )가 호출되는지 호출 정보도 받을 수 있다.
스프링 인터셉터에도 URL 패턴을 적용할 수 있는데, 서블릿 URL 패턴과는 다르고, 매우 정밀하게 설정할 수 있다.

