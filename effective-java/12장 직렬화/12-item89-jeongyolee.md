## [item 89] 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라.

## 핵심정리
> 불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 열거타입을 사용하자.<br>
> 여의치 않은 상황에서 직렬화와 인스턴스 통제가 모두 필요하다면 readResolve 메서드를 작성해서 넣어햐 하고,
<br>그 클래스에서 모든 참조 타입 인스턴스 필드를 transient로 선언해야한다.


### 인스턴스가 하나만 만들어짐을 보장하는 클래스
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}

    public void leaveTheBuilding() {...}
}
```
implements Serializable을 추가하는 순간 싱글턴이 아니게된다. 
<br>(어떤 readObject를 사용해도 초기화때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 된다.)

---

### 싱글턴 속성을 유지하는 방법 - readResolve() 메서드

readResolve 메서드를 사용하면 readObject 메서드가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다. <br>
역직렬화 후 새로 생성된 객체를 인수로 readResolve가 호출되고, 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환한다.

---

### 필드를 transient로 선언하지 않으면 발생하는 취약점
readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다.
이렇게 하지 않으면 readResolve 메서드가 수행되기 전에 역직렬화된 객체 참조를 공격할 여지가 남는다.

---

### 잘못된 싱글턴
1) 잘못된 싱글턴 - transient가 아닌 참조 필드
```java
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
  private Elvis() { }

  private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};

  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }

  private Object readResolve() {
    return INSTANCE;
  }
}
```

2) 도둑 클래스
```java
// 싱글턴의 비휘발성 인스턴스 필드를 훔쳐러는 도둑 클래스
public class ElvisStealer implements Serializable {
  private static final long serialVersionUID = 0;
  static Elvis impersonator;
  private Elvis payload;

  private Object readResolve() {
    // resolve되기 전의 Elvis 인스턴스의 참조를 저장
    impersonator = payload;
    // favoriteSongs 필드에 맞는 타입의 객체를 반환
    return new String[] {"A Fool Such as I"};
    
    private static final long serialVersionID = 0;
  }
}
```

3) 직렬화의 허점을 이용해 싱글턴 객체를 2개 생성
<br>(수작업으로 만든 스트림을 이용해 생성)

```java
// 직렬화의 약점을 이용해 싱글턴 객체를 2개 생성한다.
public class ElvisImpersonator {
  private static final byte[] serializedForm = new byte[]{
      (byte) 0xac, (byte) 0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
      // 코드 생략
  };

  private static Object deserialize(byte[] sf) {
        try (ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(sf)) {
      try (ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream)) {
        return objectInputStream.readObject();
      } catch (IOException | ClassNotFoundException e) {
        throw new IllegalArgumentException(e);
      }
  }

  public static void main(String[] args) {
    // ElvisStealer.impersonator 를 초기화한 다음,
    // 진짜 Elvis(즉, Elvis.INSTANCE)를 반환
    Elvis elvis = (Elvis) deserialize(serializedForm);
    Elvis impersonator = ElvisStealer.impersonator;
    elvis.printFavorites(); // [Hound Dog, Heartbreak Hotel]
    impersonator.printFavorites(); // [There is no cow level]

    //==> 서로 다른 2개의 Elvis 인스턴스 생성
  }
}
```

### 해결방법 : 열거 타입 싱글턴
```java
public enum Elvis {
    INSTANCE;

    private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```
위 예제의 필드를 transient로 선언하여 문제를 해결할 수 도 있지만, Elvis 원소를 열거 타입으로 바꾸는 게 더 좋은 해결 방법이다.
<br>
직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해준다.
