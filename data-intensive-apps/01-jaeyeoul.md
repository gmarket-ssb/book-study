# 01 신뢰할 수 있고 확장 가능하며 유지보수하기 쉬운 애플리케이션
## 개요
오늘날의 애플리케이션은 계산중심(Compute-intensive)가 아닌 데이터 중심(Data-intensive)적이다.
- CPU 성능 보다는 데이터의 양, 데이터의 복잡도, 데이터의 변화 속도가 어플리케이션의 한계를 제한하는 요소다.

일반적으로 데이터 중심 애플리케이션은 표준 구성 요소로 만든다. 하지만 대부분의 엔지니어들은 구성요소들을 직접 처음부터 만들지는 않는다.
하지만 요구사항에 맞춰서 실제 구현체들의 특성을 이해하고 적용해서 문제를 해결해야한다.

일반적인 표준구성요소 (성공적으로 추상화되어서 당연한 것들이라고 읽힌다.)

- 데이터베이스 : 구동 애플리케이션이나 다른 애플리케이션에서 나중에 다시 데이터를 찾을 수 있게 데이터를 저장
- 캐시: 읽기 속도 향상을 위해 값비싼 수행 결과를 기억
- 검색색인(Search Index): 사용자가 키워드로 데이터를 검색하거나 다양한 방법으로 필터링 할 수 있게 제공
- 스트림처리(Stream Processing): 비동기 처리를 위해 다른 프로세스로 메시지 보내기
- 일괄처리(Batch): 주기적으로 대량의 누적된 데이터를 분석


이책에서 소개된 도구의 공통점과 차이점을 알아보고 어떻게 구현했는지를 알아보자.
이를 통해 시스템의 원칙과 실용성 그리고 개발하는 방법을 알아보자.

---
## 데이터 시스템에 대한 생각

데이터의 저장과 처리를 위한 도구들은 이제 분류간 경계가 흐려지고 있다.
- RDB, Message Queue, Cache 등등

Redis는 데이터스토어의 역할로 캐시로 많이 사용되지만, 메시지 큐로도 사용된다.
- https://github.com/OptimalBits/bull

<img width="862" alt="image" src="https://user-images.githubusercontent.com/8626130/185163011-68a5aa5c-fcb3-4f3e-8363-d7744bbfdbe1.png">

메시지 큐(메시지브로커) 아파치 카프카는 데이터베이스처럼 지속성을 보장하기도 한다.

----
> Apache Kafka
> LinkedIn에서 최초로 만들고 opensource화 한 확장성이 뛰어난 분산 메시지 큐(FIFO : First In First Out)

- 분산 아키텍쳐 구성, Fault-tolerance한 architecture(with zookeeper), 데이터 유실 방지를 위한 구성이 잘되어 있음
- AMQP, JMS API를 사용하지 않은 TCP기반 프로토콜 사용
- Pub / Sub 메시징 모델을 채용
- 읽기 / 쓰기 성능을 중시
- Producer가 Batch형태로 broker로 메시지 전송이 가능하여 속도 개선
- 파일 시스템에 메시지를 저장하므로, 데이터의 영속성 보장
- Consume된 메시지를 곧바로 삭제하지 않고 offset을 통한 consumer-group별 개별 consume가능

<img width="729" alt="image" src="https://user-images.githubusercontent.com/8626130/185168474-4f4b327e-ecc9-4937-9363-2dfa1e861b56.png">

----

단일 어플리케이션에서는 데이터 처리와 저장 모두를 만족시킬 수 없는 광범위한 요구사항이 빈번해짐
-> 단일 프로세스에서 수행 할 수 있는 작업 단위로 분리 후 각 도구(프로세스)를 결합하여 이용한다.
-> 개발자가 로직을 짜는 작업에서 데이터 시스템 설계의 역할까지 진행하게 됨

<img width="808" alt="image" src="https://user-images.githubusercontent.com/8626130/185170497-ac152991-249f-41b9-94d1-c7089f36a8b3.png">

데이터 시스템, 서비스를 설계 할 때는 아래 세가지를 주요 관심사로 둬야 한다.
- 신뢰성
- 확장성
- 유지보수성

----

## 신뢰성

무언가 잘못되더라도 지속적으로 올바르게 동작함

- 애플리케이션은 사용자가 기대한 기능을 수행한다.
- 시스템은 사용자가 범한 실수나 예상치 못한 소프트웨어 사용법을 허용 할 수 있다.
- 시스템 성능은 예상된 부하와 데이터 양에서 필수적인 사용 사례를 충분히 만족한다.
- 시스템은 허가되지 않은 접근과 오남용을 방지한다.

잘못 될수 있는 일을 결함(fault)라고 부른다. 이를 예측하고 대처 할 수 있는 시스템을 내결함성(Fault-tolerant) 또는 탄력성(resilient)를 지녔다고 한다.

결함은 장애(Failure)와 동일하지 않다. 결함은 기획(사양) 에서 벗어난 요소지만 장애는 시스템 전체가 멈추는 등 서비스 제공에 문제가 있는 경우다.

고의적으로 내결함성을 훈련하고 테스트 하는 방법으로 넷플릭스의 카오스 몽키가 있다.

<img width="620" alt="image" src="https://user-images.githubusercontent.com/8626130/185172452-ec45d462-580e-4af6-aeab-b0babc5a64a4.png">

