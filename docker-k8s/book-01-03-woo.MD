# 마이크로서비스를 왜 쓰는가

-   소프트웨어가 커지고 복잡해지면 더 나은 관리 방법과 복잡성을 줄이기 위한 노력, 방법이 필요
-   마이크로서비스는 위와 같은 문제를 해결할 수 있지만 기존 모놀리스 방식보다 어렵고 복잡하며 시간이 많이 소요되는 방식이다.
-   처음엔 모놀리스로 만들자는 의견도 있지만, 앱은 항상 복잡해지는 방향으로 흐르고 결국 확장해야된다. 이 때 마이크로서비스를 변환하는 것은 많은 힘든 상황을 만날 수 있다.

## 이 책에서 만들 것

-   개별적으로 동작하는 마이크로서비스
-   docker를 이용해 마이크로서비스 패키지를 만들고 publish
-   docker compose를 이용해 개발환경에서 마이크로서비스를 개발
-   코드와 앱을 jset, cypress로 테스트
-   mongoDB, RabbitMQ와 같은 서드파티 서버와 통합
-   HTTP, RabbitMQ로 마이크로서비스간 통신
-   마이크로서비스 동작에 필요한 데이터와 파일을 저장
-   terraform을 이용해 kubernetes를 운영
-   terraform을 이용해 deploy
-   CD 파이프라인 구축

## 복잡성 관리

-   마이크로서비스는 작은 단위의 수준별로 복잡성을 관리할 방법을 제시
-   마이크로서비스 앱은 구성 요소의 상호 작용을 통해 자연스럽게 복잡성이 증가하는 일종의 복잡 적응 시스템(Complex Adaptive System)이다.
    -   전체 시스템은 이해하기에 복잡하더라도 각각의 구성요소는 작고, 관리 가능하며, 이해하기 쉽다.
-   여러가지 자동화 도구를 사용하여 복잡성에 압도당하지 않고 더 크게 확장할 수 있다.

## 마이크로서비스란?

개별적인 배포 일정을 갖고 운영이 가능한 작고 독립적인 소프트웨어 프로세스.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbspsTM%2FbtrJ2ooiuUW%2FUhlKCC4kA8qM76NkrfGkWK%2Fimg.png)

-   각각의 마이크로서비스는 상호작용을 위해 외부에서 접근 가능하도록 노출할 수 있지만, 내부 통신만 허용하도록 할 수도 있다.
-   하나의 마이크로서비스는 많은 기능을 갖지 않는다. 여러 마이크로서비스들이 모여 커다란 앱과 같은 기능을 제공한다.

## 마이크로서비스 앱이란?

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcax9ta%2FbtrJXj3zPzM%2FDkGBUrzePezAJQwTWVBC3k%2Fimg.png)

-   분산 애플리케이션(Distributed Application)
-   격리된 프로세스와 네트워크 통신을 하는 물리 또는 가상으로 나뉘어진 작은 구성 요소(Component)
-   클러스터 오케스트레이션 플랫폼 쿠버넷을 사용, 서비스를 배포하고 관리하는 역할
-   클러스터 내부 또는 외부에 있는 데이터베이스

## 모놀리스의 문제점

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FRHtyE%2FbtrJXkO1O9I%2FqTBjsD6Oh8CQ23FgB6xNeK%2Fimg.png)
-   모놀리스
    -   장점
        -   더 만들기 쉽다. 더 적은 기술과 설계 방법이 필요하기 때문.
        -   새로운 앱을 만들 때 좋은 시작점. 마이크로서비스의 많은 기술적 요소 비용을 아끼고, 초기 비즈니스 모델의 유효성을 테스트하기 적합.
        -   버릴 수 있는 실험적 모델을 만들어보기에 적합. 작은 범위 또는 추가 확장이 필요없을 때 빠르게 잘 동작하게 만들려는 앱이 목적일 때.
    -   단점
        -   모든 코드가 같은 프로세스에 동작하므로 코드가 복잡해지는 것을 막을 수 없고 일부만 떼어놓고 작업하는 것이 불가능
        -   담당자 변경 등 여러 손을 거치며 점점 위와같은 상황이 심화됨
        -   코드 한줄 추가/변경에 전체 시스템이 영향을 받으므로 리스크가 커지고 이는 비즈니스 민첩성을 떨어트림.
        -   앱의 용량이 증가할수록 물리적 컴퓨터 자원의 제약에 의해 개발, 테스트에 어려움이 생김. -> 운영 비용 증가
