## Architecture
- Monolithic Architecture: 서비스가 하나의 애플리케이션으로 돌아가는 구조
- MicroService Architecture: 애플리케이션의 각각의 기능을 분리하여 개발 및 관리하는 구조
<img src="https://user-images.githubusercontent.com/57446639/183696349-c8ff2c90-c0a1-4090-9e5e-14515b0096a4.png" width="500"/>
<br><br>


분산 시스템 환경을 많이 사용하면서 Transaction 보장, 테스트, 배포, 관리가 복잡해졌다.<br>
이 문제를 해결하기 위해 나온게 "이미지" "도커" "쿠버네티스" 라고 볼 수 있다.
- 도커를 사용하면 컨테이너를 편리하게 쓸 수 있게 해주고
- 쿠버네티스를 사용하면 도커를 잘 다룰 수 있게 해준다.
<br><br>


## 용어 정리
이미지: 필요한 프로그램과 라이브러리, 소스를 설치한 뒤 만든 하나의 파일<br>
컨테이너: 이미지를 격리하여 독립된 공간에서 실행한 가상 환경
- 가상머신처럼 하드웨어를 전부 구현하지 않기 때문에 매우 빠른 실행이 가능하다.
- VM 과 컨테이너의 실행 시 CPU 사용량 비교
  - <img src="https://user-images.githubusercontent.com/57446639/183699034-d5150198-b3e9-43d4-9727-1a49f36aa612.png" width="20%" style="float: right"/>
<br><br>


## Docker
> https://docs.docker.com/
컨테이너 기술을 지원하는 다양한 프로젝트 중에 하나, 사실상 '컨테이너 기술의 표준'<br>
애플리케이션에 국한되지 않고 의존성 및 파일 시스템까지 패키징하여 빌드, 배포, 실행을 단순화
<img src="https://user-images.githubusercontent.com/57446639/183700034-7f3f2729-17eb-4f01-8f3a-c17d59ad96b4.png" width="500"/>

도커를 사용하면, 원하는 서비스를 백업하기도 쉽고 종속성 없이 빠르게 개발할 수 있는 환경을 만들 수 있다.
<br><br>


## Kubernetes, k8s
2014년에 나온 컨테이너 기반 오픈소스 가상화 프로젝트
<br><br>


## Docker 실습
1. docker download
    - https://docs.docker.com/get-docker/
2. docker version check
    - ```$ docker --version```
3. docker image download
    - ```bash
      ## tomcat image search
      $ docker search tomcat
      ## image background run (80 접속 > 8080 포트의 tomcat 포워딩, name: tc)
      $ docker run -d --name tc -p 80:8080 amd64/tomcat
      ## temp run container (stop 시키면 container 가 사라짐)
      $ docker run -d --name tc -p 80:8080 --rm amd64/tomcat
      ```
4. tomcat run check
    - <img src="https://user-images.githubusercontent.com/57446639/183723693-f344d3db-ad84-42e5-b7a2-e7a19c860811.png" width="500"/>
<br><br>

## Docker Registry?
<img src="https://user-images.githubusercontent.com/57446639/183725587-a0a55704-d33c-47bd-bddc-5b47890bbb25.png" width="400"/><br>
- https://hub.docker.com/
- 도커 레지스트리는 누구나 이미지를 올리고 내려받을 수 있다.
- 방금한 실습도 레지스트리에 있는 톰캣 이미지를 찾아서 컨테이너로 변환해서 만들고, 컨테이너가 웹 페이지를 띄워줬다.
- 도커 레지스트리에서 원하는 이미지를 다운로드 하는 방법 : ```$ docker pull mysql```
- 다운로드 받은 이미지를 확인하는 방법: ```$ docker images```
<br><br>

## Docker Life Cycle
<img src="https://user-images.githubusercontent.com/57446639/183859851-2c0fe2d7-9f4f-4086-a3a0-f88574fe0c59.png" width="400"/><br>
- `run = pull + create + start`
  - pull 이 되어 있는 이미지라면 pull 을 또 하지는 않는다.
  - 그렇다고 run 명령어를 계속 날리면, 중복된 컨테이너가 계속 생겨나므로 create 와 start 명령어를 명시적으로 분리해서 사용하는게 좋다.

## Simple Docker Commands
- 기존에 설치된 모든 컨테이너와 이미지 정지 및 삭제
  - ```bash
    ## -a: all, -q: container name
    $ docker stop `docker ps -a -q`
    $ docker rm `docker ps -a -q`
    $ docker rmi `docker images -q`
    ```
- 도커 기능을 사용해 Jenkins 검색
  - ```bash
    $ docker search jenkins
    ```
- Jenkins 를 사용하여 설치
  - ```bash
    $ docker pull jenkins/jenkins
    ## 오픈된 포트 확인
    $ docker inspect jenkins/jenkins
    $ docker run -d --name jenkins -p 8080:8080 jenkins/jenkins
    ```
- Jenkins 포트로 접속하여 웹 서비스 열기
  - ```bash
    $ curl 127.0.0.1:8080
    ```
- Jenkins 의 초기 패스워드 찾아서 로그인하기
  - ```bash
    ## init password 확인, 저장된 path 확인
    $ docker logs jenkins
    $ docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
    ```
