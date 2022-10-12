> 11. 파드 디자인 패턴과 잡(Job) 실행
> 12. 헬름 차트를 활용한 쿠버네티스 애플리케이션 패키지 배포


# 파드 디자인 패턴과 잡(Job) 실행
대표적인 쿠버네티스 구조(파드 디자인) 패턴에는 사이트카 패턴, 어댑터 패턴, 앰배서더 패턴이 있습니다.
<br><br>

## 사이드카 컨테이너 개념과 실습
**사이드카 패턴**: 메인 애플리케이션은 변경하지 않고 독립적으로 동작하는 별도의 보조 애플리케이션을 붙이는 패턴<br>
**사이드카 컨테이너**: 쿠버네티스에서는 기존 컨테이너의 변경 없이 파드의 기능을 확장시키고 향상시키는 컨테이너, 파일 시스템을 공유하는데 목적이 있다.

<img src="https://user-images.githubusercontent.com/57446639/195340880-1e21bada-957e-4712-82c3-13fbf46ae795.png" width="500"/>

쿠버네티스 공식 홈페이지에 나와있는 이 그림은
1. app-container 가 파일 시스템에 파일을 저장하면
2. streaming-container 에서 자체 `stdout(표준 출력)` `stderr(표준 에러)` 으로 스트리밍하여 로그를 출력하는 구성되어 있다.

이 방법을 사용하면 애플리케이션의 다른 부분에서 여러 로그 스트림을 분리할 수 있다고 함.
<br><br>


## 어댑터 컨테이너 개념과 실습
**어댑터 패턴**: 클래스의 인터페이스를 사용자가 기대하는 다른 인터페이스로 변환하는 패턴으로, 호환성이 없는 인터페이스 때문에 함께 동작할 수 없는 클래스들이 함께 작동하도록 해준다.<br>
**어댑터 컨테이너**: 쿠버네티스에서는 원본 컨테이너에 대한 변경사항 없이 현재 컨테이너 기능을 시스템에 적용시키는 컨테이너

<img src="https://user-images.githubusercontent.com/57446639/195345566-3c4a5a55-9197-4ced-826a-689ff0bdc6cf.png" width="400"/>
<img src="https://user-images.githubusercontent.com/57446639/195346535-7cb223d0-0944-4827-8535-e042d37e71a4.png" width="500"/>

이 그림으로 설명을 하자면,
1. simple_log 를 찍은 로그 파일을 읽어서
2. 원하는 형태(Json Format)로 파싱해서 로그를 출력하는 adaptor-container 를 덧붙인 그림입니다.
<br><br>


## 앰배서더 컨테이너 개념과 실습
**앰배서더 패턴**: 소비자 서비스 또는 응용 프로그램을 대신하여 네트워크 요청을 전송하는 도우미 서비스를 만드는 패턴
**앰배서더 컨테이너**: 파드 외부의 서비스에 대한 액세스를 간소화하는 특수 유형으로, 파드에 앰배서더 컨테이너를 배치하여 통신을 대신해주는 역할을 소화해줌

<img src="https://user-images.githubusercontent.com/57446639/195348164-7cd94742-f92c-492f-a69c-5233700dcaa5.png" width="400"/>
<img src="https://user-images.githubusercontent.com/57446639/195348313-af460c4d-8d6d-4d50-aa82-43a19daf8468.png" width="500"/>

이 그림으로 설명을 하자면,
1. main-application-container 에서 요청을 받으면 직접 처리하지 않고, ambessador-container 를 호출하고
2. ambessador-container 는 remote-service 를 호출하는게 핵심인 그림입니다. (proxy 와 비슷함)
<br><br>


## Job 을 활용한 파드 생성 개념과 실습
Job 과 CronJob 은 파드를 일회성/반복적/정해진 횟수만큼 예약실행을 할 수 있도록 해준다.

Job: 하다 이상의 파드를 만들고 지정된 수의 파드가 성공적으로 종료될 때까지 Pods 실행을 계속 재시도 함
- Job 을 삭제하면 해당 Job 이 생성한 파드들은 정리된다.
- Job 을 일시 중단하면 작업이 다시 재개될 때 까지 활성 파드는 삭제된다.
- Job 을 사용하여 여러 파드를 병렬로 실행을 할 수 있다.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-parallelism
spec:
  completions: 5 #목표 완료 파드 개수 (첫 번째 파드가 실행되고 완료하고 죽으면 두 번째 파드가 실행되고...)
  paralleism: 2 #동시 실행 가능 파드 개수
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4 #실패시 재시도 하는 제한 횟수
```

**잡에 대한 병렬 실행 방법**
- 비 벙렬 잡: df, 일반적으로 파드가 하나만 실행되고 파드가 종료되면 Job 이 완료됨
- 정해진 횟수를 반복하는 잡: `.spec.completions` 에 0이 아닌 양수를 지정하면 정해진 횟수까지 파드가 반복적으로 실행됨
- 병렬 실행 가능 수를 지정하는 잡: `.spc.parallelism` 에 0이 아닌 양수를 지정하면 정해진 개수만큼 파드가 동시에 실행이 가능
<br><br>


## CronJob 을 활용한 예약 파드 생성 개념과 실습
CronJob 은 반복 일정에따라 Job 을 만든다.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-1
spec:
  concurrencyPolicy: Allow # 동시 실행 가능
  schedule: "*/1 * * * *" # 매분마다 컨테이너를 실행
  jobTemplate:
    ...
```

**동시성 정책 설정**
- Allow: 중복 실행을 허용(df)
- Forbid: 중복 실행을 금지
- Replace: 현재 실행중인 크론잡을 내리고 새로운 크론잡으로 대체
<br><br>
<br>



# 헬름 차트를 활용한 쿠버네티스 애플리케이션 패키지 배포

## helm 소개와 설치
## 공개 레포지토리를 활용한 애플리케이션 배포
## 새로운 차트 생성과 실행
## 차트 패키징 및 github 레포지토리를 활용한 배포
