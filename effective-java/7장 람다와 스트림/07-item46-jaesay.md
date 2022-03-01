# 스트림에서는 부작용 없는 함수를 사용하라

- 스트림은 함수형 프로그래밍에 기초한 패러다임이다.
- 각 변환(tansformation) 단계는 가능한 한 이전 단계의 결과를 받아서 처리하는 순수 함수여야 한다.
    - 순수함수(Pure Function)
        - 오직 입력만이 결과에 영향을 주는 함수
        - 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다. (side effect X)

⇒ 스트림이 제공하는 표현력, 속도, (상황에 따라서는) 병렬성을 얻을 수 있다.

```java
String source = "Abc,abc,def,def,def,ghi,jkl";

// Stream을 잘못 사용한 예
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(source).useDelimiter(",").tokens()) {
    words.forEach(word -> freq.merge(word.toLowerCase(), 1L, Long::sum));
}

// Stream을 잘 사용한 예
Map<String, Long> freq2;
try (Stream<String> words = new Scanner(source).useDelimiter(",").tokens()) {
    freq2 = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

> Stream의 `forEach` 연산은 종단 연산 중 기능이 가장 적고 가장 ‘덜' 스트림답다. 대놓고 반복적이라서 병렬화할 수도 없다. forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자. 물론 가끔은 스트림 계산 결과를 기존 컬렉션에 추가하는 등의 다른 용도로도 쓸 수 있다.
>

## java.util.stream.Collectors 클래스

- [https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html)
- 39개 메서드를 가지고 있다. (java 8 기준)
  - java 10에서는 toMap(), toList(), toSet()의 변형인 toUnmodifiableX가 4개 추가되었다.
- 복잡한 세부 내용을 잘 몰라도 이 API의 장점을 대부분 활용할 수 있다.
- 축소(reduction) 전략을 캡슐화한 블랙박스 객체라고 생각하면 좋다.
    - 여기서 축소는 스트림의 원소들을 객체 하나에 취합한다는 뜻이다.
- 수집기가 생성하는 객체는 일반적으로 컬렉션이며, 그래서 collector라는 이름을 쓴다.

### Collector Factory 예

1. toList  
   원소들을 `List`으로 모으는 수집기를 반환한다.
    ```java
    List<String> topTen = freq2.keySet().stream()
            .sorted(comparing(freq2::get).reversed())
            .limit(10)
            .collect(toList());
    ```

2. toMap

    ```java
    // 인수 2개 받는 toMap 예
    private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(toMap(Object::toString, e -> e));
    
    // 인수 3개 받는 toMap 예
    // 인수 3개를 받는 toMap은 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들때 유리하다.
    // 앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 응악가의 베스트 앨범을 짝지은 것이다.
    Map<Artist, Album> topHits = albums.stream().collect(toMap(Album::getArtist, album -> album, BinaryOperator.maxBy(comparing(Album::getSales))));
    
    // 충돌이 나면 마지막 값을 취하는(last-write-wins) 수집기를 만들 때도 유용하다.
    Map<Artist, Album> lastAlbums = albums.stream().collect(toMap(Album::getArtist, album -> album, (oldAlbum, newAlbum) -> newAlbum));
    
    // 인수 4개 받는 toMap 예
    TreeMap<String, Book> bookTreeMap = books.stream().collect(toMap(Book::getName, book -> book, (o1, o2) -> o1, TreeMap::new));
    ```

   - 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합하다.
   - 스트림의 원소 다수가 같은 키를 사용한다면 파이프라인이 `IllegalStateException`을 던지며 종료될 것이다.
   - `toMap`이나 `groupingBy`는 이런 충돌을 다루는 다양한 전략을 제공한다.
       - 키매퍼와 값맵퍼
       - 병합(merge) 함수까지 제공할 수 있다. (`BinaryOperator<U>` 형태)
   - `toMap`은 네 번째 인수로 맵팩터리를 받아 `EnumMap`이나 `TreeMap`처럼 원하는 특정 맵구현체를 직접 지정할 수 있다.
   - `toConcurrentMap`은 병렬 실행된 후 결과로 `ConcurrentMap` 인스턴스를 생성한다.

3. toSet  
원소들을 `Set`으로 모으는 수집기를 반환한다.
    ```java
    Set<String> toSet = Stream.of(source.split(",")).map(String::toUpperCase).collect(toSet());
    ```

4. groupingBy  
입력으로 분류 함수(classifier)를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다. 세가지에 대응하는 `groupingByConcurrent` 메서드도 제공된다.
    ```java
    // 인수 1개 받는 groupingBy 예
    Map<String, List<String>> groupingBy = Stream.of(source.split(",")).collect(groupingBy(word -> alphabetize(word)));
    
    // 인수 2개 받는 groupingBy 예
    words.collect(groupingBy(word -> alphabetize(word), toSet()))
                        .values().stream()
                        .filter(group -> group.size() >= minGroupSize)
                        .forEach(group -> System.out.println(group.size() + ": " + group));
    
    freq2 = words.collect(groupingBy(String::toLowerCase, counting()));
    
    // 인수 3개를 받는 groupingBy 예
    Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = garden.stream()
                    .collect(groupingBy(plant -> plant.lifeCycle, () -> new EnumMap<>(Plant.LifeCycle.class), toSet()));
    ```

5. joining  
원소들을 연결(concatenate)하는 수집기를 반환한다.
    ```java
    String joining1 = Stream.of(source.split(",")).collect(joining(","));
    String joining2 = Stream.of(source.split(",")).collect(joining(",", "[", "]"));
    ```

## 핵심정리

- 스트림 연산에 건네지는 모든 함수 객체는 부작용이 없어야 한다.
- 스트림의 종단 연산 `forEach`는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다.
- 스트림을 올바로 사용하려면 수집기를 잘 알아둬야 한다.