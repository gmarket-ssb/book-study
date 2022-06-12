# 냄새 6. 가변 데이터(Mutable Data)
데이터를 변경함으로써 발생하는 리스크를 줄일수 있는 리팩토링 기법들을 소개한다.

- 하나의 변수에 용도가 다른 값들을 저장하느라 값을 갱신하는 경우라면 **변수 쪼개기**
- API를 만들 때는 **질의 함수와 변경 함수 분리하기**를 활용해서 꼭 필요한 경우가 아니라면 부작용이 있는 코드를 호출할 수 없게 해야 한다.
- 수정이 가능한 필드 말고는 **세터 제거하기**
- 값을 다른 곳에서 설정할 수 있는 가변데이터는 **파생 변수를 질의함수로 바꾸기**
- **여러함수를 변환 함수로 묶기**를 활용해서 변수를 갱신하는 코드들의 유효범위를 (클래스나 변환으로) 제한한다.
- 구조체처럼 내부 필드에 데이터를 담고 있는 변수라면, 일반적으로 **참조를 값으로 바꾸기**를 적용하여, 내부 필드를 직접 수정하지 말고 구조체를 통째로 교체하는 편이 낫다.

## 변수 쪼개기

### 배경

대입이 두번 이상 이뤄진다면 여러가지 역할로 수행한다는 신호이다. 역할이 둘 이상인 변수가 있다면 쪼개야 한다. 여러 용도로 쓰인 변수는 코드를 읽는 이에게 커다란 혼란을 주기 때문이다.

### 절차

1. 변수를 선언한 곳과 값을 처음 대입하는 곳에서 변수 이름을 바꾼다.
2. 가능하면 이때 불변으로 선언한다. (final)
3. 두번 째로 변수에 값을 대입하기 전까지 변수가 쓰인 곳을 첫번째 바꾼 이름으로 변경한다.
4. 테스트 및 반복

### 예시

헤기스(양의 내장으로 만든 스코틀랜드 음식)라는 음식이 다른 지역으로 전파된 거리를 구하는 함수

#### before

```java
public double distanceTravelled(int time) {
    double result;
    double acc = primaryForce / mass; // 첫번째 변수
    int primaryTime = Math.min(time, delay);
    result = 0.5 * acc * primaryTime * primaryTime;

    int secondaryTime = time - delay;
    if (secondaryTime > 0) {
        double primaryVelocity = acc * delay;
        acc = (primaryForce + secondaryForce) / mass; // 두번째 변수
        result += primaryVelocity * secondaryTime + 0.5 * acc * secondaryTime + secondaryTime;
    }

    return result;
}
```

#### after

```java
public double distanceTravelled(int time) {
    double result; // 결과값을 계산하는 용도이기 때문에 사용
    final double primaryAcc = primaryForce / mass; // 2. 변경을 못하도록 하는 값을 final로 명시
    int primaryTime = Math.min(time, delay);
    result = 0.5 * primaryAcc * primaryTime * primaryTime;

    int secondaryTime = time - delay;
    if (secondaryTime > 0) {
        final double primaryVelocity = primaryAcc * delay;
        final double secondaryAcc = (primaryForce + secondaryForce) / mass;
        result += primaryVelocity * secondaryTime + 0.5 * secondaryAcc * secondaryTime + secondaryTime;
    }

    return result;
}
```

## 질의 함수와 변경 함수 분리하기

### 배경

겉보기 부수효과가 있는 함수와 없는 함수는 명확히 구분하는 것이 좋다. 이를 위한 한가지 방법은 ‘질의 함수는 모두 부수효과가 없어야 한다'라는 규칙을 따르는 것이다. 이런 함수는 어느 때건 원하는 만큼 호출해도 아무 문제 없다. 호출하는 문장의 위치를 호출하는 함수 안 어디로든 옮겨도 되며 테스트하기도 쉽다. ‘겉보기’ 부수효과 라고 한 데는 이유가 있다. 캐싱의 경우 객체의 상태를 변경하지만 객체 밖에서는 관찰할 수 없다. 즉, 겉보기 부수효과 없이 어떤 순서로 호출하든 모든 호출에 항상 똑같은 값을 반환할 뿐이다.

### 절차

1. 질의/변경을 함께 가진 함수를 복제하고 질의 목적에 충실한 이름을 짓는다.
2. 새 질의 함수에서 부수 효과를 제거한다.
3. 원래 함수(변경 함수)를 호출하는 곳을 모두 찾아낸다. 호출하는 곳에서 반환 값을 사용한다면 질의 함수를 호출하도록 바꾸고, 원래 함수를 호출하는 코드를 아래 줄에 새로 추가한다. 하나 수정할 때마다 테스트한다.
4. 원래 함수에서 질의 관련 코드를 제거한다.