- https://github.com/codecentric/chaos-monkey-spring-boot

### 해결책이 있는 결함유형
- 하드웨어결함 : 각 하드웨어 구성요소의 이중화 -> 디스크 raid 구성, 이중전원, hot-swap 가능한 cpu, 데이터센터의 예비 발전기 구축
- 소프트웨어 : 프로세스 격리, 프로세스 재시작 허용, 빈틈없는 테스트, 모니터링, 로깅, 주의깊게 생각하기..
- 인적오류 : 사람이 문제다. 하드웨어는 고작 10~25%만 원인이다. 추상화가 잘된 API 사용하여 내부 로직을 위부로 부터 캡슐화 하여 안전하게 하기, 시스템 환경 격리 운영, 단위테스트-E2E 테스트 까지 운영하기 및 테스트 자동화, 배포 롤백도구 이용, APM 등 모니터링 도구, 교육하기 


----
## 확장성

증가한 부하에 대처하는 시스템 능력으로, "추가 부하를 다루기 위한 계산 자원의 투입 방안", "시스템이 특정 방식으로 커지면 이에 대처하기 위한 선택은 무엇인가?"를 다룬다.

### 부하 기술하기
- 시스템의 현재 부하 상태를 아는것이 중요하다. 그래야 추후 얼마나 필요 한지 알 수 있다.
- 웹서버의 TPS, 데이터베이스의 읽기 대 쓰기 비율, 동시 활성사용자, 캐시 적중률 등임

#### 트위터
방안 1. RDB를 이용하여 내가 팔로우 하고 있는 사람의 피드 조회 후 노출 
방안 2. 수신 사용자별 타임라인 캐시 운영
<img width="809" alt="image" src="https://user-images.githubusercontent.com/8626130/185177890-b6d93ee0-fe83-4926-b0b0-59df3e01842a.png">

방안 1과 2 둘중 어느것이 효율적인가?
- 1의 경우는 매우느림
- 2의 경우에는 특정 유저는 팔로우가 3000만명이다. 그 사람의 타임라인 캐쉬에는 3000만명의 피드가 가득하다.
- 결론. 혼합형으로 운영한다. 팔로워가 많은 사람은 RDB를 이용한다.

### 성능 기술하기
기술된 부하를 기준으로 현재 처리량과 응답시간등의 성능을 확인한다.

추가지연을 유발 할 수 있는 원인
- 백그라운드의 컨텍스트 스위치
- 네트워크 패킷손실 및 TCP 재전송
- 가비지 컬렉션 휴지(GC Pause)
- 디스크에서 읽기를 강제하는 페이지 폴드(Page fault)

APM의 평균시간도 참고 하지만 95분위, 99분위, 99.9분위 (P95, p99, p999) 값도 확인하자.
P999는 요청 1000개 중 1개의 경우를 말한다.

<img width="795" alt="image" src="https://user-images.githubusercontent.com/8626130/185180892-8507ebf5-6417-4b92-88d6-2f2d911416f4.png">

- 보통 응답시간이 가낭 느린 요청을 경험한 고객이 많은 데이터를 들고 다니는 고객이다. 구매를 많이 하는 고객 일 수 있다.
- 아마존은 응답시간이 100ms 증가하면 판매량이 1% 줄어들고 1초가 느려지면 고객만족도가 16% 감소하는 결과를 찾았다.

선두 차단(Head of Line Blocking)
- 큐 대기 지연(Queueing delay)는 응답시간의 상당부분을 차지한다. 
- 서버는 병렬로 소수의 작업을 처리 할 수 있기 때문에(CPU 코어수의 한계) 소수의 느린 처리로도 후속 요청 처리가 지체된다.
- 이 때문에 인위적으로 부하를 생성시에는 지속적으로 요청을 보내야 한다.

<img width="800" alt="image" src="https://user-images.githubusercontent.com/8626130/185181049-176d4252-0983-4701-8941-b0e9969fd0b1.png">


### 부하 대응 접근방식
- Scale out
- Scale Up

----
## 유지보수성
- 운용성 : 시스템이 원할 하게 운영할 수 있게 쉽게 만들기
- 단순성 : 시스템 복잡도를 최대한 제거하고 새로운 엔지니어가 시스템을 이해하기 쉽게 만들기
- 발전성 : 시스템을 쉽게 변경 할 수 있게하라.

### 운용성
좋은 운영팀의 책임
- 시스템 상태 모니터링후 문제 발견시 빠르게 서비스 복원 가능해야함
- 시스템 장애 및 성능 저하의 문제원인 추적
- 보안패치등 소프트웨어와 플랫폼 최신상태유지
- 용량계획, 시스템간 영향도 분석
- 개인 인사이동에도 시스템에 대한 조직의 지식보존

### 단순성
- 응집도는 높이고 결합도는 낮추자.
- 추상화하자..

### 발전성
- 애자일하게 운영하며 변하는 것에 민첩하게 움직이자.
- TDD 리팩토링

----
## 정리하며
안타깝게도 애플리케이션을 신뢰할 수 있고 확장 가능하며 유지보수하기 쉽게 만들어주는 간단한 해결책은 없다.

하지만 특정 패턴과 기술이 도움을 줄 수 있다.

다음 장에서부터 몇가지 예제를 살펴보자.