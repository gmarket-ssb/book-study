## 인스턴스화를 막으려거든 private 생성자를 사용하라

static 메서드, static 필드만을 담은 클래스를 만들고 싶을 때가 있음
- 이건 사실 객체지향적인 사고는x
- java.lang.Math, java.util.Arrays 같은 타입, 배열 관련 메서드들을 모아 놓을 수 있음
- 이런 유틸리티 클래스 들은 인스턴스로 만들어 쓰려고 설계한게 아님
- 근데 생성자를 명시하지 않으면 컴파일러가 자동으로 public 생성자를 만듦
- 사용자 입장에선 컴파일러가 만든건지 알 수x
- 컴파일러가 기본 생성자를 만드는 경우는 오직 명시된 생성자가 없을 때 뿐

````java
public class UtilityClass {
    private UtilityClass() {
        // 꼭 에러를 던질 필요는x 혹시라도 클래스 내부에서 호출하지 않도록 주의
        throw new AssertionError();
    }
}
````

- private 생성자는 어떠한 환경에서도 클래스가 인스턴트와 되는것을 막아줌
- 생성자가 있는데 호출을 할 수 없으니 직관적이지 않음 -> 주석 달아주자
  java.lang.Math 에도 초기화 하지 말라고 주석 달려 있음
````java
public final class Math {
    /**
     * Don't let anyone instantiate this class.
     */
    private Math() {}
}
````
- 상속도 불가능
- 모든 생성자는 명시적이든 묵시적이든 상위 클래스의 생성자를 호출하는데, private으로 선언 할 경우
하위 클래스가 상위 클래스의 생성자에 접근할 길이막힘

### 객체 생성 못하게 막으려면 private 생성자를 사용하고 주석을 달아주자