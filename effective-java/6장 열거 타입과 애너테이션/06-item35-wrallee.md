## ordinal 메서드 대신 인스턴스 필드를 사용하라

Enum 타입은 내부적으로 열거타입 상수가 몇번째인지를 반환하는 ordinal 메서드를 제공한다.

**결론부터 말하면 대부분의 프로그래머는 이 메서드를 쓸 일이 없으며, 이 메서드는 EnumSet이나 EnumMap같은 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다. 따라서 이런 용도가 아니라면 ordinal 메서드는 절대 사용하지 말자.**

<br>

만약 Enum 상수값에 숫자나 문자 등의 값을 저장하고 싶다면 인스턴스 필드를 선언하여 사용하자.  
다음은 앙상블의 형태를 정의하고 사람 수를 저장하는 열거 타입이다.

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

위와 같이 구현하면 된다.

잘못된 예시로 다음과 같이 구현하는 경우가 있는데 <ins>절대 따라해서는 안된다.</ins>

```java
public enum Ensemble {
    ...
    public int numberOfMusicians() { return ordinal() + 1; }
}
```

위와 같이 구현하는 것은 다음과 같은 문제를 만들어낸다.

- 열거타입 상수의 순서가 바뀌면 밸류값도 바뀌어 버린다.
- `OCTET(8)`, `DOUBLE_QUARTET(8)`을 동시에 정의할 수 없다.
- `TRIPLE_QUARTET(12)`을 정의하려면 `???(11)`같은 더미를 필수로 추가해줘야 한다.

<br>

> 그러므로 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드에 저장하자.

