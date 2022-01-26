## 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라
여기서 말하는 상속의 위험성은 *외부* 클래스 상속을 말한다. 여기서 *외부* 란 프로그래머의 통제권 밖에 있어서 언제 어떻게 변경될지 모른다는 뜻이다.

### 상속용 클래스 만드는 방법
- 클래스 내부의 자기사용패턴을 모두 문서화한다.
```java
public abstract class AbstractCollection<E> implements Collection<E> {
    /**
     * {@inheritDoc}
     *
     * @implSpec
     * This implementation iterates over the collection looking for the
     * specified element.  If it finds the element, it removes the element
     * from the collection using the iterator's remove method.
     *
     * <p>Note that this implementation throws an
     * {@code UnsupportedOperationException} if the iterator returned by this
     * collection's iterator method does not implement the {@code remove}
     * method and this collection contains the specified object.
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     */
    public boolean remove(Object o) {
```
- 효율 좋은 하위 클래스를 만들 수 있게 하는 메소드가 있다면 protected로 제공한다.
```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    /**
	...
	* @implSpec
     * This implementation gets a list iterator positioned before
     * {@code fromIndex}, and repeatedly calls {@code ListIterator.next}
     * followed by {@code ListIterator.remove} until the entire range has
     * been removed.  <b>Note: if {@code ListIterator.remove} requires linear
     * time, this implementation requires quadratic time.</b>
     *
     * @param fromIndex index of first element to be removed
     * @param toIndex index after last element to be removed
     */
    protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
    }
```

- 상속용 클래스의 생성자는 재정의 가능 메소드를 호출하면 안된다.
```java
public class Super {
	public Super() {
		overrideMe(); // 하위 클래서에서 재정의한 메소드가 하위클래스의 생성자보다 먼저 호출 -> 오작동
	}
	public void overrideMe() {}
}

public final class Sub extends Super {
	private final Instant instant;
	Sub() {
		instant = instant.now();
	}
	@Override public overrideMe() {
		System.out.println(instant);
	}
}
```

- Cloneable과 Serailizable 인터페이스를 구현한 클래스를 상속할 수 있게 설계하는 것은 권장되지 않는다. (클래스를 확장하려는 프로그래머에게 엄청난 부담을 준다.)

### 일반적인 구체 클래스에서는?
- 상속을 금지하는 법이 있다. 하지만 일반적으로 계측, 통지, 동기화 기능 제약들을 추가해왔을 테니 다소 논란의 여지가 있다.
- 핵심기능을 정의한 인터페이스가 만들고 그 인터페이스를 구현하고 상속을 금지한다 (ex. Set, List, Map)
- 상속 대신 래퍼 클래스 패턴을 사용한다. (Item 18)
- 꼭 상속을 허용해야겠다면 재정의 가능 메소드를 호출하는 자기 사용 코드를 완벽히 제거하고 이 사실을 문서로 남겨야 한다.

```java
public class Super {
	public void 재정의가능메소드를_호출하는_재정의가능메소드() {
		재정의가능메소드코드의_도우미메소드(); // 재정의가능메소드 대신 도우미 메소드 호출
	}
	public void 재정의가능메소드() {
		재정의가능메소드코드의_도우미메소드();
	}
	private void 재정의가능메소드코드의_도우미메소드() {}
}
```

### 핵심정리
- 상속용 클래스를 만들어야 한다면 상속을 고려한 설계와 문서화를 해야 한다.
- 상속용 클래스를 배포 전 반드시 검증해야 한다. (배포 후에는 성능과 기능 영원히 영향을 미친다.)
- 클래스를 확장해야할 명확한 이유가 떠오르지 않는다면 상속을 금지해야 한다.