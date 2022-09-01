## 도커로 개발환경 확장하기

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbcXbM0%2FbtrJ17mM02q%2Ff9GeW6Z5ZutdbbQbBjrc90%2Fimg.png)

## 마이크로서비스 패키징

1.  마이크로서비스에 관한 도커 파일을 만든다.
2.  도커 이미지 형태로 마이크로서비스를 패키징한다.
3.  컨테이너에서 부팅하고 게시된 이미지를 테스트한다.

-   도커파일, Dockerfile : 도커로 만들 이미지의 세부 사항을 포함하는 하나의 스크립트 파일

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdyzxHx%2FbtrJZPHdrqD%2FTK3PNFlgYoR47U3GNEDi9K%2Fimg.png)

도커파일에서 마이크로서비스와 그 종속성 및 관련 파일들을 정의한다.

```
FROM node:12.18.1-alpine   // node:12.18.1-alpine 이미지에서 시작한다
WORKDIR /usr/src/app       // 작업할 디렉토리를 설정한다. cd /usr/src/app 과 유사하다.
COPY package*.json ./      // 도커파일이 실행되는 root dir의 package*.json 파일을 복사한다. 현재 package.json이 복사된다.
RUN npm install --only=production
COPY ./src ./src           // 구동에 필요한 소스코드를 복사한다.
COPY ./video ./videos      // 비디오파일을 복사한다.
CMD npm start              // node app을 실행한다
```

> RUN, CMD, ENTRYPOINT
> 
> -   RUN {COMMAND} : 새로운 레이어에서 명령어를 실행하고 새로운 레이어를 만든다. 일반적으로 패키지 설치(apt-get)에 많이 사용된다. **이미지를 빌드할 때 실행되는 명령어이다. 이미지가 컨테이너가 되어 실행될 땐 실행되지 않는다.**
> -   CMD {COMMAND} : 이미지가 컨테이너가되어 실행될 때 실행되는 명령어이다. `docker run` 명령어와 함께 실행되는 명령어에 의해 덮어쓰기 되면 실행되지 않는다.
> -   ENTRYPOINT {COMMAND} : 컨테이너가 최초로 실행될 때 실행되는 명령어이다.  
>     [Dockerfile Entrypoint와 CMD의 올바른 사용 방법](https://bluese05.tistory.com/77)

> alpine vs non-alpine  
> 경량화 리눅스인 alpine linux를 기반으로 만들어진 이미지. 일반 ubuntu, centos 이미지 대비 용량이 낮고 속도가 빠르다. node:12.22.12 이미지 용량을 보면 기본 이미지 용량이 332mb, alpine이 28mb 이다.

도커 이미지 빌드

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FExxtb%2FbtrJZIVURL1%2FWvYEXKm0zH9DWqDqUBAVN1%2Fimg.png)

```
# docker build -t {이미지 이름} {도커파일 이름} {실행할 디렉터리}
docker build -t video-streaming --file Dockerfile .

# 빌드 이미지 확인
docker image list

# 실행
docker run -d -p 3000:3000 video-streaming
# -d : detached 백그라운드에서 실행
# -p : port, 포트 바인딩, {호스트에서 연결할 포트}:{컨테이너에서 오픈한 포트}

# 로그 확인
docker logs {container id}
```

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fk374Z%2FbtrJZJtIEde%2FB5wd3ijZp3thcv21JmwZB0%2Fimg.png)

[![asciicast](https://asciinema.org/a/516645.svg)](https://asciinema.org/a/516645)

## 마이크로서비스 게시하기

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbEUM33%2FbtrJZJHjRqX%2FQr7CxXFXi7HEdU3PI8EXW1%2Fimg.png)

1.  애저에 있는 private container registry를 생성한다.
2.  publish 하기 전 `docker login` 명령어를 사용해 인증한다.
3.  `docker push` 명령어로 이미지를 업로드한다.
4.  `docker run`으로 게시된 이미지 마이크로서비스가 시작됬는지 확인한다.

```
docker login rura6502.azurecr.io --username rura6502 --password {PASSWORD}
docker tag video-streaming rura6502.azurecr.io/video-streaming:latest

docker push rura6502.azurecr.io/video-streaming:latest

docker rmi rura6502.azurecr.io/video-streaming:latest
docker pull rura6502.azurecr.io/video-streaming:latest
docker run --rm -p 3000:3000 rura6502.azurecr.io/video-streaming:latest
```

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbRhQoA%2FbtrJ2pt0HzU%2F3wQZN332VJH9lIqHFKrCs0%2Fimg.png)

