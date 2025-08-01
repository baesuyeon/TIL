## Volume이란?

컴퓨터 과학에서 볼륨은 데이터를 저장하는 논리적 단위이다.
하드디스크, SSD와 같은 저장장치에 존재하며 파일 시스템을 갖춘 독립 저장 공간이다.
사용자는 이 볼륨을 통해 데이터를 저장, 관리, 백업, 복구할 수 있다.

## Docker Volume이란?
도커에서 데이터를 지속적으로 저장할 수 있도록 해주는 기능이다.
컨테이너가 삭제되거나 재시작되어도 데이터는 유지된다.

### 익명 Volume
```bash
docker run -v /data nginx
```
/data 디렉토리에 익명 볼륨이 마운트된다.

익명 볼륨도 데이터 저장은 가능하지만 이름이 없기 때문에 관리, 재사용이 어렵고
특히 `--rm` 옵션을 사용하여 컨테이너를 삭제하면 함꼐 삭제되므로 지속적인 저장용도로는 부적절하다.
중요한 데이터를 다룰때는 이름있는 볼륨(명시적 볼륨)을 사용하는 것이 안전하다.

### 이름있는 Volume
```bash
docker volume create my-volume
docker run -v my-volume:data nginx
```
my-volume이라는 이름의 볼륨이 /data 디렉토리에 마운트된다.  
이름있는 볼륨은 다양한 컨테이너 간에 데이터를 공유할 수 있으며, 컨테이너의 수명과 무관하게 데이터를 지속적으로 저장하고 쉽게 관리할 수 있다.


컨테이너: 나는 /app/data 경로에 저장할래!
도커 엔진: 좋아, 실제로는 호스트의 /var/lib/docker/volumes/my-volume/_data에 저장해줄게!

### Volume 조회
```bash
docker volume ls
```
현재 호스트에 존재하는 모든 도커 볼륨 목록 조회

```bash
docker volume inspect <볼륨 이름>
```
특정 볼륨 상세 조회

```bash
docker volume inspect my-volume
```

출력 예시
```
[
    {
        "CreatedAt": "2024-11-03T04:59:26Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/my-volume/_data",
        "Name": "my-volume",
        "Options": null,
        "Scope": "local"
    }
]
```
Mountpoint는 호스트 컴퓨터에서 해당 도커 볼륨의 실제 저장 경로를 의미한다.

### 호스트에서 볼륨 데이터 접근
```bash
ls /var/lib/docker/volumes/my-volume/_data
```
Linux 에서는 이 명령어를 통해 해당 볼륨의 디렉토리에 저장된 파일 목록을 확인할 수 있지만 윈도우, 맥에서는 컨테이너를 실행하여 해당 볼륨을 마운트한 후 조회해야 한다.

```bash
docker volume ls
```

1. 볼륨 확인

```bash
docker run --rm -v shared-volume:/data busybox sh -c "echo 'Hello, Docker!' > /data/hello.txt"
```

명령어 정리
| 명령어            | 설명                                       |
| -------------- | ---------------------------------------- |
| `docker run`   | 새 컨테이너를 실행한다.                        |
| `--rm	`        | 컨테이너 실행이 끝나면(-c 명령어를 실행한 뒤) 자동으로 컨테이너를 삭제하겠다. |
| `-v shared-volume:/data`  | 호스트의 shared-volume 볼륨을 컨테이너의 /data에 마운트한다. |
| `busybox`   | 리눅스에서 흔히 쓰는 sh, ls, cp, echo, cat, mkdir, rm 등 여러 명령어를 내장한 초경량 이미지이다.             |
| `sh -c ""`   | sh는 리눅스에서 사용하는 쉘 프로그램 중 하나이다. -c는 뒤에 나오는 문자열을 하나의 명령어로 처리한다.             |
| `echo 'Hello, Docker!' > /data/hello.txt`   | shared-volume에 연결된 /data 경로에 hello.txt 파일을 생성하고 그 안에 "Hello, Docker!"를 쓴다. |

1. 컨테이너를 실행하여 볼륨에 데이터 작성  

```bash
docker run --rm -v shared-volume:/data busybox cat /data/hello.txt
```
1. 컨테이너에서 데이터 확인

```bash
docker run --rm -v shared-volume:/data busybox sh -c "chmod 644 /data/hello.txt"
```
만약 권한이 없다면 chmod로 권한 수정 후 다시 실행