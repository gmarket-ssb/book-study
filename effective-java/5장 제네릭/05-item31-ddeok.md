## [item 31] 한정적 와일드카드를 사용해 API 유연성을 높여라

<br>




```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    } 

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size==0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```


```java
// 코드 31-1 와일드카드 타입을 사용하지 않은 pushAll 메서드 - 결함이 있다!
public void pushAll(Iterable<E> src) {
    for (E e : src)
    push(e);
}
```

```java
 // 코드 31-2 E 생산자(producer) 매개변수에 와일드카드 타입 적용
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
    push(e);
}
```


```java
// 코드 31-3 와일드카드 타입을 사용하지 않은 popAll 메서드 - 결함이 있다!
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

```java
// 코드 31-4 E 소비자(consumer) 매개변수에 와일드카드 타입 적용
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

```java
// T 생산자 매개변수에 와일드카드 타입 적용 (184쪽)
public class Chooser<T> {
private final List<T> choiceList;
private final Random rnd = new Random();

    // 코드 31-5 T 생산자 매개변수에 와일드카드 타입 적용 (184쪽)
    public Chooser(Collection<? extends T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }

    public static void main(String[] args) {
        List<Integer> intList = List.of(1, 2, 3, 4, 5, 6);
        Chooser<Number> chooser = new Chooser<>(intList);
        for (int i = 0; i < 10; i++) {
            Number choice = chooser.choose();
            System.out.println(choice);
        }
    }
}
```

```java
package effectivejava.chapter5.item31;
import java.util.*;

// 코드 30-2의 제네릭 union 메서드에 와일드카드 타입을 적용해 유연성을 높였다. (185-186쪽)
public class Union {
    public static <E> Set<E> union(Set<? extends E> s1,
                                   Set<? extends E> s2) {
        Set<E> result = new HashSet<E>(s1);
        result.addAll(s2);
        return result;
    }

    // 향상된 유연성을 확인해주는 맛보기 프로그램 (185쪽)
    public static void main(String[] args) {
        Set<Integer> integers = new HashSet<>();
        integers.add(1); 
        integers.add(3); 
        integers.add(5); 

        Set<Double> doubles =  new HashSet<>();
        doubles.add(2.0); 
        doubles.add(4.0); 
        doubles.add(6.0); 

        Set<Number> numbers = union(integers, doubles);

//      // 코드 31-6 자바 7까지는 명시적 타입 인수를 사용해야 한다. (186쪽)
//      Set<Number> numbers = Union.<Number>union(integers, doubles);

        System.out.println(numbers);
    }
}
```

```java
// 와일드카드 타입을 사용해 재귀적 타입 한정을 다듬었다. (187쪽)
public class RecursiveTypeBound {
    public static <E extends Comparable<? super E>> E max(
        List<? extends E> list) {
        if (list.isEmpty())
            throw new IllegalArgumentException("빈 리스트");
    
        E result = null;
        for (E e : list)
            if (result == null || e.compareTo(result) > 0)
                result = e;
    
            return result;
        }

    public static void main(String[] args) {
        List<String> argList = Arrays.asList(args);
        System.out.println(max(argList));
    }
}
```

```java
// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
public class Swap {
    public static void swap(List<?> list, int i, int j) {
        swapHelper(list, i, j);
    }

    // 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
    private static <E> void swapHelper(List<E> list, int i, int j) {
        list.set(i, list.set(j, list.get(i)));
    }

    public static void main(String[] args) {
        // 첫 번째와 마지막 인수를 스왑한 후 결과 리스트를 출력한다.
        List<String> argList = Arrays.asList(args);
        swap(argList, 0, argList.size() - 1);
        System.out.println(argList);
    }
}
```


### 핵심정리
- 와일드카드 타입을 적용하면 API가 유연해진다
- PECS 공식을 기억하자
- producer는 extends를 consumer는 super를 사용한다
- Comparable과 Comparator 모두 소비자라는 사실을 잊지 말자