### 예시

이름 목록을 훑어 악당을 찾는 함수

#### before

```java
public String alertForMiscreant(List<Person> people) {
    for (Person p : people) {
        if (p.getName().equals("Don")) {
            setOffAlarms();
            return "Don";
        }

        if (p.getName().equals("John")) {
            setOffAlarms();
            return "John";
        }
    }

    return "";
}
```

#### after

```java
// 변경
public void alertForMiscreant(List<Person> people) {
    if (!findMiscreant(people).isBlank()) {
        setOffAlarms();
    }
}

// 질의
public String findMiscreant(List<Person> people) {
    for (Person p : people) {
        if (p.getName().equals("Don")) {
            return "Don";
        }

        if (p.getName().equals("John")) {
            return "John";
        }
    }

    return "";
}
```

## 세터 제거하기

### 배경

세터 메서드가 있다고 함은 필드가 수정될 수 있다는 뜻이다. 객체 생성 후에는 수정되지 않길 원하는 필드라면 세러를 제공하지 않았을 것이다. 그러면 해당 필드는 오직 생성자에서만 설정되며, 수정하지 않겠다는 의도가 명명백백해지고, 변경 가능성이 봉쇄된다.

### 절차

1. 수정되지 않는 필드는 생성자에 추가
2. 생성자 밖에서 세터 호출하는 곳을 찾아 제거하고, 새로운 생성자를 사용하도록 수정
3. 생성자 내 세터 메서드를 인라인 메서드로 수정하고 가능하다면 해당 필드는 불변으로 만듦
4. 테스트

### 예시

#### before

```java
public class Person {

    private String name;

    private int id;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }
}
```

#### after

```java
public class Person {

    private String name;

    private final int id;

    public Person(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }
}
```

## 파생 변수를 질의 함수로 바꾸기

가변 데이터를 완전히 배제하기란 현실적으로 불가능할 때가 많지만, 가변 데이터의 유효 범위를 가능한 한 좁혀야 한다. 효과가 좋은 방법으로 값을 쉽게 계산해낼 수 있는 변수들을 모두 제거할 수 있다. 계산 과정을 보여주는 코드 자체가 데이터의 의미를 더 분명히 드러내는 경우도 자주 있으며 변경된 값을 깜박하고 결과 변수에 반영하지 않는 실수를 막아준다. 여기에는 합당한 예외도 있다. 피연산자 데이터가 불변이라면 계산 결과도 일정하므로 역시 불변으로 만들 수 있다.

### 절차

1. 변수 값이 갱신되는 지점을 모두 찾는다.
2. 해당 변수의 값을 계산해주는 함수를 만든다. (temp)
3. 해당 변수가 사용되는 모든 곳에 어서션을 추가하여 함수의 계산 결과가 변수의 값과 같은지 확인한다.
4. 변수를 읽은 코드를 모두 함수로 대체하고 기존 갱신코드 제거한다.

### 예시

#### before

```java
public class ProductionPlan {

    private double production; // 파생 변수
    private List<Double> adjustments = new ArrayList<>();

    public void applyAdjustment(double adjustment) {
        this.adjustments.add(adjustment);
        this.production += adjustment; // 조정 값(adjustments)을 적용하는 과정에서 직접 관련이 없는 누적값(production) 까지 갱신했다.
    }

    public double getProduction() {
        return this.production;
    }
}
```

#### after

```java
public class ProductionPlan {

    private List<Double> adjustments = new ArrayList<>();

    public void applyAdjustment(double adjustment) {
        this.adjustments.add(adjustment);
    }

		// 질의 함수
    public double getProduction() {
        return this.adjustments.stream().mapToDouble(Double::valueOf).sum();
    }
}
```

## 여러 함수를 변환 함수로 묶기

### 배경

소프트웨어는 데이터를 입력받아서 여러 가지 정보를 도출하곤 하곤 한다. 이렇게 도출된 정보는 여러 곳에서 재사용될 수 있는데, 그러다 보면 이 정보가 사용되는 곳마다 같은 도출 로직이 반복되기도 한다. 이런 도출 작업들을 한데 모아두면 검색과 갱신을  일관된 장소에서 처리할 수 있고 로직 중복도 막을 수 있다. 이렇게 하기 위한 방법으로 변환 함수를 사용할 수 있다. 변환 함수는 원본 데이터를 입력 받아서 필요한 정보를 모두 도출한 뒤, 각각을 출력 데이터의 필드에 넣어 반환한다. 이렇게 해두면 도출과정을 검토할 일이 생겼을 때 변환 함수만 살펴보면 된다.

