## 다른 타입이 적절하다면 문자열 사용을 피하라

결론: 더 적합한 데이터 타입이 있거나 새로 작성하는게 나은 상황에선 문자열 사용을 피해라.

<br/>

데이터가 수치형이라면 **int, float, BigInteger** 등 적당한 수치나 타입으로 변환해야 한다. '예/아니오' 질문의 답이라면 적절한 **열거 타입이나 boolean**으로 변환해야 한다. 기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고, 없다면 새로 하나 작성하는게 좋다.



여러 요소가 혼합된 데이터를 하나의 문자열로 표현하는 것은 좋지 않다.

```java
String compoundKey = className + "#" + i.next();
```

실제 시스템에서 위와 같이 사용될 때가 있는데, 구분자 '#' 이 요소에 포함되어 있다면 프로그램이 오동작 할 수도 있다. 또한 개발 요소를 얻으려면 문자열을 파싱해야해서 성능상으로도 문제가 될 수 있다.

또한 equals, toString, compareTo 같은 메서드도 활용할 수 없으므로 차라리 정적 멤버 클래스로 따로 만드는것이 좋다.

<br/>

또한 문자열은 권한(capacity)을 표현하기에 적합하지 않다.  
다음은 `ThreadLocal`을 구현하는 코드인데 `key`를 보통 권한이라고도 한다.

```java
public class ThreadLocal() {
    private ThreadLocal() {}
    
    public static void set(String key, Object value) { ... }
    public static Object get(String key) { ... }
}
```

이 방식은 권한(capacity) 구분을 문자열로 하게 되는데 이로 인해 클라이언트들이 합의 없이 같은 키를 사용하게 된다면 오류가 발생할 수 있다. 또한 의도적으로 같은 키를 사용하여 다른 클라이언트의 값을 가져올 수도 있다.

<br/>

따라서 객체 자체를 Key로 사용하는 다음 방식으로 변경할 수 있다.

```java
public class ThreadLocal {
    private ThreadLocal() {}
    
    public static class Key {
        Key() {}
    }
    
    public static Key getKey() {
        return new Key();
    }
    
    public static void set(Key key, Object value) { ... }
    public static Object get(Key key) { ... }
}
```

이렇게 되면 사실상 ThreadLocal은 Key 생성, ThreadLocalMap에 접근하는 역할뿐인 유틸클래스가 된다. 따라서 ThreadLocalMap에 접근하는 `set`과 `get`을 Key의 인스턴스 메서드로 변경하면 아래와 같이 바뀐다.

```java
public final class ThreadLocal<T> {
    public ThreadLocal() {};
    
    public void set(T value) { ... }
    public T get() { ... }
}
```

이 사례를 통해 권한(capacity)에 있어서 단순 문자열 보다는 Key라는 객체를 만들어 사용하는것이 더 적절함을 알 수 있었다.

여기서는 키 값으로 문자열 대신 객체를 사용함으로써 오류를 방지하는것만 설명했지만 실제로는 위에서 설명했듯이 문자열을 부적절하게 사용하는 것은 파싱으로 인한 문제(성능저하), 각종 유용한 메서드를 사용할 수 없다는 점(유연성 저하)과 같은 문제들도 존재한다.

<ins>따라서 다른 타입을 고려할만 하다면 문자열 사용은 최대한 피하는것이 좋다</ins>.