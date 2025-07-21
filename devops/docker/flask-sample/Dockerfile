# Python 3 이미지 사용 (Docker Hub로부터 공식 파이썬 이미지를 사용)
FROM python:3.10-slim

# 작업 디렉토리 지정
WORKDIR /app

# 필요한 파일 복사 (COPY <호스트의 경로> <컨테이너의 경로>)
COPY requirements.txt /app
COPY app.py /app

# requirements.txt에 정의된 필요한 패키지 목록 설치
RUN pip install --no-cache-dir -r requirements.txt

# 이 이미지가 어떤 포트를 사용하는지 도커 이미지 사용자에게 알려준다. ('이 도커 이미지는 5000번 포트를 외부에 공개할 예정이다'라고 명시하는 것)
EXPOSE 5000

# 도커에서는 print() 출력이 버퍼링돼서 바로 로그에 안 찍히는 문제가 생길 수 있는데 Python의 출력을 실시간으로 보여주도록 설정한다.
ENV PYTHONUNBUFFERED=1

# Flask 실행 명령 지정 (컨테이너 실행 시 자동 실행됨)
CMD ["python", "app.py"]