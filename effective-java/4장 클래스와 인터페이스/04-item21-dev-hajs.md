## 인터페이스는 구현하는 쪽을 생각해 설계하라

인터페이스에서 default method 를 선언하면, default method 를 재정의하지 않은 모든 클래스에서 디폴트 구현이 쓰이게 된다.
결국, default method 는 구현 클래스에 대해 아무것도 모른채 합의 없이 무작정 '삽입' 된다고 볼 수 있다.
<br><br>

### 그저 '삽입' 된 default method 의 대표적인 예
> 자바 8의 `Collection` 인터페이스에 추가된 `removeIf` 메서드
```java
/**
 * Removes all of the elements of this collection that satisfy the given
 * predicate.  Errors or runtime exceptions thrown during iteration or by
 * the predicate are relayed to the caller.
 * ...
 * @since 1.8
 */
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
        if (filter.test(each.next())) {
            each.remove();
            removed = true;
        }
    }
    return removed;
}
```
이 메서드는 true 를 반환하는 모든 원소를 제거한다.<br>

이 메서드가 *'모든 메서드에서 주어진 락 객체로 동기화하는 래퍼 클래스'* 인<br>
`apache.commons.collections4.collection.SynchronizedCollection` 클래스와 함께 사용될 때,<br>
`removeIf` 메서드를 재정의하지 않았을 경우에 **모든 메서드 호출을 알아서 동기화해주지 못하게 되는 상황이 발생**하게 된다.
<br><br>

### 결론
* 기존 인터페이스에 default method 로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야 한다.
* default method 라는 도구가 생겼더라도 **인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야 한다.**
* 인터페이스는 구현하는 쪽을 생각해 설계하라 (아이템 주제)