### 절차

1. 변환할 레코드를 입력받아서 값을 그대로 반환하는 변환 함수를 만든다. (대체로 깊은 복사)
2. 묶을 함수 중 함수 하나를 골라서 본문 코드를 변환 함수로 옮기고, 처리 결과를 레코드에 새 필드로 기록한다. 그런 다음 클라이언트 코드가 이 필드를 사용하도록 수정한다.
3. 테스트한다.

### 예시

```java
public class Client2 {

    /**
     * 기본 요금
     */
    private double base;
    /**
     * 세금
     */
    private double taxableCharge;

    public Client2(Reading reading) {
				// 이곳 저곳 자주 사용되는 계산 코드
        this.base = baseRate(reading.month(), reading.year()) * reading.quantity();
        this.taxableCharge = Math.max(0, this.base - taxThreshold(reading.year()));
    }

    /**
     * 세금을 부과할 소비량
     * @param year
     * @return
     */
    private double taxThreshold(Year year) {
        return 5;
    }

    /**
     * 기본 소비량
     * @param month
     * @param year
     * @return
     */
    private double baseRate(Month month, Year year) {
        return 10;
    }

    public double getBase() {
        return base;
    }

    public double getTaxableCharge() {
        return taxableCharge;
    }
}
```

#### after

```java
public class Client2 extends ReadingClient {

    private double base;
    private double taxableCharge;

    public Client2(Reading reading) {
        EnrichReading enrichReading = enrichReading(reading);
        this.base = enrichReading.baseCharge();
        this.taxableCharge = enrichReading.taxableCharge();
    }

    public double getBase() {
        return base;
    }

    public double getTaxableCharge() {
        return taxableCharge;
    }
}

/**
 * 자주 사용하는 필드 (baseCharge, taxableCharge)를 가지고 있음
 */
public record EnrichReading(Reading reading, double baseCharge, double taxableCharge) {
}

public class ReadingClient {
    protected double taxThreshold(Year year) {
        return 5;
    }

    protected double baseRate(Month month, Year year) {
        return 10;
    }

    // 변환 함수
    protected EnrichReading enrichReading(Reading reading) {
        return new EnrichReading(reading, baseCharge(reading), taxableCharge(reading));
    }

    private double taxableCharge(Reading reading) {
        return Math.max(0, baseCharge(reading) - taxThreshold(reading.year()));
    }

    private double baseCharge(Reading reading) {
        return baseRate(reading.month(), reading.year()) * reading.quantity();
    }
}
```

## 참조를 값으로 바꾸기

### 배경

객체(데이터 구조)를 다른 객체(데이터 구조)에 중첩하면 내부 객체를 참조 혹은 값으로 취급할 수 있다. 필드를 값으로 다룬다면 내부 객체의 클래스를 수정하여 값 객체(Value Object)로 만들 수 있다. 값 객체는 대체로 자유롭게 활용하기 좋은데, 특히 불변이기 떄문이다.

### 절차

1. 후보 클래스가 불변인지, 혹은 불변이 될 수 있는지 확인한다.
2. 각각의 세털르 하나씩 제거한다.
3. 이 값 객체의 필드들을 사용하는 동치성(equality) 비교 메서드를 만든다.

### 예시

#### before

```java
public class Person {

    private TelephoneNumber officeTelephoneNumber;

    public String officeAreaCode() {
        return this.officeTelephoneNumber.areaCode();
    }

    public void officeAreaCode(String areaCode) {
        this.officeTelephoneNumber.areaCode(areaCode);
    }

    public String officeNumber() {
        return this.officeTelephoneNumber.number();
    }

    public void officeNumber(String number) {
        this.officeTelephoneNumber.number(number);
    }

}

public class TelephoneNumber {

    private String areaCode;

    private String number;

    public String areaCode() {
        return areaCode;
    }

    public void areaCode(String areaCode) {
        this.areaCode = areaCode;
    }

    public String number() {
        return number;
    }

    public void number(String number) {
        this.number = number;
    }
}
```

#### after

```java
public class Person {

    private TelephoneNumber officeTelephoneNumber;

    public String officeAreaCode() {
        return this.officeTelephoneNumber.areaCode();
    }

    public void officeAreaCode(String areaCode) {
        this.officeTelephoneNumber = new TelephoneNumber(areaCode, this.officeNumber());
    }

    public String officeNumber() {
        return this.officeTelephoneNumber.number();
    }

    public void officeNumber(String number) {
        this.officeTelephoneNumber = new TelephoneNumber(this.officeAreaCode(), number);
    }

}

public record TelephoneNumber(String areaCode, String number) {
}
```