-   많은 앱이 실제로 작다. 지금도 세상에 잘 동작하는 모놀리스가 많다.
-   앱이 작고, 간단하게 남을 수 있고, 특별한 그닝 향상도 필요없다면 모놀리스로 제작하는 것이 답이다.
-   지속적으로 발전, 개선해 나갈 앱은 시작부터 마이크로서비스를 고려하는 것이 낫다. 나중에 전환하는 것은 어렵고 위험한 작업이다.
-   모놀리스도 결국 커다란 단일 서비스.

## 마이크로서비스가 많이 사용되는 이유

-   이전에도 분산 컴퓨팅은 많았지만, 최근 클라우드와 가상화, 가상 인프라 관리 자동화 도구의 발전으로 시스템 구축 비용이 훨씬 더 줄었으며, 분산 시스템의 구성요소들을 이전보다 더 작게 만들어 효율적으로 운영할 수 있게 되어 이를 오늘날 마이크로서비스라고 부르고 있다.

## 마이크로서비스의 장점

-   자신만의 프로세스, 메모리 자원을 가져 인프라의 가성비를 이끌어 낼 수 있음
    -   무거운 서비스를 위한 별도의 자원을 확보해 사용하도록 구성하는 것이 쉬움
-   일부 변경이 전체 애플리케이션의 사이클에 영향을 미치지 않음 -> 빠른 추가, 수정이 가능함
    -   일부의 문제가 전체의 문제가 되지 않음
-   하나의 서비스는 매우 가벼우므로 빠르게 복제하고 빠르게 시작해서 쉽게 확장할 수 있음
-   작은 리소스로 테스트와 문제 해결이 쉬움 -> 전반적인 시스템의 테스트는 여전히 어려울 수 있다.
-   몇일, 몇주만에 제거되거나 재작성 될 수 있을 정도의 크기 -> 언제든지 버려도 큰 리스크가 없는 코드들
-   앱이 특정 기술 스택을 요구하지 않음. 각각의 단위 서비스에 맞는 적절한 기술 스택을 사용, 공유 프로토콜만 통일하면 됨
    -   기술적 향상은 오오래된 기술 스택의 스요를 감소시키다 결과적으로 새로운 것으로 대체됨. MSA는 이런 흐름을 따라가기에 적합

## 마이크로서비스의 단점

-   어렵다
    -   학습 곡선이 높다.

## 마이크로서비스 최신 도구

-   도커 : 패키지를 만들거나 서비스를 배포
-   도커 컴포즈 : 개발환경에서 마이크로 서비스 앱 테스트
-   쿠버네티스 : 클라우드에 앱을 호스트
-   테라폼 : 클라우드 인프라, 쿠버네티스 클러스터를 만들고 앱을 배포

## 마이크로서비스 앱의 설계

규칙

-   미래에 필요한 검증을 위해 지나치게 미리 설계하지 말자. 단순한 설계부터 시작하자
-   최대한 단순하게 유지하기 위해 개발 과정에서 지속적인 리팩토링을 하자
-   좋은 설계가 자연스럽게 완성되도록 하자 => 지속적으로 앱을 개발하고 수명을 관리하며 자연스럽게 달성

마이크로서비스의 원칙

-   단일 책임 원칙  
    작고 단순하며, 하나의 서비스는 개념적으로 하나의 업무를 책임
-   느슨한 연결  
    서비스간에 연결을 최소화하고, 반드시 필요한 정보가 아니면 공유하지 않음. 서비스간 종속성을 최소화하면 문제가 확산되지 않고 개별적인 작업들이 가능 => 유연한 업무 대응
-   강한 응집력  
    하나의 서비스 안에선 서로 기능적으로 연계되어 문제를 해결할 수 있도록 동작. 2개 이상의 문제를 해결하거나 책임을 가진다면 강한 응집력을 가지지 못한 것을 의미

마이크로서비스의 설계를 잘 반영하는 전형적인 예 : DDD(Domain Driven Design)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FohQce%2FbtrJXjWRSJB%2FUK72eSehjqYtv1fdzygPZK%2Fimg.png)

일부 DRY(Don't Repeat Yourself) 문제를 제기할 수 있지만, 해결할 수 있는 좋은 방법이 있음

## 예제 응용 프로그램

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbRtUHW%2FbtrJ0T3UQZr%2FYtQ7yhBYYW6GbqR8cjkRa0%2Fimg.png)

# 첫 마이크로서비스 만들기

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FnJ3m9%2FbtrJ2aqi8yF%2FtdbmmB5qlensHf0078ylH1%2Fimg.png)

3000번 포트로 비디오 서비스를 하는 nodejs 앱 구현

```
// 실행방법
node index.js
```

