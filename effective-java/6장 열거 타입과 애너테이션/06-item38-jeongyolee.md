## [Item 38] 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

- 열거(Enum) 타입 확장이 좋지 않은 이유
1. 확장한 타입의 원소는 기반 타입의 원소로 취급하지만 그 반대는 아니다.
2. 기반 타입과 확장된 타입들의 원소 모두를 순회할 방법도 마땅치 않다.
3. 확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 복잡해진다.
 
 ---
 #### 열거 타입 확장이 필요한 경우
- 연산코드에 어울린다 (기본 연산외에 확장 연산을 추가)
- 열거 타입이 인터페이스를 구현할 수 있다는 점을 이용하는 것이다.

```java
public interface Operation {
	double apply(double x, double y);
}
```

```java
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override 
    public String toString() {
        return symbol;
    }
}
```

위의 BasicOperation을 직접적으로 확장할 수는 없지만, Operation은 인터페이스로 확장할 수 있기 때문에
Operation을 활용해서 다른 열거타입을 구현해서 활용할 수 있다.


```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };

    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override 
    public String toString() {
        return symbol;
    }
}
```

이 확장된 열거타입의 원소까지 모두 사용하게 하는 방법
1. Class 객체에 한정적 타입 토큰을 넘기는 방법

```java
public static void main(String[] args) (
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
  for (Operation op : opEnumType.getEnumConstants()) {
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
	}
}
```

<T extends Enum<T> & Operation> 의미
Class 객체가 열거 타입인 동시에 구현한 인터페이스의 하위 타입이어야 한다.


2. Class 객체 대신 한정적 와일드카드 타입을 넘기는 방법
```java
public static void main(String[] args) (
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
  for (Operation op : opSet) {
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
  }
}
```


### 정리
> 확장할 수 있는 열거 타입이 필요한 경우 인터페이스를 정의하여 구현하자
> 열거 타입끼리는 상속이 되지 않는다.

### [추가] Enum을 상속할 수 없는 이유
열거형은 컴파일 시점에 JAVA 컴파일러가 열거형을 추상 클래스 java.lang.Enum의 하위클래스로 바꾸며, 열거형을 final class로 컴파일합니다.

따라서
- final class이므로 상속 할 수 없고
- 가능하더라도 Enum을 기본적으로 상속받고 있는 상황에서 추가 Custom Enum class을 추가로 상속하는 상황은 다중 상속이 되므로 불가합니다.
