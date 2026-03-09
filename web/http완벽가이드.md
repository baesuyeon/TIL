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
