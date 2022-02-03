## 이왕이면 제네릭 타입으로 만들라

이 장에서는 <ins>제네릭 타입을 만드는 방법</ins>에 대해 설명하고 있다.

또한 이 장에서 제네릭 타입 구현 시 배열을 사용하는데 `ITEM28: 배열보다는 리스트를 우선하라` 와 모순되어 보이지만 해당 아이템과는 별개의 이야기다. 자바에서 리스트를 기본타입으로 제공하지 않기 때문에 **어쩔 수 없이 배열을 사용해야 할 상황이 생기고**, 그렇기 때문에 이에 대한 대처법을 소개하고자 이런 제네릭 타입을 예시로 들었다.

<br>

다음은 `ITEM7` 에서 구현했던 Stack 클래스에 제네릭을 적용해보았다.

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        /* 오류 발생 */ elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }
    ...
}
```

E는 실체화 불가 타입(non-reifiable type: 컴파일 타임에 타입 정보가 사라지는 것) 이므로 배열을 만들 수 없기 때문에 `new E[DEFAULT_INITIAL_CAPACITY]` 에서 **컴파일 에러가 발생한다**.

<br>

이에 대해 다음과 같이 두 가지 방법이 있다.

1. 내부 배열로 E[]를 쓰고 <ins>생성자에서 캐스팅</ins>
2. 내부 배열로 Object[]를 쓰고 <ins>메서드에서 캐스팅</ins>

<br>

**[내부 배열을 E[]를 쓰고 생성자에서 캐스팅]**

```java
public class Stack<E> {
    private E[] elements;
    ...

    // The elements array will contain only E instances from push(E).
    // This is sufficient to ensure type safety, but the runtime
    // type of the array won't be E[]; it will always be Object[]!
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public E pop() {
        ...
    }
    ...
}
```

<br>

**[내부 배열을 Object[]를 쓰고 메서드에서 캐스팅]**

```java
public class Stack<E> {
    private Object[] elements;
    ...
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    // Appropriate suppression of unchecked warning
    public E pop() {
        if (size == 0)
            throw new EmptyStackException();

        // push requires elements to be of type E, so cast is correct
        @SuppressWarnings("unchecked")
        E result = (E) elements[--size];

        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
    ...
}
```

현업에서는 **주로 첫 번째 방식을 선호**한다. 하지만 이 방법은 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염(heap pollution; 아이템 32)을 일으킬 수 있다. 힙 오염이 맘에 걸리는 프로그래머는 두 번째 방식을 고수하기도 한다.

추가로 제네릭 사용 시 `extends`, `super` 예약어를 사용해 파라미터에 제약을 둘 수도 있다.  
자세한 내용은 [여기](https://stackoverflow.com/questions/4343202/difference-between-super-t-and-extends-t-in-java)를 참고하면 된다.

<br>

### 결론

여러 타입을 커버하는 클래스를 만들고 싶다면 **파라미터 다형성을 이용해서 구현하기 보다는 무조건 제네릭 타입으로** 만들자. 그렇게 만든 클래스가 더 안전하고 쓰기 편하다.

또한 기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경해주자. 기존 코드에도 영향을 주지 않는다.

<br>

### 기타

- ArrayList 자체도 사실 배열을 사용해 구현되어있다.  
  (자바가 리스트를 기본타입으로 제공하지 않기 때문)
- HashMap은 성능을 좋게하기 위해 배열을 사용한다.

- 제네릭 문자는 일반적으로 다음과 같이 사용한다.  
  E: Element / T: Class Type / K: Key / V: Value

