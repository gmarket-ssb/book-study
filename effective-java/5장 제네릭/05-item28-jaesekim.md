## 배열보다는 리스트를 사용하라.

### 배열과 제네릭 타입의 차이

1. 배열은 공변(covariant) 이고 제네릭은 불공변(invariant) 불공변이다.
    ```java
    // 배열은 covariant
    Super[] supers = new Sub[] {};
    
    // 제네릭은 invariant
    List<Super> superList = new ArrayList<Sub>();
    ```

    공변일 경우의 문제는?
    ```java
    // 런타임에 실패한다.
    Object[] objectArray = new Long[1];
    objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException을 던진다.
    
    // 컴파일되지 않는다.
    List<Object> objectList = new ArrayList<Long>(); // 호환되지 않는 타입이다.
    ```

1. 배열은 실체화(reify)된다.
- 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인
- 제네릭은 원소 타입을 컴파일타임에만 검사하고 런타임에는 소거

⇒ 위 두가지 이유로 제네릭 배열을 만들지 못하게 막았다.

### 둘이 섞어 쓰다가 컴파일 오류나 경고를 만나면, 가장 먼저 배열을 리스트로 대체하는 방법을 적용해보자.

```java
class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() { // method 사용한 코드에서 형변환을 해줘야 하며 런타임 시 형변환에러가 발생할 수 있다.
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}

class Chooser2<T> {
    private final T[] choiceArray;

	// 소거되어 무슨 타입인지 알 수 없기 떄문에 경고 발생
	// 애초에 경고를 제거하는 편이 더 낫다.
    public Chooser2(Collection<T> choices) {
        choiceArray = (T[]) choices.toArray(); // unchecked warnings
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}

// 코드양이 조금 늘었고 아마도 조금 더 느릴테지만, 런타임에 ClassCastException을 만날 일은 없으니 그만한 가치가 있다.
class Chooser3<T> {
    private final List<T> choiceList;

    public Chooser3(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

### 핵심 정리
제네릭 타입이 필요할 경우 섞어 쓰지말고 가장 먼저 배열을 리스트로 대체하는 방법을 적용해보자