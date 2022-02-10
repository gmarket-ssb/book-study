## 제네릭과 가변인수를 함께 쓸 때는 신중하라

가변인수(varargs)와 제네릭은 자바 5에서 함께 추가되었다.

문제는 가변인수와 제네릭을 혼용할 경우 힙 오염(heap pollution)을 일으킬 수 있기 때문에 혼용하게 될 경우 주의가 필요하다.

매개변수화 타입 변수가 잘못 된 오브젝트를 가리킬 때 **[힙 오염(heap pollution)](https://velog.io/@adduci/Java-힙-펄루션-Heap-pollution)**이라고 한다. 간단한 예시로 `List<String>`에 `Integer` 객체가 들어갈 경우 힙 오염이 발생했다고 볼 수 있다. 힙 오염은 레퍼런스-인스턴스 불일치로 인한 ClassCastException을 자주 유발한다.

<br>

**[Heap Pollution 발생 예시]**

```java
public class Dangerous {
    // Mixing generics and varargs can violate type safety!
    static void dangerous(List<String>... stringLists) {
        List<Integer> intList = List.of(42);
        Object[] objects = stringLists;
        objects[0] = intList; // Heap pollution
        String s = stringLists[0].get(0); // ClassCastException
    }
}
```

varargs와 generic 혼용으로 인한 heap pollution 발생. 그리고 ClassCastException.

<br>

우리가 흔히 아는 Arrays.asList(...), Collections.addAll(...), EnumSet.of(...)은 모두 이런 문제가 없는 Type-Safe한 메서드들이다. 자바 7부터 메서드에 @SafeVarargs 애너테이션이 적용하여 클라이언트측에 warning을 숨길 수 있으며, 세 메서드 모두 @SafeVarargs 애너테이션이 적용되어있다.

> <ins>메서드가 안전한 게 확실하지 않다면 절대로 @SafeVarargs 애너테이션을 달아서는 안 된다.</ins>

제네릭이나 매개변수화 타입의 varargs를 받는 모든 메서드에 @SafeVarargs를 붙여라.  
**= 이 말은 곧 안전하지 않은 varargs 메서드는 절대로 작성하지 말라는 이야기이다.**

<br>

### 안전한 varargs 메서드를 만드는 방법

**방법1:**

- varargs 배열을 수정하지 않는다.
- varargs 배열 참조를 노출하지 않는다.

배열을 수정하여 heap pollution이 발생하는 사례는 위에 소개를 했다.  
다음은 배열 참조를 노출하여 heap pollution이 발생하는 사례이다.

```java
public class PickTwo {
    // UNSAFE - Exposes a reference to its generic parameter array!
    static <T> T[] toArray(T... args) {
        return args;
    }

    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError(); // Can't get here
    }

    public static void main(String[] args) {
        String[] attributes = pickTwo("Good", "Fast", "Cheap"); // ClassCastException
        System.out.println(Arrays.toString(attributes));
    }
}
```

따라서 다음과 같이 수정해주면 안전한 varargs가 된다. 중요한점은 단순 레퍼런스를 노출하는것을 방지한다기보다는 T[], List\<T>[]와 같은 **실체화 불가 타입이 노출되는것을 막는것이 중요**하다.

```java
public class PickTwo {
    @SafeVarargs
    static <T> List<T> toList(T... args) {
        return List.of(args);
    }
    ...
}
```

<br>

**방법2:**

- varargs를 List로 대체한다.

다음과 같이 변경하면 컴파일러가 타입 안정성을 확인할 수 있기 때문에 @SafeVarargs 애너테이션을 달 필요도 없고, 안전성 여부를 개발자가 판단할 필요도 없다. 하지만 이 방식은 코드가 조금 지저분해지고 속도가 느려질 수 있다는 단점이 있다.

```java
public class FlattenWithList {
    static <T> List<T> flatten(List<List<? extends T>> lists) {
        List<T> result = new ArrayList<>();
        for (List<? extends T> list : lists)
            result.addAll(list);
        return result;
    }

    public static void main(String[] args) {
        List<Integer> flatList
            = flatten(List.of(List.of(1, 2), List.of(3, 4, 5), List.of(6,7)));
        System.out.println(flatList);
    }
}
```

<br>

### 결론

- 제네릭을 varargs 매개변수로 사용하려면 특별히 더 주의해야 한다. 
- 제네릭 varargs 매개변수를 사용하는 메서드는 타입안전성 확인 후 @SafeVarargs를 붙인다.
- varargs를 사용하는 메서드를 안전하게 작성할 수 없다면 varargs 대신 List를 활용하자.

<br>

### 기타

**공변이란?**

- 공변(Variance) : A가 B의 하위 타입일 때, T\<A> 가 T\<B>의 하위 타입이면 T가 공변의 성질을 가지고 있다고 말한다
- 반공변(Contravariance) : A가 B의 하위 타입일 떄, T\<B>가 T\<A> 의 하위 타입이면 T가 반공변의 성질을 가지고 있다고 말한다.
- 무공변(Invariance) A가 B의 하위 타입일 때, T\<A>와 T\<B>간의 아무 관계도 없다면 무공변이라고 말한다.

출처: https://wjdtn7823.tistory.com/88