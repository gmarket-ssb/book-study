## [item 45] 스트림은 주의해서 사용하라

스트림 API는 다량의 데이터 처리 작업 (순차/병렬)을 돕고자 JAVA 8에 추가되었다.

* 스트림 - 데이터 원소의 유한/무한 시퀀스
* 스트림 파이프라인 - 스트림 원소들로 수행하는 연산 단계

컬렉션, 배열, 파일, 정규표현식 패턴 매치, 난수 생성기, 다른 스트림<br>
스트림 안의 데이터 원소들은 객체 참조 or 기본 타입(int, long, double을 지원)<br>
--> ex) char 값들을 처리 할 때는 스트림을 삼가는 편이 낫다.

---
---
### 스트림 파이프라인

- 스트림 API는 메서드 연쇄를 지원하는 플루언트 API<br>
- 파이프라인을 구성하는 모든 호출을 연결해서 단 하나의 표현식으로 완성가능<br>
- 스트림 파이프라인은 순차적을 수행 (병렬적으로 실행하려면 parallel메서드 호출하면되나 효과를 볼 수 있는 상황이 많지 않다 item48)

* 순서 : **소스 스트림 -> 중간 연산 -> 종단 연산**<br>
[지연 평가] 종단 연산이 호출될 때 이뤄진다. (중간연산을 합친 다음에 합쳐진 중간연산을 최종 연산으로 한번에 처리)

* 스트림 API는 어떠한 계산이라도 할 수 있지만, 잘못 사용하면 가독성이 떨어지고 유지보수도 힘들어진다. 언제 써야하는지 확고부동한 규칙은 없지만 참고할 만한 노하우는 있다.

> 아나그램(anagram) : 철자를 구성하는 알파벳이 같고 순서만 다른 단어
```java
public class IterativeAnagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word),
                        (unused) -> new TreeSet<>()).add(word);
            }
        }

        for (Set<String> group : groups.values()) {
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

```java
public class StreamAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```
> 스트림을 사용했지만 가독성이 떨어지고, 한 번에 이해하기가 힘들다. (과하게 사용된 경우)<br>
> 스트림을 과용하면 프로그램이 읽거나 유지보수하기가 어려워진다.

```java
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(group -> System.out.println(group.size() + ": " + group));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
스트림을 적절하게 사용하면 깔끔하고 명료해진다. (원래 코드보다 짧으면서도 명확하기까지 하다.)

> 람다에서는 타입 이름을 자주 생략하므로 "words"처럼 매개변수 이름을 잘 지어야 가독성이 유지된다.<br>
> alphabetize 메서드처럼 도우미 메서드를 적절하게 사용하는 일은 일반 반복 코드보다 스트림 파이프라인에서 훨씬 더 도움이 된다.

* 기존 코드는 스트림을 사용하도록 리팩토링하되, 새 코드가 더 나아 보일 때만 반영하자. <br>스트림과 반복문을 적절히 조합하는 것이 최선이다.

---
---

### 스트림을 적용하기 좋은 경우

스트림 파이프라인은 되풀이되는 계산을 함수 객체(람다, 메서드 참조)로 표현
반복 코드에서는 주로 코드 블록을 사용해 표현한다.<br>
그런데 함수 객체로는 할 수 없지만, 코드 블록으로는 할 수 있는 일들이 있다.
1. 코드 블록안에서는 범위 안의 지역변수를 읽고 수정가능, 하지만 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고 지역변수 수정은 불가능
2. 코드 블록에서는 람다와는 달리 return을 쓰거나 break, continue로 반복을 멈추거나 건너뛸 수 있고 메서드 선언에 명시된 검사 예외를 던질 수 있다.

스트림을 적용하기 좋은 경우
>* 원소들의 시퀀스를 일관되게 변환한다.
>* 원소들의 시퀀스를 필터링한다.
>* 원소들의 시퀀스를 하나의 연산을 사용해 결합한다(더하기, 연결하기, 최솟값 구하기 등)
>* 원소들의 시퀀스를 컬렉션에 모은다(아마도 공통된 속성을 기준으로 묶어가며)
>* 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

---
---

## ▶ 정리

```java
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for (Suit suit : Suit.values())
        for (Rank rank : Rank.values())
            result.add(new Card(suit, rank));
    return result;
}

private static List<Card> newDeck() {
    return Stream.of(Suit.values())
            .flatMap(suit ->
                    Stream.of(Rank.values())

                            .map(rank -> new Card(suit, rank)))
            .collect(toList());
}
```
카드는 숫자(Rank)와 무늬(Suit)를 묶은 값 클래스이고, 이를 초기화하는 작업이 위와 같다.<br>
위처럼 큰 차이가 없을 때, 취향에 따라 스트림이 더 익숙해서 더 편한 경우가 있고 일반적인 반복 방식이 더 편한 경우가 있다. <br>
이럴 때는 팀원들과 커뮤니케이션 후 더 괜찮은 방식으로 사용하자 (동료들도 스트림 코드를 편하게 이해할 수 있다면 스트림 방식을 이용하면 좋다)

    스트림과 반복 중 어느쪽이 더 나은지 확신이 어렵다면, 둘 다 해보고 더 나은 쪽을 택하는게 좋다.
