견고한 앱을 위해 필요한 것

-   현재 상태를 이해하기 위한 마이크로서비스 모니터링
-   고객을 보호하기 위해 문제가 발생하면 취하는 행동
-   문제가 생길 때 디버깅하고 해결책 적용

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FqgG6C%2FbtrQkuchs00%2FVcAgddcfsHzc0O57CDvIUK%2Fimg.jpg)

-   마이크로서비스가 다수의 복제(replicas)를 가지고 있다.
-   로드 밸런서를 사용해 요청을 인스턴스에게 균등하게 분배한다.
-   장애가 발생하더라도 복제들이 그 인스턴스를 재시작하는동안 버틸 수 있다.
-   모든 마이크로서비스에서 이해할 수 있는 로그를 생성하고, 그것을 합쳐주는 일종의 로그수집 서비스가 필요하다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fm9CSk%2FbtrQjlGVlPQ%2FUqtURJK4AQUT8JKkuv8lfk%2Fimg.jpg)

## 마이크로서비스의 모니터링

개발환경에서의 모니터링

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FFoCef%2FbtrQkG4xrSH%2FziHvn7M8B6LWxm6Akkpu2k%2Fimg.jpg)

자바스크립트의 경우 `console.log()`, `console.error()`를 사용

-   무엇을 로깅할 것인가?
    -   앱에서 계속 발생하는 이벤트와 그 상세 정보
    -   중요한 동작의 성공, 실패 여부
-   무엇을 로깅하면 안되는가?
    -   다른 소스로부터 쉽게 확인할 수 있는 정보
    -   공개할 수 없는 민감 정보
    -   사용자의 개인 정보

오류가 발생하는 기본적인 사례

-   발생할 수 있는 오류 사례
    -   런타임 에러. 마이크로서비스으 치명적인 오류에 의한 예외
    -   잘못된 입력 데이터(잘못된 검증이나 사용자의 입력데이터 오류)
    -   예상하지 못한 조합으로 코드가 동작
    -   써드파티의 오류(예. RabbitMQ, MongoDB, Azure 스토리지 등)

오류는 사용자와 업무에 영향을 최소화할 수 있도록 오류 상태에서 순조롭게 회복할 수 있어야 한다. 이에 맞는 전략 개발이 필요.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FmbjMR%2FbtrQj7O6bzn%2FOktqosupJEJqrs1QVtstVK%2Fimg.jpg)

-   중지와 재시작  
    예상치못한 오류를 잡아내서 프로세스를 재시작하는 것으로 대응.
-   동작 재개  
    예상치 못한 오류를 잡고 프로세스를 지속시키는 것으로 대응. `try-catch`가 이런 방식의 대표적인 예시.

'중지와 재시작'의 업그레이드 버전

```
if 오류가 발생하면
then 로그 출력
     종료(오류코드)
end
```

-   치명적인 오류가 발생했을 때 마이크로서비스를 회복시키려고 시도하는 것은 손상된 상태로 내버려두는 것일 수 있다.

'중지와 재시작' 전략을 기본으로 하되 일부는 '동작 재개'로 설정하는 것이 최선일 수 있다.

### 쿠버네티스에 대한 자신만의 로그 수집

쿠버네티스는 대시보드에서 모니터링을 제공하지만, 로그를 모아주는 기능은 없다. 필요한 경우 자신만의 쿠버네티스 로그 수집기를 만들어야 한다.

쿠버네티스의 데몬셋(DaemonSet)을 이용한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fd0pOdQ%2FbtrQkuJ4zbE%2FmjZEpL032NEgGuraNicGa1%2Fimg.jpg)

-   모든 노드에 배치된 각각의 데몬셋이 팟에서 생성되는 로그파일 표준 출력 스트림을 자동으로 저장한다.
-   aggregation service는 실행중인 컨테이너로부터 모든 로그를 외부의 collection service에 전달한다.
-   collection service는 HTTP 형태로 들어오는 로그를 받아서 데이터베이스에 저장한다.

대규모 기업용 마이크로서비스 모니터링 솔루션으로는 Fluentd, Elasticsearch, Kibana, Prometheus, Grafana가 있다.

-   Fluentd  
    루비로 만든 오픈소스 로깅 및 데이터 수집 서비스. 클라스터 안에서 Fluentd 컨테이너 인스턴스를 만들고 로그를 외부 로그 수집기에 전달할 수 있다. 플러그인 형태로 확장가능하며, 엘라스틱서치로 로그를 전달하는 플러그인이 대표적이다.
-   엘라스틱서치  
    자바로 만든 오픈소스 검색엔진. 로그, 지표, 기타 유용한 데이터를 저장하고 가져올 수 있다.
-   키바나  
    엘라스틱서치를 기반으로 시각화한 대시보드를 제공. 여러 로그 지표를 보고, 검색하고, 시각화할수 있다. 알림도 설정할 수 있다. 유료버전의 경우 웹훅 트리거 등 다양한 추가 옵션등을 제공한다.
-   프로메테우스  
    오픈소스 모니터링 시스템. 시계열 데이터베이스. 일정한 간격으로 마이크로서비스에서 지표 데이터를 가져오고, 문제가 생기면 자동으로 알려주도록 설정할 수 있다.
-   그라파나  
    프로메테우스에서 부족한 시각화 기능을 보완할 수 있는 시각화 도구.

### 쿠버네티스 헬스 체크를 사용한 자동 재시작

쿠버네티스는 문제가 있는 컨테이너를 재시작한다. 여기서 '문제 상태'를 사용자가 정의할 수 있는 확장 포인트를 제공하는데, 준비 상태 점검(readness probe)과 동작 상태 점검(liveness probe)가 있다.

-   readness probe : 마이크로서비스를 시작했고, 요청을 받을 준비가 되어있는지 점검
-   liveness probe : 마이크로서비스가 여전히 살아있고, 요청을 잘 받을 수 있는 상태이다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FBinHX%2FbtrQkFYP95z%2FyZHqmFoLMaZkCrOu55IObK%2Fimg.jpg)

### 여러개의 마이크로서비스 추적하기

여러 마이크로서비스에서 실행되는 하나의 프로세스를 추적할 수 있다. CID(Correlation ID)를 생성해서 로깅을 남기면 추적이 가능한대 대표적인 도구로는 집킨이 있다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FtJwnL%2FbtrQiiqbrip%2F4id5FbuD6kgAmKytVjTpU1%2Fimg.jpg)
