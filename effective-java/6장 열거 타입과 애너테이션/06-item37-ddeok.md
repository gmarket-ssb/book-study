## [item 37] ordinal 인덱싱 대신 EnumMap 을 사용하라

<br>

배열이나 리스트에서 원소를 꺼낼 대 ordinal 메서드(아이템35)로 인덱스를 얻는 코드가 있다.  
식물을 간단히 나타낸 다음 클래스를 예로 살펴 보자.


```java
// 식물을 아주 단순하게 표현한 클래스 - 이름 / 생명주기
class Plant {
    enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

1. 식물들을 배열 하나로 관리하고, 이들을 생애주기(한해살이, 여러해살이, 두해살이)별로 묶는다.
2. 생애주기별로 총 3개의 집합을 만들고 정원을 한 바퀴 돌며 각 식물을 해당 집합에 넣는다.  

이때 어떤 프로그래머는 집합들을 배열 하나에 넣고 생애주기의 ordinal 값을 그 배열의 인덱스로 사용하려 할 것이다.

### 나쁜방법: 인덱스로 ordinal() 사용

```java
// 37-1 ordinal()을 배열 인덱스로 사용 - 따라하지 말 것!
Set<Plant>[] plantsByLifeCycle =
        (Set<Plant>[]) new Set[Plant.LifeCycle.values().length]; // set 배열 생성

for (int i=0; i<plantsByLifeCycle.length; i++)
    plantsByLifeCycle[i] = new HashSet<>(); // HashSet으로 생성

for (Plant p : garden)
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

