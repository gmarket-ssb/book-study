## 필요 없는 검사 예외 사용은 피하라 -> 
* **의미 없는 예외 처리는 하지 말자.**
* Checked Exception 은 발생한 문제를 프로그래머가 처리하여 안전성을 높이게끔 해주지만, 과하게 사용하면 오히려 쓰기 불편한 API 된다.
<br>

```java
} catch(TheCheckedException e) {
  throw new AssertionError(); // 일어날 수 없다!
}
```
* catch 문에서는 잡은 `TheCheckedException` 에 대해서 처리해야 하는데 새로운 Error 를 던지는건 옳은 예외 선택이 아니다.
<br>

### Checked Exception 을 권장하는 상황
1. API 를 제대로 사용해도 발생할 수 있는 예외 이거나
2. 프로그래머가 의미 있는 조치를 취할 수 있는 경우
 
-> 둘 중 어디에도 해당하지 않는다면 비검사 예외를 사용하는 게 좋다.
<br><br>

예외를 던지는 상황 중에서도 단 하나의 Checked Exception 을 던질 때 프로그래머는 큰 부담이 크다.
* 하나 뿐인 예외로 인해 try 블록을 추가해야 하거나
* 스트림에서 직접 사용을 하지 못하기 때문
<br>

### Checked Exception 을 회피하는 방법
1. 적절한 결과 타입을 담은 Optional 을 반환
2. 그대로 Error 를 사용하는 방법
3. Checked Exception 를 던지는 메서드를 두 개로 쪼개서 UnChecked Exception 으로 바꾸는 방법
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
    * 여기서는 상태 검사 메서드(`.actionPermitted()`) 와 비검사 예외(`//`)로 나누어 더 유연한 메서드가 되었다.
