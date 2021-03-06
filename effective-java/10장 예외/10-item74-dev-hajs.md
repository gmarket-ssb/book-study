## 메서드가 던지는 모든 예외를 문서화하라


### 예외를 다루는 방법
메서드가 던지는 예외는 그 메서드를 올바로 사용하는 데 아주 중요한 정보이기 때문에 공을 들여 문서화 해야 한다.<br>
이때 검사 예외는 항상 따로따로 선언하고, 공통 상위 클래스 하나로 뭉뚱그려 선언하지 말자.
* ex. 메서드가 `Exception` 이나 `Throwable` 을 던지지 않도록 하는 것
<br>

### 문서화 방법
문서화에는 자바독의 `@throws` 태그를 사용하면 되며, 검사 예외만 throws 문에 일일이 선언하도록 하자.
* API 사용자가 해야 할 일이 달라지므로 검사 예외와 비검사 예외를 구분해두는 게 좋다.
<br>

### 예외 상황
한 클래스에 정의된 많은 메서드가 같은 이유로 같은 예외를 던진다면, 클래스 설명에 그 예외를 추가하는 방법도 있다.
```java
/**
 * 이 클래스의 모든 메서드는 인수로 null 이 넘어오면 NullPointerException 을 던진다
 */
public class Item74 {
  ... // method
}
```
