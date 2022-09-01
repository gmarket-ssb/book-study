# 마이크로서비스 간의 통신

마이크로서비스 앱은 보통 서로 자신의 영역을 책임지고 있는 여러개의 마이크로서비스로 구성되어 있으며 협업을 통해 복잡한 동작을 해낸다. 협업을 위해선 서로 통신할 방법이 필요하다.

마이크로서비스간 협업을 위해 필요한 도커, 도커 컴포즈를 다시 살펴보고, 개발 생산성 향상을 위해 라이브 리로드를 구성한다. 이전 챕터에서 마이크로서비스간 HTTP 요청을 통해 서비스를 제공했다면, 이번엔 직접 메시징(direct messaging)으로 확장해보고, 이를 RabbitMQ를 사용해 간접 메세징(indirect messaging)으로 변경하는 것을 진행한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcvh4fY%2FbtrLaeFt0vp%2FNn7Sz2E7XqDZCIlLHPxJg1%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F1pAuY%2FbtrK90ncVdm%2F4IRWJ2vRo33ZsUbTqBmD6k%2Fimg.png)


## 히스토리 마이크로서비스 소개

사용자가 시청한 내용을 기록하는 히스토리 서비스를 추가한다. viewed 라는 메세지 이름을 가진 메세지를 video-streaming 마이크로서비스가 history 마이크로서비스에게 전달하고 history 마이크로서비스는 이를 데이터베이스에 기록한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcf8t8B%2FbtrK7CNVmXQ%2FSlGukuK7PdVqSADYdEh3H1%2Fimg.jpg)
