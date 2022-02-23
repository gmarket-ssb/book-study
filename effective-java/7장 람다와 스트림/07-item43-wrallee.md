## 람다보다는 메서드 참조를 사용하라

람다는 함수형 익명클래스를 간결하게 구현하는 방식이다. 람다를 더 간결하게 만드려면 메서드 참조를 사용한다.  
보통은 메서드 참조를 사용하는게 더 낫다. 하지만 때로는 람다가 더 나을 때도 있으니 잘 고민하며 사용하자.

<br>

**[메서드 참조를 쓰는게 나은 경우]**

```java
map.merge(key, 1, (count, incr) -> count + incr);
```

위 람다를 아래와 같이 메서드 참조로 대체할 수 있다.

```java
map.merge(key, 1, Integer::sum);
```

<br>

**[람다를 쓰는게 더 나은 경우]**

```java
service.execute(GoshThisClassNameIsHumongous::action);
```

이렇게 클래스 이름이 너무 긴 경우는 람다를 사용하는게 더 간결하다.

```java
service.execute(() -> action());
```

또한 `Function::indentity` 보다는 `(x -> x)` 가 더 직관적이다(두 함수의 기능은 같다).

<br>

추가로 메서드 참조의 유형은 아래와 같이 다섯가지로 분류할 수 있다.

| 메서드 참조 유형   | 예                                               | 같은 기능을 하는 람다                                   |
| ------------------ | ------------------------------------------------ | ------------------------------------------------------- |
| 정적               | Integer::parseInt                                | str -> Integer.parseInt(str)                            |
| 한정적(인스턴스)   | Instant then = Instant.now();<br />then::isAfter | Instant then = Instant.now();<br />t -> then.isAfter(t) |
| 비한정적(인스턴스) | String::toLowerCase                              | str -> toLowerCase()                                    |
| 클래스 생성자      | TreeMap<K, V>::new                               | () -> new TreeMap<K, V>()                               |
| 배열 생성자        | int[]::new                                       | len -> new int[len]                                     |

**[Example]**

```java
public static void main(String[] args) {
    String result = "AAA";
    String aaa = Stream.of(2560L, 160L, 10L) // 16진수로 AAA
        .reduce(Long::sum) // 정적
        .map(Long::toHexString) // 비한정적(인스턴스)
        .filter(result::equalsIgnoreCase) // 한정적(인스턴스)
        .orElseThrow();
    
    System.out.println(aaa);
}
```



### 결론

메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하자.



### 기타

따라서 람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없다.  
하지만 단 하나의 유일한 <ins>예외가 있으니 참고</ins>하자. 그 예외는 바로 제네릭 함수 타입이다.

```java
@FunctionalInterface
interface GenericFunction {
    <E extends Exception> String methodCall() throws E;
}

class Tester {
    void test() {
        // Using Method Reference
        Function<GenericFunction, String> fun = GenericFunction::methodCall;
        
        // Lambda Expression
        Supplier<String> stringCallable = () -> ???; // 어떻게 표현해야 할 지 모르겠다
    }
}

```

제네릭 람다식이라는 문법이 존재하지 않기 때문에 제네릭 함수를 구현하고 싶다면 별도의 함수형 인터페이스를 정의해야 한다. 사실 람다는 앞서 말했듯이 함수형 익명클래스를 간편하게 활용하는 방법이고, 메서드 참조는 기존에 존재하는 메서드를 호출하는것이다. 따라서 람다와 메서드 참조는 함수를 활용한다는 점에서는 유사하지만 사실상 별개이다.

그래서 이 예시는 메서드 참조와 연관짓기보다는 "람다가 못하는것"으로만 정리해두면 될 것 같다.
