# [item 55] 옵셔널 반환은 신중히 하라


## 메서드가 특정 조건에서 값을 반환할 수 없을 때

---

## 자바 8 이전 방법
1. 예외를 던진다 
    1) 예외는 진짜 예외 상황에서만 던져야 한다 (item 69)
    2) 예외 생성 시 stackTrace 비용이 만만치 않다.
2. 반환타입이 객체 참조라면 null을 반환
    1) null 처리 추가 코드가 필요하다. (NullPointerException 대비)

## 자바 8 이후로 생긴 Optional\<T>
* Optional\<T>를 이용하여 null이 아닌 T 타입 참조를 담거나, 아무것도 담지 않을 수 있다.
* 옵셔널은 원소를 최대 1개 가질 수 있는 불변 컬렉션이다.
* 보통은 T를 반환해야 하지만 **특정 조건에서는 아무것도 반환하지 않아야 할 때 T대신 Optional<T>를 반환하도록 선언**

---

### 컬렉션에서 최댓값을 구하는 방법
1) 컬렉션이 비어있으면 예외
```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if(c.isEmpty())
        throw new IllegalArgumentException("빈 컬렉션");

    E result = null;
    for(E e : c)
        if(result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return result;
}
```

2) 컬렉션에서 최댓값을 구해서 Optional을 반환

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if(c.isEmpty())
        return Optional.empty();

    E result = null;
    for(E e : c)
        if(result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return Optional.of(result);
}
```

    옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 말자 (옵셔널이 도입된 취지에 반하는 행위이다.)


컬렉션에서 최댓값을 구해서 Optional을 반환 (스트림 사용)
```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
   return c.stream().max(Comparator.naturalOrder()); 
} 
```

---

## 언제 Optional\<T>를 사용해야 하나
> 옵셔널은 검사 예외와 취지가 비슷하다. (item 71)

* 반환 값이 있을 수도 없을 수도 있음을 API 사용자에게 명확히 알려주는 것
* 검사 예외를 던지면 클라이언트는 이에 대처하는 코드가 필요
* 같은 맥락으로 옵셔널을 반환한다면 클라이언트는 값을 받지 못할때의 상황에 대비해야한다.

---
## Optional\<T> 활용

### (1) 기본값을 설정해놓기
```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

### (2) 원하는 예외 던지기
```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```
여기서는 실제 예외가 아닌 예외 팩토리를 던지므로 추가적인 비싼 예외 생성 비용을 막았다.

### (3) 항상 값이 있다고 가정하기 (하지만 없으면 NoSuchElementException이 발생)
```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```


더 다양한 고급 메서드들이 있으니 filter, map, flatMap, ifPresent 등 API문서를 참조해서 사용하자.<br>

### (4) isPresent 메서드
(옵셔널이 채워져 있으면 true, 비어 있으면 false 반환)

```java
Optional<ProcessHandle> parentProcess = ph.parent();

//isPresent 사용
System.out.println("부모 PID : " + (parentProcess.isPresnet() ? String.valueOf(parentProcess.get().pid()) : "N/A"));

//Optional의 map사용
System.out.println("부모 PID : " + parentProcess.map(h -> String.valueOf(h.pid())).orElse("N/A"));
```

옵셔널들을 Stream으로 받아서, 채워진 옵셔널들에서 값을 뽑아 Stream에 담아 처리하는 경우
```java
//java 8
streamOfOptionals
        .filter(Optional::isPresent)
        .map(Optional::get)

//java 9 Optional에 stream()추가
//옵셔널에 값이 있으면 그 값을 원소로 담은 스트림으로, 없다면 빈 스트림으로 변환
streamOfOptionals
        .flatMap(Optional::stream)

```

---

## Optional<T> 주의 사항

* **컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 Optional로 감싸면 안된다.**
    * 빈 컨테이너를 그대로 반환하면 클라이언트에 굳이 옵셔널 처리 코드를 넣지 않아도 된다.
* 어떤 경우에 메서드 반환 타입을 T 대신 Optional<T>로 선언해야 하나
    *  결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional<T>를 반환
* 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자.
    * 박싱된 기본 타입을 담는 옵셔널은 사실상 값을 두 겹이나 감싸기 때문에 무겁다.
    * int, long, double 전용 OptionalInt, OptionalLong, OptionalDouble 옵셔널 클래스가 있다.

* 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는게 적절한 상황은 거의 없다.

---
---
### 핵심정리
    * 값을 반환하지 못할 수도 있고, 호출할 때마다 반환값이 없을 가능성을 고려해야하는 메서드라면 옵셔널을 반환해야 할 상활일 수 있다.
    * 하지만 성능 저하가 뒤따르니, 때로는 null을 반환하거나 예외를 던지는 편이 나을 수도 있다.
    * 옵셔널 반환값 이외의 용도로 쓰는 경우는 거의 없다고 생각하자.