HTTP 서비스를 제공할 포트번호를 환경변수로부터 불러와 설정할 수 있도록 유연하게 변경

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FnJ3m9%2FbtrJ2aqi8yF%2FtdbmmB5qlensHf0078ylH1%2Fimg.png)

```
// 실행방법
export PORT=3000
node index.js
```
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcewtfU%2FbtrJ2ou5iur%2Fv7vrxqjkvae5inbcwKGtgk%2Fimg.png)

---

-   `node index.js` 방식의 실행보단 `package.json`을 추가하여 `npm start` 라는 명령어를 사용
-   라이브리로딩을 위해 `nodemon`을 설치 => 이런 스크립트는 `package.json`에 `start:dev`와 같은 스크립트를 추가하여 편하게 사용

---

# 첫 마이크로서비스 게시

## 컨테이너의 개념

컨테이너는 운영체제와 하드웨어를 가상화할 방법을 제공하는 도구. 마이크로서비스가 필요로 하는 자원을 추상화한다. 컨테이너는 하나의 쿠분된 자원을 할당하고, 여러 서비스가 그 자원을 공유할 수 있게 해준다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FCiG2I%2FbtrJXSdzLmf%2FvzeppddxkDHLxplZUBkI5k%2Fimg.png)

## 이미지의 개념

일종의 스냅샷. 부팅이 가능한 서버(여기선 마이크로서비스)의 스냅샷. 모든 코드, 종속성 및 실행에 필요한 관련 파일들을 포함하여 앱이 필요할 때 초기화할 수 있도록 컨테이너에 부팅할 준비가 완료된 상태.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FnK3KU%2FbtrJYyzdT9K%2Fp7QGHHAyv8T6NXWFxlAO31%2Fimg.png)

## 도커를 사용하는 이유

도커는 마이크로서비스 패키지를 만들고 게시하는 도구. 유니버설 패키지 관리자, 하나의 패키지 관리자로서 모두를 관리하나는 뜻.  
도커는 시스템 환경을 표준화할 수 있는 좋은 도구. 모든 개발자들이 동일한 개발 환경을 가지고 작업할 수 있게 해주며 운영 환경에서도 마찬가지로 적용됨. 개발환경에서의 코드 동작을 운영환경에서도 보장할 수 있는 가능성을 극대화

## 도커로 수행할 작업

-   마이크로서비스를 도커 이미지로 패키징
-   이미지를 프라이빗 컨테이너 레지스트리에 게시
-   마이크로서비스를 컨테이너에서 실행

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FpxKCk%2FbtrJ2lLSAXP%2FVZpWKIferNXIlQHYFRC8Tk%2Fimg.png)

## 도커로 개발환경 확장하기

[##_Image|kage@bcXbM0/btrJ17mM02q/f9GeW6Z5ZutdbbQbBjrc90/img.png|CDM|1.3|{"originWidth":1920,"originHeight":1080,"style":"alignCenter","width":618,"height":348}_##]

## 마이크로서비스 패키징

1.  마이크로서비스에 관한 도커 파일을 만든다.
2.  도커 이미지 형태로 마이크로서비스를 패키징한다.
3.  컨테이너에서 부팅하고 게시된 이미지를 테스트한다.

-   도커파일, Dockerfile : 도커로 만들 이미지의 세부 사항을 포함하는 하나의 스크립트 파일

![(https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbcXbM0%2FbtrJ17mM02q%2Ff9GeW6Z5ZutdbbQbBjrc90%2Fimg.png)

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
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbspsTM%2FbtrJ2ooiuUW%2FUhlKCC4kA8qM76NkrfGkWK%2Fimg.png)

## 마이크로서비스 게시하기

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbEUM33%2FbtrJZJHjRqX%2FQr7CxXFXi7HEdU3PI8EXW1%2Fimg.png)

1.  애저에 있는 private container registry를 생성한다.
2.  publish 하기 전 `docker login` 명령어를 사용해 인증한다.
3.  `docker push` 명령어로 이미지를 업로드한다.
4.  `docker run`으로 게시된 이미지 마이크로서비스가 시작됬는지 확인한다.

```
docker login rura6502.azurecr.io --username rura6502 --password {PASSWORD}
docker tag video-streaming rura6502.azurerc.io/video-streaming:latest

docker push rura6502.azurecr.io/video-streaming:latest

docker rmi rura6502.azurecr.io/video-streaming:latest
docker pull rura6502.azurecr.io/video-streaming:latest
docker run --rm -p 3000:3000 rura6502.azurecr.io/video-streaming:latest
```
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbRhQoA%2FbtrJ2pt0HzU%2F3wQZN332VJH9lIqHFKrCs0%2Fimg.png)
