# Docker

## 도커 개념

1. 컨테이너: 다양한 프로그램을 컨테이너 안에 추상화 시켜서 손쉽게 이동, 배포, 관리를 할 수 있게 해줌
   - 코드와 모든 종속성을 패키지화하는 한 단위라고 보면 될듯
   - **도커 이미지의 인스턴스**
2. 이미지 : 프로그램을 실행하는데 필요한 설정이나 종속성을 가짐

3. 도커는 도커 CLI 와 도커 서버(Daemon)으로 이루어져 있음
   - 도커 클라이언트(CLI)에 커맨드를 입력하면, 도커 서버가 그 커맨드를 받아 이미지를 생성하거나 컴테이너를 생성하는 등의 작업을 하게 됨
   - 이미지 run 명령어를 입력하면, 먼저 도커 서버에서 이미지 Cache 보관 장소를 뒤지고, 캐시 보관 장소에 해당 이미지가 없으면 도커 허브에서 가져옴 

4. 가상화 기술과 도커와의 차이

   - 우선 가상화 기술이 나오기 전에는 한대의 서버를 하나의 용도로만 사용 가능했음, 하나의 서버에 하나의 운영체제, 하나의 프로그램만을 운영
   - 하이퍼 바이저 기반의 가상화 출현 후: 호스트 시스템에서 다수의 게스트 os를 구동할 수 있게 하는 소프트웨어, 하드웨어를 가상화하면서 하드웨어와 각각의 vm을 모니터링 하는 중간 관리자
     - 하이퍼바이저에 의해 구동되는 vm은 각 vm마다 독립된 가상 하드웨어 자원을 할당 받음. 논리적으로 분리되어 있어서 한 vm에 오류가 발생해도 다른 vm으로 퍼지지 않는다는 장점 있음
   - 하이퍼바이저 기반의 가상화와 다르게 도커에는 게스트 os가 필요 없음 -> 더 가볍
   - 도커는 커널을 공유, host operating system에서 os도 공유


5. 컨테이너 나누기 기술: 리눅스의 Cgroup과 네임스페이스 기술을 사용

   - 컨테이너와 호스트에서 실행되는 다른 프로세스 사이에 벽을 만드는 리눅스 커널 기능들

   - cGroup: cpu나 메모리등 관리

   - 네임스페이스: 하나의 시스템에서 프로세스 분리하는 가상화 기술

   - 근데 왜 리눅스에서 사용되는게 맥에서 될까? 도커에는 리눅스 환경이 기본 시스템인가봐

6. 이미지 컨테이너로 만들기 디테일

   - Docker 클라이언트에 docker run <이미지> 입력
   - 도커 이미지에 있는 파일 스냅샷을 컨테이너 하드 디스크에 옮겨줌




## 도커 클라이언트 명령어 알아보기

1. docker run <이미지> : 이미지 실행 (create + start -a 합친것)
2. docker run <이미지> ls : 내부 파일 시스템 구조 보기, 맨 마지막 자리는 원래 이미지가 가지고 있는 시작 명령어를 무시하고 마지막 자리에 있는 커맨드를 실행하게 함
3. docker ps : 현재 실행되고 있는 컨테이너 나열 (-a 붙이면 실행 안되고 있는 컨테이너도 다 같이)

4. Stop / kill : stop은 자비롭게 그동안 하고 있던 작업들 완료하고 컨테이너 중지, kill은 걍 킬
5. Rmi : 이미지 삭제
6. Docker system prune: 컨테이너 이미지 네트워크 모두 삭제, 실행중인 컨테이너에는 영향 주지 않음
7. docker exec <컨테이너 아이디> ls : 컨테이너 실행중에도 사용 가능

## 도커 이미지 만들기

### 도커 파일 작성하기

1. 베이스 이미지 명시(파일 스냅샷에 해당)
   - 베이스 이미지: 도커 이미지는 여러개의 레이어로 되어있음. 베이스 이미지는 이 이미지의 기반이 되는 부분, 레이어는 중간 단계의 이미지
   - os라고 생각하면 될듯
2. 추가적으로 필요한 파일을 다운받기 위한 명령어들 명시(파일 스냅샷에 해당)
3. 컨테이너 시작시 실행 될 명령어 명시(시작 시 실행 될 명령어에 해당)

```dockerfile
# 간단하게 hello를 출력하는 이미지 만들어보자
# 베이스 이미지를 명시해준다
FROM alpine # <이미지 이름><태그> 형식으로 작성

WORKDIR /usr/src/app # 어플리케이션을 위한 소스들 디렉토리 만들어 보관
COPY ./ ./ # 현 디렉토리에 있는 모든 것들 복사(이것만 사용하면 문제 많음)

# 추가적으로 필요한 파일들 다운로드, 이미지가 생성되기 전에 수행할 쉘 명령어
# RUN command

# 컨테이너 시작시 실행할 실행 파일 또는 쉘 스크립트, 딱 1회만 쓸 수 있음
CMD [ "echo", "hello" ]
```

