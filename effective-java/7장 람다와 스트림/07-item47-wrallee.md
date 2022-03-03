## 반환 타입으로는 스트림보다 컬렉션이 낫다

원소 시퀀스를 반환 할 때는 웬만하면 컬렉션으로 반환하길 권장한다.  
컬렉션을 반환함으로써 스트림 혹은 반복자를 사용하는 사용자를 모두 만족시킬 수 있다.

일반적으로 ArrayList와 같은 표준 컬렉션을 활용하지만, 경우에 따라 아래와 같은 커스텀 컬렉션을 구현할 수 있다.  
아래 예시는 부분집합을 구성하는 컬렉션을 생성하는 클래스인데, 객체 중복생성을 방지하여 성능을 확보하였다.

```java
public class PowerSet {
    // Returns the power set of an input set as custom collection (Page 218)
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException("Set too big " + s);
        return new AbstractList<Set<E>>() {
            @Override public int size() {
                return 1 << src.size(); // 2 to the power srcSize
            }

            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }

            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }

    public static void main(String[] args) {
        Set s = new HashSet(Arrays.asList(args));
        Collection powerSet = PowerSet.of(s);
        System.out.println(powerSet);
    }
}

>>>>> java PowerSet A B C
>>>>> [[], [A], [B], [A, B], [C], [A, C], [B, C], [A, B, C]]
```

위의 경우 인스턴스를 만들어내기 때문에 기본적으로 메모리를 많이 차지한다.  
따라서 스트림으로 반환하는것도 하나의 선택지가 될 수 있으며 그 구현은 다음과 같다.

```java
// Two ways to generate a stream of all the sublists of a list (Pages 219-20)
public class SubLists {
    // Returns a stream of all the sublists of its input list (Page 219)
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()),
            prefixes(list).flatMap(SubLists::suffixes));
    }

    private static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
            .mapToObj(end -> list.subList(0, end));
    }

    private static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.range(0, list.size())
            .mapToObj(start -> list.subList(start, list.size()));
    }

    public static void main(String[] args) {
        List<String> list = Arrays.asList(args);
        Stream<List<String>> subListStream = SubLists.of(list);
        subListStream.forEach(System.out::println);
    }
}

>>>>> java SubLists a b c
>>>>> [][a][a, b][b][a, b, c][b, c][c]
```

<br>

만약 반복자만, 스트림만을 반환하는 API의 경우 사용자는 다음과 같이 어댑터를 구현하여 변환할 수 있다.

```java
// Adapters from stream to iterable and vice-versa (Page 216)
public class Adapters {
    // Adapter from  Stream<E> to Iterable<E> (
    public static <E> Iterable<E> iterableOf(Stream<E> stream) {
        return stream::iterator;
    }

    // Adapter from Iterable<E> to Stream<E>
    public static <E> Stream<E> streamOf(Iterable<E> iterable) {
        return StreamSupport.stream(iterable.spliterator(), false);
    }
}
```

<br>

### 결론

1. 스트림으로 처리하려는 사용자와 반복으로 처리 하려는 사용자를 모두 고려하여 공개 API를 설계해야한다.
2. 따라서 Stream과 Iterable을 모두 지원할 수 있는 **Collection 타입으로 반환**하는것이 좋다. <ins>하지만 3번에 유의하자!</ins>
3. 일반적으로 ArrayList같은 기본 컬렉션으로 반환하되, 컬렉션 **원소의 수가 많다면 메모리 초과에 대해 고민해야 한다.**
3. 이 경우 전용 컬렉션 구현을 고려하거나, Stream으로 반환하도록 구현한다. 전용 컬렉션 구현 시 contains와 size를 구현한다.
3. contains와 size를 구현하는게 불가능 할 경우(반복이 시작되기 전 시퀀스가 확정되지 않았을 경우) 반복자를 반환하는게 낫다.
4. Stream과 Iterable로 반환해도 된다. 하지만 이렇게 한다면 각각 Iterable, Stream으로 쓰기를 원하는 사용자는 어댑터 메서드를 구현해야 한다. 어댑터는 클라이언트 코드를 어수선하게 만들고, 성능상으로도 좋지 않으니 위의 1~4 정도에서 끝내도록 노력하자.