[![asciicast](https://asciinema.org/a/516649.svg)](https://asciinema.org/a/516649)

# 마이크로서비스 데이터 관리

이번엔 두 가지 기능, 비디오를 저장하는 공간인 파일 저장소와 비디오 경로를 저장하기 위한 데이터베이스를 추가한다. 데이터베이스로 MongoDB를 저장하고 파일 저장소로는 애저 스토리지를 사용한다. 또한 이렇게 추가되는 서비스를 같이 실행시키기 위해 도커 컴포즈를 사용해 개발 환경을 구축한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdnW5Gb%2FbtrKsFYBNHp%2F57gSdcFSKOsGO1B7XVXc90%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FIhSaX%2FbtrKp5xds5F%2FGv2XT8L2WtOB38qhQ8ik9k%2Fimg.png)

### 도커 컴포즈를 사용하는 이유

개발 환경에서 여러 마이크로서비스를 빌드, 실행, 관리하는 것을 편하게 해주는 도구. 다중 컨테이너 앱을 쉽게 관리할 수 있음. 도커를 설치하면 도커 컴포즈는 이미 설치되어 있다.

```
docker-compose --version
```

도커 컴포즈 파일 생성

```
// docker-compose.yaml
version: '3'
services:
  video-streaming:                <-- 서비스 이름 지정
    image: video-streaming        <-- 이미지의 이름
    build:                         <-- 이미지 파라미터
      context: ./video-streaming    <-- 디렉토리 설정
      dockerfile: Dockerfile        <-- 도커파일 설정
    container_name: video-streaming    <-- 이미지 실행 시 컨테이너 이름
    ports:    <-- docker의 -p 옵션과 같음
     - "4000:80"    <-- 호스트 포트 : 컨테이너 포트,
    environment:
      - PORT=80        <-- 환경변수 지정
    restart: "no"    <-- 비정상 종료 시 자동 재시작하지 않음
```

-   `restart: "no"` : 재시작할 경우 문제를 놓치기 쉽기 때문에 no로 지정

```
# 도커 이미지를 빌드(--build)하고 구동(up) 한다.
docker-compose up --build

# -d : 백그리운드에서 실행한다

docker-compose ps        # 현재 docker-compose로 실행중인 서비스 목록을 출력한다.
docker-compose stop        # 중지한다.
docker-compose down        # 
```

> `--build` 명령어를 포함하지 않으면 이전에 저장되어있던 이미지를 사용하기 때문에 직전 변경사항이 적용되지 않을 수 있으므로 항상 붙이는 습관을 들이는 것이 좋다.

> docker-compose를 foreground로 실행중일 때 ctrl+c로 강제중지 시 한 번만 누르도록 해야한다. 한 번 누르면 도커 컴포즈가 실행중인 도커들을 차례대로 중지하지만 한 번 더 누를 경우 중지하는 프로세스가 중지되어 해당 도커 컴포즈 서비스들이 모두 종료되지 않을 수 있다.

[![asciicast](https://asciinema.org/a/516644.svg)](https://asciinema.org/a/516644)

### 도커 컴포즈를 운영환경에 사용한다면?

도커 컴포즈만으론 운영환경에 적용하기 어렵다. 확장 등에 대한 기능이 약하다. 이를 보완하여 운영환경에서도 적용가능한 docker swarm도 있지만 kuberntes에 비해 기능이 약하다.

## 앱에 파일 저장소 추가

마이크로소프트가 제공하는 클라우드 저장소인 azure storage를 이용해서 비디오 스트리밍 서비스가 이용할 비디오 파일을 저장한다. 여기서 역할 분리(SoC, Separation Of Concerns)와 단일 책임 원칙(Single Responsibility Principle)을 적용해 아래와 같은 모양으로 설계할 것이다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FrcQAy%2FbtrKq4xRfRF%2FIvrUlKJ4pyKBSwiffB8Vj0%2Fimg.png)

위와 같이 스토리지에 연결하는 서비스를 별도로 분리하면, 다른 클라우드 서비스를 사용할 때 해당 부분만 쉽게 교체할 수 있으며, 핫스왑(운영 중에 중단 없이 교체가 가능한 기능)을 지원한다고 볼 수 있다.

```
export PORT=3000
export STORAGE_ACCOUNT_NAME=rura6502
export STORAGE_ACCESS_KEY={ACCESS_KEY}
```

### 비디오 스트리밍 마이크로서비스 업데이트

### 새로운 마이크로서비스를 도커 컴포즈 파일에 추가하기

새로 추가되고 기존에 변경된 애플리케이션을 다시 이미지로 만들고 구동시키기 위해 docker-compose 파일을 업데이트한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Flojam%2FbtrKuTVTjY5%2FzrKHRQj9v1nOwxwtrdaJCk%2Fimg.png)

```
version: '3'
services:

  azure-storage:
    image: azure-storage
    build: 
      context: ./azure-storage
      dockerfile: Dockerfile
    container_name: video-storage
    ports:
     - "4000:80"
    environment:
      - PORT=80
      - STORAGE_ACCOUNT_NAME=<insert your Azure storage account name here>
      - STORAGE_ACCESS_KEY=<insert your Azure storage account access key here>
    restart: "no"

  video-streaming:
    image: video-streaming
    build: 
      context: ./video-streaming
      dockerfile: Dockerfile
    container_name: video-streaming
    ports:
     - "4001:80"
    environment:
      - PORT=80
      - VIDEO_STORAGE_HOST=video-storage    <-- docker compose의 내부 네트워크 설정에 따라 container_name을 찾아간다
      - VIDEO_STORAGE_PORT=80
    restart: "no"
```

docker compose를 사용하지 않았다면 azure-storage, video-streaming 컨테이너를 개별적으로 실행, 종료, 관리했어야 했고 네트워크를 개별로 실행시키는 번거로움이 발생했다. docker compose를 사용하여 하나의 파일에 같이 관리함으로써 번거로움을 줄일 수 있었다.

## 데이터베이스를 앱에 추가하기

현재 제공하고있는 비디오에 대한 메타데이터(비디오 경로, 비디오 정보 등)를 저장하기 위해 mongo db를 사용한다.

mongo db는 다루기 쉽고, 다양한 구조적 데이터를 저장할 수 있다. 다양한 구조적 데이터를 저장할 수 있으며 용량 확장이 매우 쉽다.

mongo db를 추가하기위해 docker-compose.yml 파일을 수정한다.

```
version: '3'
services:

  db:
    image: mongo:4.2.8
    container_name: db
    ports:
     - "4000:27017"
    restart: always

  azure-storage:
      ...

  video-streaming:
    image: video-streaming
    build: 
      context: ./video-streaming
      dockerfile: Dockerfile
    container_name: video-streaming
    ports:
     - "4002:80"
    environment:
      - PORT=80
      - DBHOST=mongodb://db:27017        <-- mongo db 설정 정보
      - DBNAME=video-streaming
      - VIDEO_STORAGE_HOST=video-storage
      - VIDEO_STORAGE_PORT=80
    depends_on:
      - db
    restart: "no"
```

몽고디비에 테스트용 데이터를 넣기 위해 Robo 3T라는 몽고디비 클라이언트를 사용한다.

### 운영 환경에서의 데이터베이스 서버 추가

운영 환경에선 쿠버네티스 클러스터에 별도의 데이터베이스를 배포, 운영한다면 데이터베이스를 쉽게 배포할 수 있다. 데이터베이스를 구성할 땐 안정성을 위해 서비스를 운영하는 클러스터와는 별개의 클러스터에서 구성/운영하거나 클라우드엣더 제공하는 매니지드 서비스를 이용하는 것도 스타트업에서 비용, 시간적인 측면에서 많은 도움이 될수있다. 클러스터 구성 방법은 7장에서 진행

### 마이크로서비스 - 데이터베이스 또는 앱 - 데이터베이스

객체 지향 프로그래밍에서 객체 안에 데이터를 캡슐화하듯, 마이크로서비스에선 각각의 서비스가 자신만의 데이터베이스를 갖고 데이터를 캡슐화하는 것이 좋다. 각기 다른 서비스가 데이터베이스를 공유해서 사용하거나 통합해서 사용한다면 설계와 확장성 측면에서 문제를 만날 수 있다. 하지만 설계 원칙은 목표 달성을 위해서 가끔씩 깨어질 수 있으며 데이터베이스를 공유해야 하는 등의 상황이 생기면 왜/정말 공유가 필요한지 신중하게 고민해야 한다.