// 결과 출력
for (int i=0; i<plantsByLifeCycle.length; i++) {
    System.out.printf("%s: %s%n", 
        Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

문제점
1. 배열은 제네릭과 호환되지 않으니 비검사 형번환을 수행해야 하고 깔끔히 컴파일되지 않을 것이다.  
2. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다. -> %s 로 스트링을 표시
3. 정확한 정숫값을 사용 한다는 것을 보증 해야함 -> 예외 발생 or 오동작 유발

즉, ordinal 을 index로 사용하지 말자


### 좋은방법: EnumMap 방식

EnumMap: 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체

```java
// 37-2 EnumMap을 사용해 데이터와 열거 타입을 매핑한다.
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
    new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc, new HashSet<>());

for (Plant p : garden)
    plantsByLifeCycle.get(p.lifeCycle).add(p);

System.out.println(plantsByLifeCycle);
```

더 짧고 명료하고 안전하고 성능도 원래 버전과 비등하다.  
EnumMap 내부에서 배열을 사용하기 때문에 안정성과 배열의 성능을 모두 얻어낸 것이다.  


### stream 사용

스트림을 사용해 맵을 관리하면 코드를 더 줄일 수 있다.  


```java
// 37-3 스트림을 사용한 코드 1 - EnumMap을 사용하지 않는다!
System.out.println(Arrays.stream(garden)
    .collect(groupingBy(p -> p.lifeCycle)));
```

문제점
1. 아무것도 명시하지 않으면 EnumMap이 아닌 고유한 맵 구현체를 사용
2. EnumMap을 써서 얻을 수 있는 공간과 성능 이점이 사라진다는 문제가 있다.



### Stream + EnumMap 사용
매개변수 3개짜리 Collectors.grouopingBy 메서드는 mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있다.

```java
// 37-4 스트림을 사용한 코드 2 - EnumMap을 이용해 데이터와 열거 타입을 매핑했다.
System.out.println(Arrays.stream(garden)
    .collect(groupingBy(p -> p.lifeCycle, 
        () -> new EnumMap<>(LifeCycle.class), toSet())));
```

- 위 코드와의 차이는 groupingBy 메소드에 원하는 맵 구현체 사용 여부

스트림을 사용하면 EnumMap만 사용했을 때와는 살짝 다르게 동작한다.  
EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다.  
예컨대 정원에 한해살이와 여러해살이 식물만 살고 두해살이는 없다면, EnumMap 버전에서는 맵을 3개 만들고 스트림 버전에서는 2개만 만든다.

ex)
```java
Plant[] garden = {
    new Plant("바질", LifeCycle.ANNUAL),
    new Plant("캐러웨이", LifeCycle.BIENNIAL),
    new Plant("딜", LifeCycle.ANNUAL),
    new Plant("라벤더", LifeCycle.PERENNIAL),
    new Plant("파슬리", LifeCycle.BIENNIAL),
    new Plant("로즈마리", LifeCycle.PERENNIAL)
};
```

EnumMap만 사용  
{ANNUAL=[바질, 딜], PERNNIAL=[], BIENNIAL=[라벤더]}

스트림 사용  
{ANNUAL=[바질, 딜], BIENNIAL=[라벤더]}



### 핵심정리
- 배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않다
- EnumMap을 사용하자


----------------


두 열거 타입 값들을 매핑하느라 ordinal을 두 번이나 쓴 배열들의 배열을 본 적이 있을 것이다.
다음은 이 방식을 적용해 두 가지 상태(Phase)를 전이(Transition)와 매핑하도록 구현한 프로그램이다.  
- 액체(LIQUID)에서 고체(SOLID)로의 전이는 응고(FREEZE)
- 액체에서 기체(GAS)로의 전이는 기화(BOIL)


```java
// 코드 37-5 배열들의 배열의 인덱스에 ordinal()을 사용 - 따라하지 말 것!
public enum Phase {
SOLID, LIQUID, GAS;

public enum Transition {
MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

      // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다.
      private static final Transition[][] TRANSITIONS = {
         { null, MELT, SUBLIME },
         { FREEZE, null, BOIL },
         { DEPOSIT, CONDENSE, null }
      };
      
      // 한 상태에서 다른 상태로의 전이를 반환한다.
      public static Transition from(Phase from, Phase to) {
         return TRANSITIONS[from.ordinal()][to.ordinal()];
      }
}
}
```
앞서 보여준 간단한 정원 예제와 마찬가지로 컴파일러는 ordinal과 배열 인덱스의 관계를 알 도리가 없다.  
즉, Phase나 Phase.Transition 열거 타입을 수정하면서 상전이 표 TRANSITIONS를 함께 수정하지 않거나 실수로 잘못 수정하면 런타임 오류가 날 것이다. 
그리고 상전이 표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며 null로 채워지는 칸도 늘어날 것이다.

다시 이야기하지만 EnumMap을 사용하는 편이 훨씬 낫다.  
전이 하나를 얻으려면 이전 상태(from)와 이후 상태(to)가 필요하니, 맵 2개를 중첩하면 쉽게 해결할 수 있다.

- 다차원 관계는 EnumMap<..., EnumMap<...>>으로 표현

```java
// 37-6 중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결했다.
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOILD(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);
        
        private final Phase from;
        private final Phase to;
        
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }
        
        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>>
            m = Stream.of(values())
                .collect(groupingBy(t -> t.from, // 바깥 맵의 key
                () -> new EnumMap<>(Phase.class), // 바깥 맵의 구현체
                toMap(t -> t.to, t -> t, 
                (x, y) -> y, () -> new EnumMap<>(Phase.class))));
              
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```
상전이 맵을 초기화 하는 코드는 제법 복잡하다.  
이 맵의 타입인 Map<Phase, Map<Phase, Transition>> 은 "이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵" 이라는 뜻이다.

이제 여기에 새로운 상태인 PLASMA를 추가해보자.  
이 상태와 연결된 전이는 2개다.  
첫 번째는 기체에서 플라스마로 변하는 이온화(IONIZE)이고, 둘째는 플라즈마에서 기체로 변하는 탈이온화(DEIONIZE)다.  
배열로 만든 코드를 수정하려면 배열을 원소 16개짜리로 교체해야 한다.

반면 EnumMap 버전에서는 상태 목록에 PLASMA를 추가하고, 전이 목록에 IONIZE(GAS, PLASMA)와 DEIONIZE(PLASMA, GAS)만 추가하면 끝이다.

```java
public enum Phase {
SOLID, LIQUID, GAS, PLASMA;
    public enum Transition {
    IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
    
          .. // 나머지 코드는 그대로다.
    }
}
```

나머지는 기존 로직에서 잘 처리해주어 잘못 수정할 가능성이 극히 적다.  
실제 내부에서는 맵들의 맵이 배열들의 배열로 구현되니 낭비되는 공간과 시간도 거의 없이 명확하고 유지보수하기 좋다.
