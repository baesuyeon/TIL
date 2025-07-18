## Dockerfile 작성 방법

### FROM
: 베이스 이미지를 지정
```bash
FROM ubuntu:20.04
```
Dockerfile은 반드시 FROM으로 시작해야 한다. 
기반이 되는 운영체제 이미지를 설정한다.

### LABEL
이미지에 메타데이터를 추가한다.
```bash
LABEL maintainer="you@example.com"
LABEL version="1.0"
```
LABEL로 다양한 키-값 쌍 메타데이터를 저장할 수 있다.
이미지 작성자, 버정 등의 정보를 추가하여 이미지를 관리하고 문서화하는데 도움을 준다.

### RUN
컨테이너 내에서 명령어를 실행한다.
```bash
RUN apt-get update && apt-get install -y python3
```
패키지 설치나 환경 설정 등 이미지 빌드 시 필요한 작업을 실행한다.
RUN은 이미지 빌드 과정에서 여러 번 사용될 수 있다.

### COPY
로컬 파일 시스템의 파일이나 디렉토리를 도커 이미지 내로 복사한다.
애플리케이션 소스 코드나 구성 파일을 이미지에 포함시키는 데 사용된다.
```bash
COPY . /app
```
현재 빌드 컨텍스트 내의 모든 파일과 디렉토리를 도커 이미지 내의 `/app` 디렉토리로 복사한다.

빌드 컨텍스트 : `docker build` 명령어를 실행하는 디렉토리가 빌드 컨텍스트가 된다.
일반적으로 `Dockerfile`이 위치한 디렉토리에서 `build` 명령어를 실행하므로 `Dockerfile`이 있는 위치가 빌드 컨텍스트가 된다.

### ADD
파일을 복사하거나, URL에서 파일을 다운로드하거나, 압축 파일을 해제한다.
```
ADD myfile.tar.gz /app
```
`ADD`는 로컬 파일 복사, 압축 파일 자동 해제, 외부 URL에서 직접 다운로드 기능(외부 URL에서 파일을 받을 경우 보안 문제가 생길 수 있다)을 제공한다.
`COPY`보다 기능이 많지만, 예측이 어렵고 불필요한 기능이 작동할 수도 있기 떄문에 일반적으로는 단순 복사엔 `COPY`를, 꼭 필요한 경우만 `ADD`를 쓰는 게 좋다.

### WORKDIR
```bash
WORKDIR /app
```
이후의 명령들이 실행될 디렉토리를 설정한다.
`WORKDIR`은 `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, `ADD` 명령어가 실행될 작업 디렉토리를 설정한다.

### CMD
```bash
CMD ["python3", "app.py"]
```
컨테이너 실행 시 기본으로 실행될 명령을 지정한다.