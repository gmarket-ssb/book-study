## 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

*원서제목: Prefer dependency injection to hardwiring resources*

<br>

**SpellChecker(맞춤법 검사기)** 내부에 **dictionary(사전)** 라는 필드가 있을 때  
아래와 같이 SpellChecker의 여러 메소드들은 dictionary 객체에 의존하여 동작한다.

아래 hardwiring-resource가 존재하는 정적 유틸리티 클래스 예시에서는 SpellChecker로 영어 이외의 언어 맞춤법을 검사할 수 없다. 만약 한글 맞춤법을 검사하고자 한다면 dictionary를 교체하는 메소드를 별도로 추가해야 하는데 이 방식은 불변을 보장하지 않아 오류를 만들어내기 쉬우며, 멀티스레드 환경에서도 사용이 불가하다.

**[Hard Wiring 예시]**

```java
public class SpellChecker {
    private static final Lexicon dictionary = new EnglishDictionary();
    
    private SpellChecker() {} // 객체 생성 방지
    
    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}

class Test {
    void test() {
        SpellChecker.isValid("hey");
    }
}
```

<br>

따라서 이 경우에는 정적 유틸리티 클래스나 싱글턴으로 만드는 것 보다는 Dependency Injection을 적용한 유연한 클래스로 구현하는것이 좋다. 아래와 같이 생성자에 dictionary를 넘기며 생성한다면 여러 언어의 SpellChecker를 만들 수 있다.

**[Dependency Injection 예시]**

```java
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}

class Test {
    void test() {
        SpellChecker korSpellChecker = new SpellChecker(new KoreanDictionary());
        SpellChecker engSpellChecker = new SpellChecker(new EnglishDitionary());
        korSpellChecker.isValid("안녕");
        engSpellChecker.isValid("hey");
    }
}
```

<br>

이 패턴의 변형으로, 생성자에 자원 팩터리를 넘겨주는 방식도 있다.  
아래는 팩터리가 생성한 **Tile**들로 구성된 **Mosaic** 객체를 만드는 메소드이다.

```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

<br>

Dependency Injection은 유연성과 테스트 용이성을 개선해주지만 의존성이 수천개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 한다. 이는 스프링같은 DI 프레임워크를 사용하면 해소할 수 있다.

#### 요약

- 클래스가 내부 자원에 의존하여 동작한다면 싱글턴이나 정적 유틸리티 클래스로 구현하지 말자.
- 자원들을 클래스 내에서 직접 만들기보다는 필요한 자원이나 팩터리를 생성자에 넘겨주자. 
- Dependency Injection은 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.
- DI 프레임워크를 사용하면 복잡한 의존성이 어느정도 해소된다.