4. 해당 디렉토리 내에서 도커 파일을 찾아 도커 클라이언트로 보내기 : docker build ./ 또는 docker build .(이미지 -> 임시 컨테이너 -> 새로운 이미지 -> 컨테이너 : 컨테이너를 통해서 새로운 이미지를 만들게 됨)
   - 베이스 이미지 가져옴
   - 임시 컨테이너 생성(하드 디스크에 파일 시스템 스냅샷 추가)
   - 실행 후 명령어 삽입
   - 새로운 이미지 생성

5. 이미지 이름 쉽게 해주기: docker build -t "나의 도커 아이디/저장소/프로젝트 이름 : 버전" ./



### docker volume (로컬 변화 빌드 안해도 도커 컨테이너에 바로 반영)

copy는 로컬에 있는 파일들을 컨테이너로 복사하는 것이고,

**volumn은 컨테이너가 로컬에 있는 파일들을 참조**하는 것

`docker run -d -p 5000:8080 -v /usr/src/app/node_modules -v $(pwd):/usr/src/app <이미지 아이디>` 

이러고 빌드 안해도 껏키 하면 반영 됨

- Node_modules는 변화 추적 안하고 나머지는 추적



## Docker Compose

Docker compose: 다중 컨테이너 도커 애플리케이션을 정의하고 실행하기 위한 도구

도커에서 접근을 위해서 `docker run redis`로 redis 컨테이너 켜주고, 또 다른 컨테이너에서 해당 서버에 접근하려 하면 접근이 안됨 -> 바로 도커 컴포즈가 필요한 이유

Docker-compose.yml 

- `docker-compose up`으로 실행하면 두 컨테이너 동시에 켜기 가능
- `docker-compose up --build` 하면 빌드부터 가능
- `docker compose down` 작동시킨 컨테이너 한꺼번에 중단

```yaml
version: "3"
services: 
  redis-server: # 연결 할 컨테이너 1
    image: "redis"
  node-app: # 연결 할 컨테이너 2
    build: .
    ports:
      - "5000:8080"
```



## 단일 컨테이너 배포해보기

### 개발 환경 용

Dockerfile.dev 로 나누어서 도커파일 작성(실행시 `docker build -f docerfile.dev ./`로 해야) 

dockerfile.dev

```dockerfile
FROM node:alpine

WORKDIR /usr/src/app

COPY package.json ./

RUN npm install

COPY ./ ./

CMD ["npm", "run", "start"]

ENV CHOKIDAR_USEPOLLING=true # 핫 리로딩에 필요하다고 해서 추가
```



Docker-compose 작성

```javascript
version: "3"
services: 
  react:
    build: 
      context: . # 도커 이미지를 구성하기 위한 파일과 폴더들이 있는 위치
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumns # 로컬 머신에 있는 파일들 맵핑
      - /usr/src/app/node_modules
      - ./:/usr/src/app
    stdin_open: true # 리액트 앱을 끌때 필요(버그 수정)
    environment: # 리엑트 핫 리로딩
      - CHOKIDAR_USEPOLLING=true
  tests: # 테스트 코드도 볼륨으로 맵핑
    build: 
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /usr/src/app/node_modules
      - ./:/usr/src/app
    command: ["npm", "run", "test"]
```



### 운영 환경 용

#### nginx 사용

- 빌드 파일을 우선 만들어서 넣어 놓고(spa)
- nginx 서버를 이용해서 브라우저의 요청에 따라 제공

dockerfile

```dockerfile
FROM node:alpine as builder # 여기 FROM부터 다음 FROM전까지는 모두 builder stage

WORKDIR '/usr/src/app'

COPY package.json ./

RUN npm install

COPY ./ ./

RUN npm run build


# run stage
FROM nginx 

EXPOSE 80

# 위의 builder 에서 빌드된 파일 복사
COPY --from=builder /usr/src/app/build /usr/share/nginx/html
```

- `docker run -p 8080:80 <이미지 이름>`: nginx는 80포트 사용



### travis.yml

```yaml
sudo: required # 관리자 권한 갖기

language: generic # 언어(플랫폼)

services:
  - docker

before_install: # 스크립트를 실행할 환경
  - echo "start"
  - docker build -t hjy/docker_react -f Dockerfile.dev .

script: # 실행할 스크립트
  - docker run -e CI=true hjy/docker_react npm run test -- --coverage

deploy:
  provider: elasticbeanstalk
  region: "ap-northeast-2"
  app: "docker-test"
  env: "Dockertest-env"
  bucket_name: "elasticbeanstalk-ap-northeast-2-810212283950"
  bucket_path: "docker-test"
  on:
    branch: main
  access_key_id: $AWS_ACCESS_KEY
  secret_access_key: $AWS_SECRET_ACCESS_KEY
```



