## [item 40] @Override 애너테이션을 일관되게 사용하라

@Override 에너테이션은 상위 타입의 메서드를 "재정의"했음을 명확히 나타낸다.
<br>이는 다양한 버그를 막아 줄 수 있는데, @Override 애너테이션을 사용함으로 상위 타입의 메서드를 올바르게 재정의 했는지 확인 가능합니다.

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram bigram) {
        return bigram.first == this.first &&
                bigram.second == this.second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }

        System.out.println(s.size());
        //결과는 260
    }
}
```

@Override를 강제로 붙이면
>java: Method does not override or implement a method from a supertype

>equals 메서드를 재정의 한게 아니라 Overloading 했다<br>
>equals를 재정의 하려면 파라미터 타입이 Object이어야 한다.

```java
@Override
public boolean equals(Object bigram) {
    if(!(bigram instanceof Bigram)) {
        return false;
    }
    Bigram b = (Bigram) bigram;
    return b.first == this.first &&
        b.second == this.second;
}
```

### 정리

- 모든 재정의한 메서드에 @Override 애너테이션을 의식적으로 달면, 개발자가 실수했을때 컴파일러가 바로 알려준다.
- @Override를 달지 않아도, 인터페이스를 상속한 구체 클래스인데 아직 구현하지 않은 추상 메서드가 남아있다면 컴파일러가 바로 사실을 알려준다.
- 대부분의 IDE는 재정의할 메서드를 선택하면 자동으로 @Override를 붙여주니 참고하자.

- **상위 클래스의 메서드를 재정의하는 모든 메서드에는 되도록 @Override 애너테이션을 달도록 하자**


