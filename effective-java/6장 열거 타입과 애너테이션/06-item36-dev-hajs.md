# 비트 필드 대신 EnumSet 을 사용하라

### 비트 필드
* 비트 필드를 가지며, 비트별 연산을 사용해 여러 상수를 하나의 집합으로 만들어 진 패턴
  ```java
  public class Text {
    public static final int STYLE_BOLD          = 1 << 0; // 1
    public static final int STYLE_ITALIC        = 1 << 1; // 2
    public static final int STYLE_UNDERLINE     = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    public void applyStyles(int styles) {...}
  }
  ```
* BOLD 이면서 ITALIC 한 글자(3) 표현은 `text.applyStyles(STYLE_BOLED | STYLE_ITALIC)` 으로 할 수 있다.
  * 이는 list, set 과 같은 collection 보다 빠른 연산이 가능하다는 장점이 있..으나
  * 정수 열거 타입의 단점을 그대로 가지고 있고, 출력하면 해석하기가 어렵다는 단점이 있다.
  * 개인적인 견해지만 비트 연산을 모르거나 많이 접해보지 않은 동료가 있다면 ..... ?
<br>

### EnumSet
* 그래서 등장한 클래스
* `java.util.EnumSet` 은 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해주는 클래스이다.
  ```java
  public abstract class EnumSet<E extends Enum<E>> extends AbstractSet<E> implements Cloneable, java.io.Serializable {...}
  ```
  * EnumSet 은 Set 인터페이스를 완벽히 구현하여 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다.<br><br>
* 그럼 EnumSet 을 사용하여 위 코드를 바꿔보자., 짧고 깔끔하고 안전해진다.
  ```java
  public clss Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
    // 클라이언트가 EnumSet 으로 던질걸 알아도, 인터페이스로 받는 게 좋은 습관이다. (아이템 64)
    public void applyStyles(Set<Style> styles) {...}
  ```
  * 짧고, 깔끔하고, 안전해진다.
  * BOLD 이면서 ITALIC 한 글자(3) 표현은 `text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC))` 으로 할 수 있다.
<br>

### 정리
* 대부분의 상황에서 비트 필드를 사용할 이유가 없다.
* EnumSet 사용을 권장한다.
