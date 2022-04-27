## 커스텀 직렬화 형태를 고려해보라

기본 직렬화 형태는 항상 먼저 고민해보고 괜찮다고 판단될 때만 사용하도록 하자.  
객체의 <ins>물리적 표현과 논리적 내용이 같다면</ins> 기본 직렬화 형태를 사용해도 무방하다.

<br/>

[기본 직렬화 형태가 적합한 클래스]

```java
public class Name implements Serializable {
    /**
     * 성. null이 아니어야 함.
     * @serial
     */
    private final String lastName;
    
    /**
     * 이름. null이 아니어야 함.
     * @serial
     */
    private final String firstName;
    
    /**
     * 중간이름. 중간이름이 없다면 null.
     * @serial
     */
    private final String middleName;
    ...
}
```

<br/>

[기본 직렬화가 적합하지 않은 클래스]

```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;
    
    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    ...
}
```

<br/>

위 클래스는 논리적으로 일련의 문자열을 표현하고, 물리적으로는 양방향 연결 리스트로 구현 되어있다. 이 클래스에 기본 직렬화 형태를 사용하면 각 노드의 양방향 연결 정보를 포함해 모든 엔트리를 철두철미하게 기록한다.

객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하면 크게 <ins>네 가지 면에서 문제가 생긴다</ins>.

1. **공개 API가 현재의 내부 표현 방식에 영구히 묶인다.**  
   Entry 값도 직렬화 요소가 되며, 따라서 공개 API가 된다. 그러므로 내부 표현 방식을 바꾸더라도 StringList는 여전히 연결 리스트로 표현된 입력도 처리할 수 있어야 한다.
2. **너무 많은 공간을 차지할 수 있다.**  
   엔트리와 연결 정보는 내부 구현에 해당하기 때문에 직렬화 형태에 포함할 가치가 없다. 하지만 포함돼서 디스크 용량을 차지하거나 네트워크 전송 속도가 느려질 수 있다.
3. **시간이 너무 많이 걸릴 수 있다.**  
   앞의 예시에서 직렬화 로직은 객체 그래프의 위상에 관한 정보가 없으니 그래프를 전부 순회하게 된다.
4. **스택 오버플로를 일으킬 수 있다.**  
   기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 원소의 수에 따라, 환경에 따라 스택 오버플로를 발생시킬수 있다.

<br/>

StringList를 합리적으로 직렬화 하는 방식은 다음과 같다.

```java
public final class StringList implements Serializable {
    private transient int size   = 0;
    private transient Entry head = null;

    // No longer Serializable!
    private static class Entry {
        String data;
        Entry  next;
        Entry  previous;
    }

    // Appends the specified string to the list
    public final void add(String s) {  }

    /**
     * Serialize this {@code StringList} instance.
     *
     * @serialData The size of the list (the number of strings
     * it contains) is emitted ({@code int}), followed by all of
     * its elements (each a {@code String}), in the proper
     * sequence.
     */
    private void writeObject(ObjectOutputStream s)
            throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }

    private void readObject(ObjectInputStream s)
            throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();

        // Read in all elements and insert them in list
        for (int i = 0; i < numElements; i++)
            add((String) s.readObject());
    }

    // Remainder omitted
}
```

transient 한정자는 해당 인스턴스 필드가 기본 직렬화 형태에 포함되지 않는다는 표시다. read/writeObject 메서드 첫줄에서 기본 직렬화를 호출하는데, 추후 transient가 아닌 필드가 추가되더라도 상호 호환되도록 하기 위함이다. 이렇게 추가하고 신버전 인스턴스를 구버전으로 역직렬화 하면 새로 추가 된 필드들은 무시되어 정상적으로 변환될 것이다. 하지만 만약 구버전 readObject 메서드에서 defaultReadObject를 호출하지 않는다면 StreamCorruptedException이 발생한다.

<br/>

또한 불변식이 세부 구현에 따라 달라지는 객체에서 기본 직렬화를 사용하는것은 위험하다. 예를 들어 해시테이블을 구현할 때 기본 직렬화를 사용한다면, 해시함수가 구현에 따라 달라질 수 있기 때문에 자칫 잘못된 객체들이 튀어나올 수도 있다.

일반적으로 transient로 선언해도 되는 인스턴스 필드에는 모두 transient를 붙여주는게 좋다. 특히 캐시된 해시값과 같은 다른 필드에서 유도되는 값이거나, JVM을 실행할때마다 달라지는 필드에 반드시 붙여줘야 한다. long과 같은 네이티브 자료구조를 가리키는(pointing) 필드가 여기 속한다. 참고로 transient 필드들은 역직렬화 될 때 기본값으로 초기화된다.

객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다. 예를 들어 모든 메서드를 synchronized로 선언하여 스레드 안전하게 만든 객체에서 기본 직렬화를 사용하려면 writeObject도 synchronized로 선언해야 한다.

<br/>

```java
private static final long serialVersionUID = <무작위로 고른 long 값>;
```

어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 serialVersionUID를 명시적으로 부여하자. 이렇게 serialVersionUID값을 직접 지정하게 되면 런타임에서 serialVersionUID를 생성하지 않아 성능이 약간 향상되기도 하고, 구버전과의 직렬화 호환성을 관리할 수도 있다.

<br/>

#### 결론

- 자바의 기본 직렬화 형태는 객체를 직렬화한 결과가 해당 객체의 논리적 표현에 부합할 때만 사용해라.
- 그렇지 않다면 객체를 적절히 설명하는 커스텀 직렬화 형태를 고안하라.
- 직렬화 형태도 공개 메서드(아이템 51)를 설계할 때에 준하는 시간을 들여 설계해야 한다. 한번 공개된 메서드는 향후 릴리스에서 제거할 수 없듯이, 직렬화 형태에 포함된 필드도 마음대로 제거할 수 없다.