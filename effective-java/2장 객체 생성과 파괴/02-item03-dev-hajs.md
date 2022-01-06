## private 생성자나 열거 타입으로 싱글턴[^1]임을 보증하라

### 싱글턴을 만드는 방법 3가지
- **public static final 필드 방식의 싱글턴**
  ```java
  public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
  }
  ```
  - private 생성자는 public static final 필드인 `Elvis.INSTANCE` 를 초기화할 때 딱 한 번만 호출하여 싱글턴임을 보장한다.
  - 예외는 있다.
    권한이 있는 클라이언트가 리플릭션 API 인 [AccessibleObject.setAccessible](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/AccessibleObject.html) 을 사용하면 private 생성자를 호출할 수도 있다.
  - 명확하고 간결한 방법

- **public static 정적 팩터리 메서드 방식의 싱글턴**
  ```java
  public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    public static Elvis getInstance() {
      return INSTANCE;
    }
  }
  ```
  - `Elvis.getInstance` 는 항상 같은 객체의 참조를 반환하므로 싱글턴임을 보장한다.
  - 이 방식 역시 위에서 언급한 (동일한)예외는 발생한다.
  - 유연하게 설계를 변경할 수 있는 방법

- **열거 타입 방식의 싱글턴**
  ```java
  public enum Elvis {
    INSTANCE;
  }
  ```
  - Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.
  - 그럼에도 앞선 예제들 보다 간결하고, 추가 노력 없이 직렬화를 할 수 있다는게 핵심이다.
    - Enum 은 항상 하나의 인스턴스만 생성되기 때문이다.
  - 🎯 대부분의 상황에서는 이 방식을 사용하는게 가장 좋은 방법이다.

<br>

### 싱글턴의 직렬화와 역직렬화
> 싱글턴 클래스를 직렬화 할 일은 거의 없겠지만, 불가피하게 직렬화 해야 한다면 아래 조건을 참고해서 싱글턴임을 꼭 보장해주자.
- 싱글턴 클래스를 직렬화[^2] 가능한 클래스로 만들려면 단순히 `Serializable` 을 선언하는 것만으로는 부족하다.
- 모든 인스턴스 필드를 transient(일시적)이라고 선언하고 readResolve 메서드를 제공해야 한다.
  ```java
  public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    
    // 싱글턴임을 보장해주는 readResolve 메서드
    private Object readResolve() {
      // '진짜' Elvis 를 반환하고, 가짜 Elvis 는 가비지 컬렉터에 맡긴다.
      //                    -> readObject 메서드에서 생성된 인스턴스는 GC에 의해 해제
      return INSTANCE;
    }
  }
  ```
  - *transient 라고 선언된 필드는 직렬화 대상에서 제외된다.*
  - *readResolve 를 사용하여 readObject 메소드를 통해 직렬화 된 데이터를 변경할 수 있습니다. (싱글톤을 제외하고서 사용하는 부분이 거의 없음)*
- 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화[^3]할 때 마다 새로운 인스턴스가 만들어진다.

<br>

[^1]: 인스턴스를 오직 하나만 생성할 수 있는 클래스
[^2]: 객체 데이터를 전송이 가능한(바이트) 형태로 만드는 것
[^3]: 전송이 가능한(바이트) 형태로 변환된 데이터를 다시 원래의 형태로 만드는 것
