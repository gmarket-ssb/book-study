# 표준 함수형 인터페이스를 사용하라
(람다식 자체가 하나의 인터페이스를 정의하기 때문에) 두 개 이상의 추상 메소드가 선언된 인터페이스는 람다식을 이용해서 구현 객체를 생성할 수 없다.
<br><br>

### 표준 함수형 인터페이스
`java.util.function` 패키지에 있는 표준 API 의 함수형 인터페이스는 크게 아래 표로 구분된다.<br>
구분 기준은 "인터페이스에 선언된 추상 메소드의 매개값과 리턴값의 유무" 이다.

|종류|인터페이스|추상메소드 특징|함수 시그니처|예|
|---|-------|-----------|----------|-|
|Consumer|Consumer<T>|매개값O,리턴값X|void accept(T t)|System.out::println|
|Supplier|Supplier<T>|매개값X,리턴값O|T get()|Instant::now|
|Function|Function<T,R>|매개값O,리턴값O(주로 타입변환)|R apply(T t)|Arrays::asList|
|Operator|UnaryOperator<T>|매개값O,리턴값O(주로 연산)|T apply(T t)|String::toLowerCase|
||BinaryOperator<T>||T apply(T t1, T t2)|BigInteger::add|
|Predicate|Predicate<T>|매개값O,리턴값O(boolean)|boolean test(T t)|Collection::isEmpty|

<br>
  
### 표준 함수형 인터페이스 vs 직접 만든 함수형 인터페이스
* 아래와 같은 상황에서 직접 만든 함수형 인터페이스를 사용하면 된다.
  * 표준 인터페이스 중 필요한 용도에 맞는 게 없을 때
  * 전용 함수형 인터페이스를 구현해야 할 때
    * 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
    * 반드시 따라야 하는 규약이 있다.
    * 유용한 default mmethod 를 제공할 수 있다. (ex. `Comparator<T>` 인터페이스)
<br><br>

### `@FunctionalInterface`
* 인터페이스가 람다용으로 설계된 것임을 명시적으로 알려주는 어노테이션
* 인터페이스가 추상 메소드를 두 개 이상 가지고 있는 경우 컴파일 오류가 발생한다.
* **따라서** 어노테이션 자체는 optional 일지라도 직접 만든 함수형 인터페이스에는 항상 추가하자.
<br><br>

### 결론
* 표준 함수형 인터페이스를 사용하되, 직접 만드는 케이스가 더 나은지도 고려해보자.
