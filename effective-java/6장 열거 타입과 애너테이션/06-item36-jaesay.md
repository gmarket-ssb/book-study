# int 상수 대신 열거 타입을 사용하라

## 정수 열거 패턴

```java
private static final int APPLE_FUJI = 0;
private static final int APPLE_PIPPIN = 1;
private static final int APPLE_GRANNY_SMITH = 2;

private static final int ORANGE_NAVEL = 0;
private static final int ORANGE_TEMPLE = 1;
private static final int ORANGE_BLOOD = 2;

private static final int ELEMENT_MERCURY = 0;
private static final int PLANET_MERCURY = 0;
```

- 타입 안정성을 보장할 수 없다.
- 접두어를 써서 이름 충돌을 방지해야 한다.
- 프로그램이 깨지기 쉽다. (컴파일하면 그 값이 클라이언트 파일에 그대로 새겨짐)
- 문자열로 출력하기 까다롭다.
- 정수 열거 그룹에 속한 모든 상수를 한바퀴 순회하는 방법도 마땅치 않다.
- 그 안에 상수가 몇개인지 알기 어렵다.

## 열거 타입

### **특징**

- 클래스
- 상수 하나 당 인스턴스 하나씩 만들어 public static final 필드로 공개
- 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final
    - 확장 X, 인스턴스는 딱 하나씩만 존재
    - enum 싱글톤 예: [https://github.com/spring-projects/spring-data-redis/blob/485bcad1108cdcb250157dbbc1bdad8c7fcac4af/src/main/java/org/springframework/data/redis/serializer/ByteArrayRedisSerializer.java](https://github.com/spring-projects/spring-data-redis/blob/485bcad1108cdcb250157dbbc1bdad8c7fcac4af/src/main/java/org/springframework/data/redis/serializer/ByteArrayRedisSerializer.java)

### **장점**

- 컴파일 타임 타입 안전성 제공
- 열거 타입에는 각자의 namespace가 있어 이름이 같은 상수도 평화롭게 공존
- 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 됨
- toString 메소드는 출력하기에 적합한 문자열을 출력
- 임의의 메서드나 필드 추가 가능
- 임의의 인터페이스를 구현 가능
- Object 메서드들을 높은 품질로 구현
- Comparable과 Serializable을 구현

### **상황 별 열거 타입 선언법**

1. 명시적 생성자나 메서드 없이 쓰는 열거타입

    ```java
    public enum Apple {
        FUJI, PIPPIN, GRANNY_SMITH
    }
    ```

1. 각 상수를 특정 데이터와 연결 짓거나 상수마다 다르게 동작하게 할 떄 열거 타입

    ```java
    public enum Planet {
        // 열거 타입의 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.
        MERCURY(3.302e+23, 2.439e6),
        VENUS(4.869e+24, 6.052e6),
        EARTH(5.975e+24, 6.378e6),
        MARS(6.419e+23, 3.393e6),
        JUPITER(1.899e+27, 7.149e7),
        SATURN(5.685e+26, 6.027e7),
        URANUS(8.683e+25, 2.556e7),
        NEPTUNE(1.024e+26, 2.477e7);
    
        private final double mass; // 질량(단위: 킬로그램)
        private final double radius; // 반지름(단위: 미터)
        private final double surfaceGravity; // 표면중력(단위: m / S^2)
    
        private static final double G = 6.67300E-11;
    
        Planet(double mass, double radius) {
            this.mass = mass;
            this.radius = radius;
            surfaceGravity = G * mass / (radius * radius);
        }
    
        public double getMass() {
            return mass;
        }
    
        public double getRadius() {
            return radius;
        }
    
        public double getSurfaceGravity() {
            return surfaceGravity;
        }
    
        // 대상 객체의 질량을 입력받아 그 객체가 행성 표면에 있을 때의 무게를 반환한다.
        public double surfaceWeight(double mass) {
            return mass * surfaceGravity;
        }
    }
    ```

1. 하나의 메서드가 상수별로 다르게 동작해야 할 때 열거 타입

    ```java
    public enum Operation {
        PLUS("+") {
            @Override
            public double apply(double x, double y) {
                return x + y;
            }
        },
        MINUS("-") {
            @Override
            public double apply(double x, double y) {
                return x - y;
            }
        },
        TIMES("*") {
            @Override
            public double apply(double x, double y) {
                return x * y;
            }
        },
        DIVIDE("/") {
            @Override
            public double apply(double x, double y) {
                return x / y;
            }
        };
    
        private final String symbol;
    
        // Operation 상수가 stringToEnum Map에 추가되는 시점은 열거 타입 상수 생성 후 정적 필드가 초기화 될 때다.
        private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(toMap(Object::toString, e -> e));
    
        // private
        Operation(String symbol) {
            this.symbol = symbol;
        }
    
        public abstract double apply(double x, double y);
    
        @Override
        public String toString() {
            return symbol;
        }
    
        // toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함꼐 제공하는 걸 고려해보자
        // e.g. 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
        public static Optional<Operation> fromString(String symbol) {
            return Optional.ofNullable(stringToEnum.get(symbol));
        }
    }
    ```

1. 열거 타입 상수 일부가 같은 동작을 공유할 때 사용하는 전략 열거 타입 패턴
    1. switch method / 각 상수별로 도우미 메소드 호출 / 평일 상수 주말 상수 분기 후 주말 상수에서 메소드 재정의에서 발생하는 문제를 피하면서 열거 타입 상수끼리 코드를 공유할 수 있다.
    2. [https://stackoverflow.com/questions/23127926/static-enum-vs-non-static-enum](https://stackoverflow.com/questions/23127926/static-enum-vs-non-static-enum)

    ```java
    public enum PayrollDay {
    
        // 새로운 상수를 추가할 때 잔업수당 전략을 선택하도록 한다.
        // PayrllDay 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임한다.
        MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), THURSDAY(WEEKDAY), FRIDAY(WEEKDAY), SATURDAY(WEEKEND), SUNDAY(WEEKEND);
    
        private final PayType payType;
    
        PayrollDay(PayType payType) {
            this.payType = payType;
        }
        
        int pay(int minutesWorked, int payRate) {
            return payType.pay(minutesWorked, payRate);
        }
    
            // effectively static
        enum PayType {
            WEEKDAY {
                @Override
                int overtimePay(int minutesWorked, int payRate) {
                    return minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
                }
            },
            WEEKEND {
                @Override
                int overtimePay(int minutesWorked, int payRate) {
                    return minutesWorked * payRate / 2;
                }
            };
            
            private static final int MINS_PER_SHIFT = 8 * 60; // 하루 8시간
            abstract int overtimePay(int minutesWorked, int payRate);
            
            int pay(int minutesWorked, int payRate) {
                int basePay = minutesWorked * payRate;
                return basePay + overtimePay(minutesWorked, payRate);
            }
        }
    }
    ```

## 핵심정리

필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.