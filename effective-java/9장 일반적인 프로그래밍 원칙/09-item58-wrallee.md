## 전통적인 for 문보다는 for-each 문을 사용하라

결론: 웬만하면 for-each 문(향상된 for 문)을 사용하자. for-each 문은 명료하고, 유연하고, 버그를 예방해주며 성능 저하도 없다. 가능한 모든 곳에서 for 문이 아닌 for-each 문을 사용하자. 



**전통적인 방식 - 컬렉션 순회, 배열 순회**

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // e로 무언가를 한다.
}

for (int i = 0; i < a.length; i++) {
    ... // a[i]로 무언가를 한다.
}
```

위 코드에서 우리가 실제 활용하는 인스턴스는 각각 `e`와 `a[i]` 의 참조 객체이다.  
오직 이 객체를 활용하기 위한 목적으로 `i` 라는 변수를 별도로 사용하고 있으며, 이러한 방식은 코드를 지저분하게 할 뿐만 아니라 오류를 유발할 수 있다.

실제로 for문이 중첩되어 `i`, `j`, `k`  등 활용하는 변수가 많아지면서 인덱스 변수를 잘못 쓰는 상황은 종종 벌어진다.

<br/>

다음 예시는 `iterator`를 잘못 사용하여 프로그램이 오동작 하는 사례이다.  
아래 프로그램은 `"ONE ONE"`, `"ONE TWO" `, `"ONE THREE"`, ..., `"SIX SIX"` 총 36개의 조합을 출력하려 했지만, `i.next()`가 2번째 for문 안에 있기 때문에 단 6개의 조합만 출력하고 프로그램이 종료된다.

```java
public class DiceRolls {
    enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }

    public static void main(String[] args) {
        Collection<Face> faces = EnumSet.allOf(Face.class);

        for (Iterator<Face> i = faces.iterator(); i.hasNext(); ) {
            for (Iterator<Face> j = faces.iterator(); j.hasNext(); ) {
                System.out.println(i.next() + " " + j.next());
            }
        }
    }
}
```
이를 제대로 동작하도록 아래와 같이 고칠 수 있다.
```java
for (Iterator<Face> i = faces.iterator(); i.hasNext(); ) {
    Suit suit = i.next();
    for (Iterator<Face> j = faces.iterator(); j.hasNext(); ) {
        System.out.println(suit + " " + j.next());
    }
}
```

그리고 for-each문을 사용하면 다음과 같이 간결해진다.

```java
for (Face f1 : faces) {
    for (Face f2 : faces) {
        System.out.println(f1 + " " + f2);
    }
}
```

<br/>

물론 for-each문을 사용할 수 없는 상황도 존재하는데 다음 3가지 경우 for-each문을 사용할 수 없어 일반적인 for문을 사용해야 한다.

- 파괴적인 필터링(destructive filtering): 순회하며 원소를 제거해야 하는 상황. 자바 8부터는 removeIf를 사용하여 for문을 직접 사용하는것을 피할 수 있다.
- 변형(transforming): 순회하며 원소를 변형/교체 할 경우
- 병렬 반복(parallel iteration): 여러 컬렉션을 병렬로 동시에 순회해야 할 경우

<br/>

for-each문은 컬렉션, 배열, Iterable 구현체를 순회할 수 있다. 따라서 개발자가 직접 내부 원소를 순회하는 커스텀 타입을 작성한다면 Collection 인터페이스를 구현하지 않는다 하더라도 <ins>Iterable은 구현해두는것이 좋다</ins>.
