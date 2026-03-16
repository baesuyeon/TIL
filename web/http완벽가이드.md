# 1장 HTTP 개관

HTTP는 웹에서 클라이언트와 서버가 데이터를 주고받기 위해 사용하는 애플리케이션 계층 프로토콜이다.
HTTP는 신뢰성 있는 전송 프로토콜(TCP) 위에서 동작하기 때문에 데이터가 전송 중

* 손상되거나
* 순서가 바뀌거나
* 중복 전달되는 문제를 걱정할 필요가 없다.

HTTP는 웹의 멀티미디어 배달부 역할을 하며 다양한 데이터 타입을 전달할 수 있다.

## 웹 리소스

웹 서버는 웹 리소스를 관리하고 제공한다.
리소스는 크게 두가지로 나뉜다.

**정적 리소스**
서버에 저장된 파일을 그대로 전달한다.

* 텍스트 파일
* HTML 파일
* JPEG 이미지 파일
* AVI 동영상 파일

**동적 리소스**
서버 프로그램이 요청에 따라 동적으로 생성하는 콘텐츠

* 검색 결과 페이지
* 로그인 사용자별 페이지
* API 응답 JSON

## MIME 타입

인터넷에서는 수천 가지 데이터 타입이 존재하기 때문에 HTTP는 MIME 타입을 사용해 데이터 타입을 명시한다.
서버는 응답 메시지의 `Content-Type` 헤더에 데이터 타입을 포함한다.

`Content-type: image/jpeg`
브라우저는 이 MIME 타입을 보고 데이터를 어떻게 처리할지 결정한다.

대표적인 MIME 타입

| 타입               | 설명       |
|------------------|----------|
| text/html        | HTML 문서  |
| text/plain       | 일반 텍스트   |
| image/jpeg       | JPEG 이미지 |
| image/gif        | GIF 이미지  |
| application/json | JSON 데이터 |

## URL

URL은 웹에서 리소스의 위치와 접근 방법을 나타내는 주소이다.

기본 구조
`scheme://host:port/path?query#fragment`

예
`http://www.example.com:80/index.html?name=bob#section1`

