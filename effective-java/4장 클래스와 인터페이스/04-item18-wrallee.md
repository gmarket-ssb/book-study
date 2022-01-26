## 상속보다는 컴포지션을 사용하라

*이 장에서의 '상속'은 **다른 클래스에 대한 extends**를 말한다. <ins>인터페이스를 implements, extends 하는것과는 무관</ins>하다.*

상속은 상위 클래스의 구현에 의존적이다.  
예를 들어 더해진 원소 수를 저장하는 InstrumentedHashSet이란 클래스를 아래와 같이 구현했다고 하자.

### 상속을 잘못 사용한 예

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;
    
    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
}
```

위 클래스에서 `addAll` 메서드를 호출하면 `super.addAll(c)` 실행 시 내부적으로 반복문을 돌며 `add`를 각각 호출하기 때문에 이미 더해진 `addCount`가 `add`에서 다시 한 번 더해지게 된다.

이와 같이 상위 클래스의 내부 구현을 알아야 적절하게 구현할 수 있다. 하지만 만약 상위 클래스의 구현이 `add`를 사용하지 않는 방식으로 변경된다면 이 프로그램은 다시 오동작 하게 된다. 이처럼 상위 클래스에 의존적이라면 상위 클래스의 구현이 변경 될 때 그 영향이 하위 클래스로 전파되게 된다.  

<br>

### 컴포지션을 사용한 예

 컴포지션은 확장하고자 하는 클래스를 private 필드로 참조하는 방식으로, **<ins>기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻</ins>**에서 컴포지션이라 한다. 컴포지션은 **has-a**관계, 상속은 **is-a**관계일 때 사용한다고 한다.

아래 클래스의 메서드들은 기존 클래스의 대응하는 메서드를 호출해 결과를 반환한다. 이 클래스를 전달 클래스라 하며, 메서드들은 전달 메서드(forwarding method)라 부른다.

**[전달 클래스]**

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> set;
    
    public ForwardingSet(Set<E> set) {
        this.set = set;
    }
    
    public void clear() { set.clear(); }
    public boolean contains(Object o) { return set.contains(o); }
    public boolean isEmpty() { return set.isEmpty(); }
    ...
    public <T> T[] toArray(T[] a) { return set.toArray(a); }
    @Override public boolean equals(Object o) { return set.equals(o); }
    @Override public int hashCode() { return set.hashCode(); }
    @Override public String toString() { return set.toString(); }
}
```

전달(forwarding) 방식으로 구현된 클래스는 기존 클래스의 내부 구현의 영향에서 벗어난다.  
또한 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향을 받지 않는다.

이렇게 인터페이스에 대한 전달 클래스를 만들어두면 이를 재사용해가며 여러 종류의 래퍼 클래스를 손쉽게 구현할 수 있다. 구글 Guava 라이브러리는 모든 컬렉션 인터페이스용 전달 메서드를 전부 구현해뒀다.

<br>

**[래퍼 클래스]**

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    
    public InstrumentedSet(Set<E> set) {
        super(set);
    }
    
    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }
}

>>>>> new InstrumentedSet<>(new TreeSet<>());
>>>>> new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

위의 전달 클래스를 활용하여 추가 된 원소의 갯수 정보를 갖는 클래스를 구현했다. 위 InstrumentedSet과 같이 다른 인스턴스를 감싸고 있는 것을 **래퍼 클래스**라고 한다. 또한 이처럼 새로운 기능을 덧씌워 기존 클래스를 꾸며주는 것을 [데코레이터 패턴](https://en.wikipedia.org/wiki/Decorator_pattern)이라 한다.

여기서 전달 클래스는 재사용성을 위해 별도로 구현하는것인데, 재사용이 필요 없다면 InstrumentedSet이 Set을 직접 implements 해도 된다. 하지만 만약 LimitedSet이라는 클래스를 새로 만들어야 되는 상황이 온다면 clear, isEmpty와 같은 메서드를 다시 각각 구현해 주어야 한다.

<br>

### 결론

**상속을 사용하기 전에는 반드시 다음 내용을 고민해보자**

- 확장하려는 클래스의 API에 아무런 결함이 없는가?
- 결함이 있다면, 이 결함이 여러분 클래스의 API까지 전파되어도 괜찮은가?
- 구현하려는 클래스와 상속하려는 클래스가 **is-a** 관계인가?

<br>

**기타**

- 자바의 Stack, Properties 클래스는 상속이 아닌 컴포지션으로 구현하는것이 더 좋았을것이다.
- [콜백 프레임워크에선 내부 클래스가 래퍼클래스를 무시하고 동작할 수 있다.](https://stackoverflow.com/questions/28254116/wrapper-classes-are-not-suited-for-callback-frameworks)

<br>

참고: https://umbum.dev/822

