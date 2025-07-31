## Docker

Docker는 애플리케이션을 실행하는 데 필요한 모든 것을 하나의 컨테이너로 패키징하여 어디서나 동일한 환경에서 실행할 수 있게 하는 기술이다.
전통적인 배포 방식에서는 dev 환경과 prod 환경의 차이로 인해 문제가 발생할 수 있었으나, Docker 를 사용하면 환경 차이로 인한 문제를 해결할 수 있다. (내 컴퓨터에서는 되는데요 문제)

### Docker CLI
Docker 엔진과 상호작용하기 위한 명령어 기반 인터페이스이다. (docker 명령어로 시작되는 커맨드)

```bash
docker --version
```
Docker CLI가 설치 완료되면 위 명령어로 확인 가능하다.

### Docker 이미지

Docker 이미지를 기반으로 컨테이너를 생성할 수 있다.
(Docker 이미지를 프로그램 설치 프로그램이라고 비유할 수 있다. Excel 설치 프로그램에는 Excel을 실행하기 위한 모든 파일과 설정이 포함되어있으며 이 설치 프로그램은 변경되지 않고 어디서든 동일하게 설치될 수 있다.)

- 이미지는 여러 layer 계층으로 구성된다.
- 각 계층은 캐시로 활용될 수  (동일한 계층이 이미 존재하면 다시 빌드하지 않고 캐시된 계층을 사용)

```bash
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y python3
COPY . /app
```
- 첫 번째 계층, Ubuntu 20.04 이미지를 기반으로 한다.
- 두 번째 계층, Ubuntu 20.04 이미지 위에 패키지 설치 명령을 실행한다.
- 세 번째 계층, 현재 디렉토리의 파일을 이미지 내 /app 디렉토리에 복사한다.

### Docker 컨테이너

Docker 이미지를 실행한 상태이다.
컨테이너는 격리된 환경에서 애플리케이션을 실행하며 필요한 리소스와 네트워크 설정을 포함한다.

## Docker Hub

Docker Hub는 Docker 이미지를 저장, 공유, 배포할 수 있는 중앙 저장소이다.

## 🐳 Docker 명령어 정리

자주 사용하는 Docker 명령어 정리

### docker build
이미지를 빌드하는 명령어

```bash
docker build [OPTIONS] PATH | URL | -s
```

```bash
docker build -t my-python-app .
```
현재 디렉토리에 있는 Dockerfile을 사용하여 my-python-app 이라는 이름의 이미지를 빌드한다.
Repository TIL/devops/docker/flask-sample 경로로 이동하여 위 명령어를 사용하여 이미지를 빌드할 수 있다.
`docker images` 명령어로 생성된 이미지를 확인할 수 있다.

옵션 정리
| 옵션            | 설명                                       |
| -------------- | ---------------------------------------- |
| `-t`           | 빌드된 이미지에 태그를 지정한다.                  |
| `.`            | Dockerfile이 있는 현재 디렉토리를 빌드 컨텍스트로 사용한다.|
| `--build-arg`  | 빌드 시  변수 값을 설정한다.                     |
| `--no-cache`   | 캐시를 사용하지 않고 이미지를 빌드한다.             |

도커는 빌드 컨텍스트 위치의 파일만 `COPY`, `ADD` 할 수 있다.

### docker pull
```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```
Docker Hub에서 이미지를 다운로드 한다.

```bash
docker pull ubuntu
```
Docker Hub에서 최신 버전의 Ubuntu 이미지를 다운로드한다.

### docker push
```bash
docker push [OPTIONS] NAME[:TAG]
```
Docker 이미지를 레지스트리에 업로드한다.

```bash
docker push my-repo/my-image:latest
```
my-repo 레지스트리에 my-image라는 이름의 최신 이미지를 업로드한다.

### docker run
빌드된 이미지를 기반으로 컨테이너를 실행한다.
```bash
docker run -d -p 8080:5000 my-python-app
```
옵션 정리
| 옵션            | 설명                                       |
| -------------- | ---------------------------------------- |
| `-d`           | 컨테이너를 백그라운드에서 실행함 (기본 실행시에는 컨테이너 로그가 터미널에 출력되고 터미널을 닫으면 실행도 중단된다. -d 옵션을 주는 경우 	터미널을 점유하지 않고 컨테이너는 백그라운드에서 계속 동작한다) |
| `-p 8080:5000` | 포트 맵핑, 호스트(로컬)의 포트 8080을 컨테이너의 5000 포트에 연결|

컨테이너 실행 후 `http://localhost:8080/` 에 접속하면 웹 페이지를 확인할 수 있다.

### docker ps
`docker ps` 명령어로 실행 중인 컨테이너를 확인할 수 있다.

### docker image
```bash
docker image [OPTIONS] COMMAND
```

주요 하위 명령어 정리
| 명령어            | 설명                                       |
| -------------- | ---------------------------------------- |
| `-ls`           | 현재 로컬에 저장된 모든 Docker 이미지 목록을 나열한다. |
| `rm`            | Docker 이미지를 삭제한다. (`docker image rm my-image`) |
| `inspect`       | Docker 이미지의 자세한 정보를 출력한다.            |
| `prune`         | 사용하지 않는 Docker 이미지를 정리한다.   |


### docker run

```bash
docker run -d -p 6379:6379 redis --name myredis
```

새로운 컨테이너를 생성하고 실행한다.
Redis 서버는 6379 포트를 열고 클라이언트의 요청을 받을 준비를 한다.