| 구성요소     | 설명                                             |
|----------|------------------------------------------------|
| scheme   | 리소스에 접근하기 위해 사용되는 프로토콜, 보통 HTTP 프로토콜 (http://) |
| host     | 서버 주소                                          |
| port     | 서버 포트                                          |
| path     | 서버 리소스 위치                                      |
| query    | 추가 요청 파라미터                                     |
| fragment | 문서 내부 위치                                       |

## HTTP 트랜잭션
HTTP 통신은 요청(Request) 과 응답(Response) 으로 이루어진다.
이를 HTTP 트랜잭션이라 한다.

요청 메시지
```text
GET /test/hi-there.txt HTTP/1.0
Accept: text/*
Accept-Language: en, fr
```

응답 메시지
```text
HTTP/1.0 200 OK
Content-Type: text/plain
Content-length: 19

Hi! I'm a message!
```

### HTTP 메시지 구조
HTTP 메시지는 다음과 같은 구조를 가진다.
```text
<시작줄>
<헤더>
<빈 줄>
<본문>
```

**시작줄**
요청
```text
GET /index.html HTTP/1.1
```

응답
```text
HTTP/1.1 200 OK
```

**헤더**
헤더는 메시지의 메타데이터를 제공한다.
시작 줄 이후에 0개 이상의 헤더 필드가 이어진다.
`:`로 구분되는 이름-값으로 구성된다
헤더는 빈 줄로 끝난다.
```text
Content-Type: text/html
Content-Length: 100
```

**본문**
본문은 실제 전달되는 데이터이다.
* HTML
* JSON
* 이미지
* 파일

## TCP 커넥션
```text
HTTP                    - 애플리케이션 계층
TCP                     - 전송 계층
IP                      - 네트워크 계층
네트워크를 위한 링크 인터페이스 - 데이터 링크 계층
물리적인 네트워크 하드웨어     - 물리 계층
```

HTTP는 자신의 메시지 데이터를 전송하기 위해 TCP를 사용한다.
위 메시지가 TCP 커넥션을 통해 어떻게 전송되는지 알아보자.

TCP는 다음을 제공한다.
* 오류없는 데이터 전송
* 순서에 맞는 전달 (데이터는 언제나 보낸 순서대로 도착한다)
* 조각나지 않는 데이터 스트림

TCP/IP는 어떤 종류의 컴퓨터든 네트워크든 서로 신뢰성있는 의사소통을 하게해준다.

클라이언트가 서버에 메시지를 전송하기 위해 IP 주소와 포트번호를 사용해 클라이언트와 서버 사이의 TCP/IP 커넥션을 맺어야 한다.
IP 주소와 포트 번호는 URL을 통해 알아낼 수 있다.

## HTTP 요청 과정
브라우저가 웹 페이지를 요청하는 과정

* 웹브라우저는 서버의 URL에서 호스트 명을 추출한다.
* 웹브라우저는 DNS를 조회하여 서버의 호스트 명을 IP로 변환한다.
* 웹브라우저는 URL에서 포트번호(없다면 80)를 추출한다.
* 웹브라우저는 웹 서버와 TCP 커넥션을 맺는다. (TCP 3-way handshake)
* 웹브라우저는 서버에 HTTP 요청을 보낸다.
* 서버는 웹브라우저에 HTTP 응답을 돌려준다.
* 커넥션이 닫히면 웹브라우저 문서를 보여준다.

## 프록시
```text
Client → Proxy → Server
```
프록시 서버는 클라이언트와 서버 사이에 위치한 중개 서버이다.

프록시 역할
* 보안
* 캐싱
* 로깅
* 접근 제어

웹 캐시는 특수한 프록시 서버이다.
캐시에 리소스가 존재하면 원 서버에 요청하지 않고 캐시에서 응답한다.

장점
* 응답 속도 향상
* 네트워크 트래픽 감소
* 서버 부하 감소

# 2장 URL과 리소스
URL은 리소스의 위치 + 접근 방법을 나타낸다.

예
```text
http://www.example.com:80/index.html?name=bob#top
```

* 스킴: 사용할 프로토콜
    * 스킴은 리소스에 어떻게 접근할지 알려준다.
    * http, https, ftp, mailto, file
* 호스트와 포트

| 프로토콜  | 기본 포트 |
| ----- |-------|
| HTTP  | 80    |
| HTTPS | 443   |
| FTP   | 21    |

* 경로
    * 리소스의 서버 내 위치
* 질의 문자열
    * 서버에 전달되는 추가 파라미터 (key=value)
    * ?name=bob&age=20
* 프래그먼트
    * 문서 내부 특정 위치를 가리킨다.
    * 브라우저 내부에서만 사용된다.

## 상대 URL

상대 URL은 기저 URL(Base URL) 을 기준으로 해석된다.

기저 URL
```text
http://www.joes-hardware.com/tools.html
```

상대 URL
```text
./hammers.html
```

결과 
```text
http://www.joes-hardware.com/hammers.html
```

기저 URL을 사용하여 상대 URL에서 기술하지 않은 정보를 추측할 수 있다.
* 상대 URL이 `./hammers.html`, 기저 URL이 `http://www.joes-hardward.com/tools.html` 인 경우
* 기저 URL의 스킴(http), 호스트(www.joes-hardward.com), 포트(80)를 상속받는다.
* 상대 컴포넌트와 상속받은 컴포넌트를 합쳐 새로운 절대 URL `http://www.joes-hardward.com/hammers.html` 을 만든다.

## URL 인코딩
URL에서는 특정 문자를 사용할 수 없다.
예
* 공백
* 한글
* 특수문자
이러한 문자는 % + 16진수 ASCII 코드로 변환된다.

| 문자    | 인코딩         |
| ----- | ----------- |
| space | %20         |
| !     | %21         |
| 한글    | UTF-8 후 인코딩 |

`hello world` → `hello%20world`

## URI vs URL
URI > URL

| 개념  | 설명              |
| --- | --------------- |
| URI | 리소스를 식별하는 모든 방법 |
| URL | 리소스 위치          |
| URN | 리소스 이름          |

URL: https://example.com/index.html
URN: urn:isbn:9780131101630

URL은 리소스를 찾는데 필요한 서버 이름과 포트를 제공한다.
리소스가 옮겨지면 URL을 더이상 사용할 수 없다는 단점이 있다.

## PURL
URL의 문제점은 리소스가 이동하면 URL이 깨진다는 것이다.
이를 해결하기 위해 PURL(Persistent URL) 이 등장했다.

```text
Client → PURL 서버 → 실제 URL
```
PURL 서버가 리소스의 실제 위치를 관리한다.
따라서 리소스 위치가 바뀌어도 PURL은 변하지 않는다.
클라이언트는 위치 할당자에게 리소스를 가져올 수 있는 영구적인 URL을 요청할 수 있다.

# 3장 HTTP 메시지
* 메시지는 인바운드로 이동하고(서버 쪽으로), 아웃바운드로 이동한다(클라이언트 쪽으로)
* 메시지는 다운스트림으로만 흐른다.
* 메시지는 시작줄, 헤더, 본문(body)로 이루어진다.
  * 시작줄: HTTP 요청 메서드, 요청 대상, HTTP Version
  * 헤더: HTTP 전송에 필요한 모든 부가정보
    * 표준 헤더가 존재하며 필요시 임의의 헤더를 추가가능하다.
  * 본문은 데이터를 담고있다. (Optional)
    * HTML 문서, 이미지, 영상, JSON 등등 byte로 표현할 수 있는 모든 데이터 전송 가능
    * 본문이 없더라도 CRLF(줄바꿈, `\r\n`)로 끝나게된다.
* 요청 메시지는 웹 서버에 어떤 동작을 요구한다.
* 응답 메시지는 요청의 결과를 클라이언트에 돌려준다.

### 요청 메시지
```text
<메서드> <요청 URL> <버전>
<헤더>

<본문>
```
메서드 

### 응답 메시지
```text
<버전> <상태 코드> <사유 구절>
<헤더>

<본문>
```

API를 설계할 때, 리소스와 행위를 분리할 수 있다.

## 메서드(행위)
* GET : 리소스 조회
  * 호출해도 리소스를 변경하지 않아 안전하다.
* POST : 요청 데이터 처리, 주로 등록에 사용
* PUT : 리소스를 대체, 해당 리소스가 없으면 생성(파일을 폴더에 PUT 하는 것 처럼)
* PATCH : 리소스 부분 변경
* DELETE : 리소스 삭제

* HEAD
  * GET과 동일하지만 본문(body)을 반환하지 않는 요청
  * 리소스 존재 여부나 Content-Length, 캐시 상태 등을 확인하기 위해 사용한다.

요청 예시
```text
HEAD /index.html HTTP/1.1
Host: example.com
```

응답 예시
```text
HTTP/1.1 200 OK
Date: Fri, 13 Mar 2026 13:00:00 GMT
Content-Type: text/html
Content-Length: 1256
Last-Modified: Mon, 10 Mar 2026 10:00:00 GMT
Server: nginx
```

* OPTIONS
  * 서버가 지원하는 HTTP 메서드 확인

요청 예시
```text
OPTIONS /users HTTP/1.1
Host: example.com
```

응답 예시
```text
HTTP/1.1 204 No Content
Allow: GET, POST, PUT, DELETE, OPTIONS
Date: Fri, 13 Mar 2026 13:00:00 GMT
Server: nginx
```
`Allow`가 핵심 헤더

## 상태코드
클라이언트가 보낸 요청의 처리 상태를 응답에서 알려주는 기능
클라이언트 입장에서 상태코드를 모르면 200대, 400대, 500대 큰 범위로 나누어 처리하면 된다.

* 100 Continue
  * 큰 요청 body를 보내기 전에 서버가 요청을 받아도 되는지 확인하는 것이다.

100 Continue 응답이 필요한 상황
```text
POST /upload HTTP/1.1
Content-Length: 500[json타입.md](../database/mysql/json%ED%83%80%EC%9E%85.md)MB
```
큰 파일을 업로드해야 하는 상황에서 서버가 요청을 거절한다면 (인증 실패, 권한 없음, 요청 형식 오류 등)
클라이언트는 불필요하게 500MB를 전송하는 **네트워크 낭비**가 발생한다.

```text
POST /upload HTTP/1.1
Host: example.com
Content-Length: 524288000
Expect: 100-continue
```
```text
HTTP/1.1 100 Continue
```
클라이언트는 body없이 헤더만 먼저 보내고 (100-continue 값이 담긴 Expect 헤더도 함께)
서버로부터 100 Continue 응답을 받으면 그때 파일을 업로드한다.
만약 서버가 거절한다면 body를 보내지 않는다. (서버는 100 Continue 응답 또는 에러 코드로만 답해야 한다)


* 200~299: 성공 상태 코드
  * 200 OK: 요청은 정상이며 본문에 요청된 리소스를 포함한다.
  * 201 CREATED: 응답은 생성된 리소스에 대한 참조가 담긴 Location 헤더와 함께 그 리소스를 참조할 수 있는 URL을 본문에 포함해야 한다.
  * 202 Accepted: 요청은 받아들여졌으나 처리가 완료되지 않았음 (요청 접수 후 1시간 뒤에 배치 프로세스가 요청을 처리하는 경우 등에서 사용)


* 300~399: 리다이렉션 상태 코드

리소스가 옮겨졌을 때, 클라이언트에게 리소스가 옮겨졌으며 어디서 찾을 수 있는지 알려주기 위해 
리다이렉션 상태 코드와 Location 헤더를 보낼 수 있다.
웹 브라우저는 3xx 응답 결과에 Location 헤더가 있으면 Location 위치로 자동 이동한다. (리다이렉트)

### 영구 리다이렉션 301, 308
* 리소스의 URI가 영구적으로 이동
* 원래의 URL을 사용하지 않는다. 검색 엔진 등에서도 변경을 인지한다.

### 301 Moved Permanently

리다이렉트 시 요청 메서드가 GET으로 변하고 본문이 제거될 수 있음 (maybe)
```text
POST /event HTTP/1.1
Host: localhost:8080

name=hello&age=20
```
```text
HTTP/1.1 301 Moved Permanently
Location: /new-event
```

```text
GET /new-event HTTP/1.1
Host: localhost:8080
```
자동 리다이렉트 될 때 GET으로 변경되어 요청될 수 있다. (일반적으로 URL이 바뀌면 내부적으로 전달해야하는 데이터가 다 바뀌기 때문에 웬만하면 GET으로 요청하는게 좋다.)
308에서는 POST가 유지된다.

### 일시적인 리다이렉션 302, 307, 303
* 리소스의 URI가 일시적으로 변경
* 따라서 검색 엔진 등에서 URL을 변경하면 안됨

| 상태코드                       | 메서드 변경                  | 의미                  |
| -------------------------- | ----------------------- | ------------------- |
| **302 Found**              | 브라우저가 **GET으로 바꿀 수 있음** | 옛날 방식 리다이렉트         |
| **303 See Other**          | **무조건 GET으로 변경**        | POST 처리 후 결과 페이지 이동 |
| **307 Temporary Redirect** | **메서드 유지**              | POST → POST 유지      |

* 302 Found
  * 원래 HTTP 스펙에서는 메서드를 유지해야 하지만 (POST → POST), 브라우저들이 멋대로 POST → GET으로 동작하는 경우가 있어 혼란이 생겼다.
* 303 See Other
  * POST → GET 로 명확히 바꾸는 상태 코드가 추가되었다.
* 307 Temporary Redirect
  * 명확하게 요청 메서드와 본문을 유지하는 상태 코드가 추가되었다. (POST → POST, GET → GET)

* 400~499: 클라이언트 에러 상태 코드

| 상태코드 | 이름           | 의미            | 설명 / 예시                                       |
| ---- | ------------ | ------------- |-----------------------------------------------|
| 400  | Bad Request  | 요청 구문, 메시지 오류 | 클라이언트는 요청 내용을 다시 검토하고 보내야 함 (요청 파라미터가 잘못된 경우) |
| 401  | Unauthorized | 인증 필요         | `WWW-Authenticate` 헤더로 인증 방법 전달               |
| 403  | Forbidden    | 권한 부족         | 인증은 됐지만 접근 권한 없음                              |
| 404  | Not Found    | 리소스 없음        | 리소스가 없거나 숨기기 위해 사용                            |

* 500~599: 서버 에러 상태 코드

| 상태코드 | 이름                    | 의미                                      |
|------|-----------------------|-----------------------------------------|
| 500  | Internal Server Error | 서버가 요청을 처리할 수 없게 만드는 에러를 만났을 때 사용       |
| 501  | Not Implemented       | 서버가 지원하지 않는 메서드를 사용한 경우                 |
| 503  | Service Unavailable   | 서비스 이용불가, 서버가 일시적인 과부하로 잠시 요청을 처리할 수 없음 |
| 504  | Gateway Timeout       | 리소스 없음                                  |

504 상태코드
```text
Client → Gateway(또는 Load Balancer) → Backend Server
```
일반적인 구조는 위와 같다.
여기서 Gateway → Backend 로 요청을 보냈는데 백엔드가 너무 오래걸려서 응답이 오지 않아 Gateway가 대신 응답한다.

## 헤더

| 분류 | 헤더 | 요청(Request) | 응답(Response) | 설명 | 예시 |
|---|---|---|---|---|---|
| 메시지 메타데이터 | Date | ❌ | ✅ | 메시지가 생성된 날짜와 시간 (서버 기준) | `Date: Tue, 3 Oct 1974 02:16:00 GMT` |
| 콘텐츠 협상 | Accept | ✅ | ❌ | 클라이언트가 선호하는 미디어 타입 전달 | `Accept: */*` |
| 콘텐츠 협상 | Accept-Charset | ✅ | ❌ | 클라이언트가 선호하는 문자 인코딩 | `Accept-Charset: utf-8` |
| 콘텐츠 협상 | Accept-Encoding | ✅ | ❌ | 클라이언트가 선호하는 압축 방식 | `Accept-Encoding: gzip, deflate` |
| 콘텐츠 협상 | Accept-Language | ✅ | ❌ | 클라이언트가 선호하는 자연 언어 | `Accept-Language: ko-KR, en-US` |
| 표현 정보 | Content-Type | ✅ | ✅ | 본문 데이터의 타입 | `Content-Type: application/json` |
| 표현 정보 | Content-Encoding | ❌ | ✅ | 본문 데이터의 압축 방식 | `Content-Encoding: gzip` |
| 표현 정보 | Content-Language | ❌ | ✅ | 본문 데이터의 자연 언어 | `Content-Language: ko` |
| 표현 정보 | Content-Length | ✅ | ✅ | 본문 데이터의 길이 (byte 단위) | `Content-Length: 348` |
| 요청 정보 | Referer | ✅ | ❌ | 이전 웹페이지 주소 (유입 경로 분석 가능) | `Referer: https://example.com/page` |
| 요청 정보 | User-Agent | ✅ | ❌ | 클라이언트 애플리케이션 정보 | `User-Agent: Mozilla/5.0` |
| 요청 정보 | Host | ✅ | ❌ | 요청한 호스트 정보 (도메인) | `Host: example.com` |
| 응답 제어 | Location | ❌ | ✅ | 리다이렉션할 URI 전달 | `Location: /login` |
| 응답 제어 | Allow | ❌ | ✅ | 허용 가능한 HTTP 메서드 | `Allow: GET, POST` |
| 응답 제어 | Retry-After | ❌ | ✅ | 다음 요청까지 기다려야 하는 시간 | `Retry-After: 120` |

# 4장 커넥션 관리
## TCP 커넥션
전 세계 모든 HTTP 통신은 TCP/IP를 통해 이루어진다.
따라서 연결이 수립된 이후에 데이터를 전달한다.
TCP/IP 커넥션이 맺어지면 클라이언트와 서버 간 주고받는 메시지들은 손실되거나 손상되지 않고 순서가 바뀌지 않고 전달됨이 보장된다.

1. 프로그램이 Hello World 메시지 생성
2. SOCKET 라이브러리를 통해 전달
3. TCP 정보 생성
4. IP 패킷 생성, TCP 데이터 포함

TCP는 데이터 스트림을 메시지 데이터를 포함한 TCP 세그먼트 단위로 잘게 나누고
IP 패킷에 담아 데이터를 전달한다.

TCP/IP 패킷
* IP 패킷
  * 출발지 IP, 목적지 IP
* TCP 세그먼트
  * 출발지 PORT, 목적지 PORT, 전송 제어, 순서, 검증 정보
  * 전송 데이터

### TCP 소켓 프로그래밍
운영체제는 TCP 커넥션 관련 여러 기능을 제공한다.
소켓 API를 사용하면 TCP endpoint를 생성하고 데이터 스트림을 읽고 쓸 수 있다.

```text
s1: 새로운 소켓을 만든다.
s2: 80 포트로 소켓을 묶는다.
s3: 소켓 커넥션을 허가한다
s4: 커넥션을 기다린다.

c1: IP 주소와 포트를 얻는다.
c2: 새로운 소켓을 생성한다.
c3: 서버의 IP 포트로 연결한다.

s5: 애플리케이션 커넥션 통지
s6: 요청을 일기 시작한다.

c4: 성공적으로 연결
c5: HTTP 요청을 보낸다.
c6: HTTP 응답을 기다린다.

s7: HTTP 요청 메시지를 처리한다.
s8: HTTP 응답을 보낸다.
s9: 커넥션을 닫는다.

c7: HTTP 응답을 처리한다.
c8: 커넥션을 닫는다.
```

### TCP 관련 지연
(1) TCP 3 way handshake
1. SYN
2. SYN + ACK
3. ACK
RTT(Round Trip Time) 만큼의 네트워크 지연이 발생한다.

(2) 확인 응답 지연
세그먼트를 받으면 확인 응답 패킷을 송신자에게 반환한다.
(송신자가 특정 시간 안에 확인 응답 메시지를 받지 못하면 패킷이 파기되었거나 오류가 있는것으로 판단하고 데이터를 다시 전송한다.)
TCP는 네트워크 효율을 위해 ACK를 즉시 보내지 않고 잠시 기다렸다가 한번에 보내는 경우가 있다.
이 과정에서 수십 ms 정도의 지연이 발생할 수 있다.

(3) 느린 시작
TCP 데이터 전송 속도는 TCP 커넥션이 만들어진지 얼마나 지났는지에 따라 달라진다.
데이터가 성공적으로 전송됨에 따라 속도를 높여나간다. (처음부터 데이터를 많이 보내면 네트워크 혼잡이 발생할 수 있으니 천천히 보내면서 네트워크 상태 확인)

(4) 네이글 알고리즘
작은 패킷들이 네트워크를 장악하여 성능을 떨어뜨리는 것을 방지하기 위해 설계된 알고리즘
전송할 데이터가 최대 세그먼트 크기만큼 쌓일 때까지 기다린다. 
만약 크기가 작더라도 이전에 보낸 패킷에 대한 확인 응답을 받았다면 즉시 보낸다.

HTTP와 같이 작은 메시지를 주고받는 상호작용형 서비스에서는 확인 응답이 올 때까지 데이터를 보내지 않고 대기하므로 심각한 지연이 발생할 수 있다.
최신 웹 서버에서는 이 알고리즘을 끄기 위해 TCP_NODELAY 옵션을 설정한다.

(5) TIME_WAIT의 누적과 포트 고갈
TCP 연결을 끊을 때 마지막 ACK를 보내는 쪽(클라이언트)은 다음 연결의 데이터와 섞이는 것을 방지하고 ACK가 유실되었을 때 재전송하기 위해
일정 시간(약 2분)동안 커넥션을 닫지 않고 유지한다.
커넥션은 종료되었지만 포트가 TIME_WAIT 상태에 묶여 재사용할 수 없게 된다.
