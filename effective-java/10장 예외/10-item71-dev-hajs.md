## 필요 없는 검사 예외 사용은 피하라
* Checked Exception 은 발생한 문제를 프로그래머가 처리하여 안전성을 높이게끔 해주지만, 과하게 사용하면 오히려 쓰기 불편한 API 된다.
<br>

### 검사 예외와 비검사 예외 선택 기준
API 를 제대로 사용해도 발생할 수 있는 예외 이거나 프로그래머가 의미 있는 조치를 취할 수 있는 경우라면 Checked Exception 을,<br>
둘 중 어디에도 해당하지 않는다면 비검사 예외를 사용하는 게 좋다.
<br><br>

### Checked Exception 회피하는 방법 1
> 비검사 예외 사용
```java
// 비검사 예외를 호출한다.
} catch(TheCheckedException e) {
  throw new AssertionError();
}
```
```java
// 에러 스택 코드를 출력하고 시스템을 종료한다.
} catch (TheCheckedException e) {
  e.printStackTrace();
  System.exit(1);
}
```
* 두 코드는 모두 프로그래머가 Checked Exception 을 책임지는 사례다.
* 이를 비검사 예외로 API 를 만들었으면 됐을 것이다.
<br>

### Checked Exception 회피하는 방법 2
> Optional 사용하기
* Checked Exception 을 던지는 대신 단순히 빈 옵셔널을 반환하면 된다.
* 다만, 예외가 발생한 이유를 알려주는 부가 정보를 담을 수 없다는 단점이 존재한다.
<br>

### Checked Exception 회피하는 방법 3
> 메소드 쪼개기
```java
// before
try {
  obj.action(args);
} catch (TheCheckedException e) {
  ... // 예외 상황에 대처한다.
}

// after
if (obj.actionPermitted(args) {
  obj.action(args);
} else {
  ... // 예외 상황에 대처한다.
}
```
* Checked Exception 을 던지는 메서드를 두 개로 쪼개서 UnChecked Exception 으로 바꾸었다.
  * `.actionPermitted()` : 예외가 던져질지 여부를 boolean 반환하는 메소드
  * `.action()` : 기능을 수행하는 메소드
* 더불어 상태 검사 메서드(`.actionPermitted()`) 와 비검사 예외(`//`)로 나누어 더 유연한 메서드가 되었다.
<br>

### 정리
* API 호출자가 예외 상황에서 복구할 방법이 없다면 비검사 예외를 던지자.
* -> 복구가 가능하고 호출자가 그 처리를 해주길 바란다면, 우선 옵셔널을 반환해도 될지 고민하자.
* -> 옵셔널만으로는 힘들 때만 검사 예외를 던지자.
