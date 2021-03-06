# 복구할 수 있는 상황에서는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

자바는 문제 상황을 알리는 타입(throwable)으로 검사 예외, 런타임 예외, 에러, 이렇게 세가지를 제공한다. 100% 명확한 건 아니지만 이럴 때 참고하면 좋은 지침들이 있으니 함께 살펴보자.

## 참고 지침

- 호출하는 쪽에서 복구하리라 여겨지는 상황이라면 검사 예외를 사용하라.
    - 검사 예외는 API 설계자는 API 사용자에게 그 상황을 회복해내라고 요구한 것이다.
    - 비검사 throwable(런타임 예외, 에러)는 프로그램에서 잡을 필요가 없거나 혹은 통상적으로 잡지 말아야 한다.
- 프로그래밍 오류를 나타낼 때는 런타임 예외를 사용하자.
    - 런타임 예외는 대부분은 전제조건을 만족하지 못했을 때 발생한다.
    - (런타임 예외를 포함한) 비검사 throwable을 잡지 않은 쓰레드는 적절한 오류 메시지를 내뱉으며 중단된다.
- Error는 상속하지 말아야 할 뿐만 아니라, throw 문으로 직접 던지는 일도 없어야 한다. (AssertionError는 예외다.)
    - 에러는 보통 JVM이 자원 부족, 불변식 꺠짐 등 더 이상 수행을 계속할 수 없는 상황을 나타낼 때 사용한다.
    - 구현하는 비검사 throwable은 모두 RuntimeException의 하위 클래스여야 한다.
- throwable은 이로울 게 없으니 절대로 사용하지 말자! throwable은 정상적인 검사 예외보다 나을 게 없으면서 API 사용자를 헷갈리게 할 뿐이다.
- 예외 역시 어떤 메서드라도 정의할 수 있는 완벽한 객체이다.
    - 특히 검사 예외는 호출자가 예외 상황에서 벗어나는 데 필요한 정보를 알려주는 메서드를 함께 제공하는 것이 중요하다.

## 핵심정리

- 복구할 수 있는 상황이라면 검사 예외를, 프로그래밍 오류라면 비검사 예외를 던지자.
- 확실하지 않다면 비검사 예외를 던지자.
- 검사 예외도 아니고 런타임 예외도 아닌 throwable은 정의하지도 말자.
- 검사 예외라면 복구에 필요한 정보를 알려주는 메서드도 제공하자.