옵션 정리
| 옵션            | 설명                                       |
| -------------- | ---------------------------------------- |
| `-d`           | Detached 모드 (컨테이너를 백그라운드에서 실행)                   |
| `-p 6379:6379` | 포트 맵핑, 호스트(로컬)의 포트 6379를 컨테이너의 6379 포트에 연결      |
| `redis`        | local 환경에서 Redis 이미지를 찾고 없으면 Docker Hub에서 자동 다운로드된다. |
| `--name`       | 컨테이너 이름을 설정한다.   |
| `-e`           | 컨테이너 환경변수를 설정한다.   |
| `-v`           | 볼륨을 마운트하여 데이터를 저장한다.   |

```bash
docker run -d --name dbcontainer -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql
```
MySQL 이미지를 사용하여 이름이 `db-container`인 컨테이너를 생성하고 환경변수를 설정하여 루트 비밀번호를 지정한다.

```bash
docker run -d --name volumne-container -v /host/data:/container/data ubuntu
```
ubuntu 이미지를 사용하여 이름이 `volumne-container`인 컨테이너를 생성하고 호스트의 디렉토리를 컨테이너의 디렉토리에 마운트한다.

### docker ps

현재 실행 중인 Docker 컨테이너 목록을 확인할 수 있다.

옵션 정리
| 옵션            | 설명                                       |
| -------------- | ---------------------------------------- |
| `-a`           | 중지된 컨테이너 포함 전체 목록 표시               

### docker exec -it <컨테이너 이름 또는 ID> bash

컨테이너 내부에 접속 

* 터미널 접속 (bash 쉘)
```bash
docker exec -it myredis bash

redis-cli
```
redis-cli는 Redis 서버에 접속해서 명령을 보낼 수 있는 도구이다. (호스트 명과 포트번호를 생략하면 localhost의 6379로 접속된다.)

* 파일 목록 확인
```bash
docker exec -it my-container ls /tmp
```
컨테이너 내부에서 특정 디렉토리의 파일 목록을 나열한다.

* 파일 생성 및 내용 입력
```bash
docker exec -it my-container sh -c 'echo "helloworld" > /tmp/newfile'
```
컨테이너 내부에서 새로운 파일을 생성하고 내용을 입력한다.

* 파일 내용 확인
```bash
docker exec -it my-container cat /tmp/newfile
```
컨테이너 내부에서 파일의 내용을 확인한다.

### docker logs <컨테이너 이름 또는 ID>
`docker logs <컨테이너 ID>` 명령어로 실행 중인 컨테이너의 표준 출력(stdout) 및 표준 에러(stderr) 로그를 확인할 수 있다.
컨테이너 ID는 `docker ps` 명령어를 통해 확인할 수 있다.

옵션 정리
| 옵션           | 설명                   |
| ------------ | -------------------- |
| `-f`         | 실시간 로그 스트리밍 (follow) |
| `--tail <N>` | 마지막 N줄만 출력           |
| `-t`         | 로그에 타임스탬프 포함         |

### docker stop <컨테이너 이름 또는 ID>
```bash
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```
실행 중인 컨테이너를 중지한다.

옵션 정리
| 옵션            | 설명                                       |
| -------------- | ---------------------------------------- |
| `-t`           | 지하기 전에 기다릴 시간(초)를 지정한다.                  |


### docker rm <컨테이너 이름 또는 ID>
중지된 컨테이너 삭제

### docker kill
```bash
docker kill [OPTIONS] CONTAINER [CONTAINER...]
```
옵션 정리
| 옵션              | 설명                                       |
| --------------   | ---------------------------------------- |
| `-s`, `--signal` | 보낼 신호를 지정한다. 기본값은 SIGKILL이다.     |

실행 중인 Docker 컨테이너를 강제로 종료하기 위해 사용한다.
이 명령어는 기본적으로 `SIGKILL` 신호를 보내어 컨테이너를 즉시 중지시키며, 컨테이너 내에서 실행 중인 모든 프로세스를 강제로 종료한다.
`docker stop` 명령어와 달리 `docker kill`은 그레이스풀 셧다운을 시도하지 않는다.

```bash
docker kill my-container
docker kill container1 container2 container3
docker kill 00959afe8c0c
```

```bash
docker kill -s SIGTERM my-container
```
my-container 컨테이너에 SIGTERM 신호를 보내 종료한다.
SIGTERM 신호는
일반적으로 프로세스를 종료할 때 사용되며,
.
프로세스
가 종료 전에 정
리
용함
.

* 그레이스풀 셧다운
  * 프로세스가 종료될 때  작업을 수행할 시간을 주는 방식
  * 프로세스가 현재 처리 중인 작업을 완료하고, 열린 파일이나 네트워크 연결을 닫으며, 필요한 상태를 저장한 후  종료될 수 
  *  데이터 손실이나 불완전한 상태를 방지하고, 시스템의 일관성을 유지하는데 중요함

* 신호(signal)
  * SIGTERM(Signal Terminate)은 운영체제에서 프로세스에 종료를 요청하는 표준 신호이다.
    * 프로세스에게 종료하라고 요청하며 프로세스는 이를 받아들이고 필요한 정리 작업을 수행한 후 종료할 수 있다. (Graceful Shutdown)
    * SIGTERM은 데이터 손실을 최소화하고 프로세스가 정상적으로 종료할 수 있는 기회를 제공한다.
    * 프로세스가 이 신호를 무시할 수 있으며, 필요한 경우 SIGKILL 신호로 강제 종료할 수 있다.
  * SIGKILL(Signal Kill)은 프로세스를 즉시 종료시키는 강제 신호이다.
    * 프로세스가 신호를 무시할 수 없으며 즉시 종료된다.
    * 정리 작업 없이 즉시 종료되기 때문에 데이터 손실이나 리소스 누수가 발생할 수 있다.
    * 주로 응답하지 않는 프로세스를 강제로 종료할 때 사용한다.
