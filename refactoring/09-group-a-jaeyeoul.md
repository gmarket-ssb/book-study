# 09 기능편애 (Feature Envy)

어떤 모듈에 있는 함수가 다른 모듈에 있는 데이터나 함수를 더 많이 참조하는 경우에 발생한다.
- 예) 다른 객체의 getter를 여러개 사용하는 메소드

## 관련 리팩토링 기술
- __함수 옮기기 (Move Function)__를 사용해서 함수를 적절한 위치로 옮긴다.
- 함수 일부분만 다른 곳의 데이터와 함수를 많이 참조한다면 __함수 추출하기 (Extract Function)__로 함수를 나눈 다음에 함수를 옮길 수 있다.

만약에 여러 모듈을 참조하고 있다면? -> 그 중에서 가장 많은 데이터를 참조하는 곳으로 옮기거나, 함수를 여러개로 쪼개서 각 모듈로 분산 시킬 수도 있다.

데이터와 해당 데이터를 참조하는 행동을 같은 곳에 두도록 하자.

예외적으로, 데이터와 행동을 분리한 디자인 패턴 (전략 패턴 또는 방문자 패턴)을 적용할 수도 있다.

```java
public class Bill {

    private ElectricityUsage electricityUsage;

    private GasUsage gasUsage;

// 기능편애 냄새가 나는 지점
    public double calculateBill() {
        var electicityBill = electricityUsage.getAmount() * electricityUsage.getPricePerUnit();
        var gasBill = gasUsage.getAmount() * gasUsage.getPricePerUnit();
        return electicityBill + gasBill;
    }

}
```
```java
public class ElectricityUsage {

    private double amount;

    private double pricePerUnit;

    public ElectricityUsage(double amount, double pricePerUnit) {
        this.amount = amount;
        this.pricePerUnit = pricePerUnit;
    }

    public double getAmount() {
        return amount;
    }

    public double getPricePerUnit() {
        return pricePerUnit;
    }
}
```
```java
public class GasUsage {

    private double amount;

    private double pricePerUnit;

    public GasUsage(double amount, double pricePerUnit) {
        this.amount = amount;
        this.pricePerUnit = pricePerUnit;
    }

    public double getAmount() {
        return amount;
    }

    public double getPricePerUnit() {
        return pricePerUnit;
    }
}
```

### 문제 1
`calculateBill()` 메서드는 `Bill` 클래스의 멤버가 아닌 `ElectricityUsage`와 `GasUsage`의 멤버에 100%기대어 각 멤버의 getter를 사용하여 구성되었다.
-> `amount`의 데이터 타입의 변화에 따라서 Bill도 자꾸 고쳐야 한다.
-> `double` 이고 다른 하나는 `BigDecimal` 혹은 커스텀 객체로 변한다면 문제가 생긴다. 왜냐면, Getter의 반환값도 바뀌니까..

### 문제 2
전기료와 가스비 산출에 사용되는 추가 항목이 발생 할 떄, `Bill`, `ElectricityUsage`, `GasUsage` 세 곳 모두 수정 확인 지점이 된다.

---

## 개선
전기료와 가스비를 얻을 수 있는 __메시지를 주고 받도록__ 개선한다.

- `getElecticityBill()` 과 `getGasBill()` 을 각각의 모듈에서 책임지도록 구성한다.
- 인터페이싱을 하는 메서드의 규격만 바뀌지 않는다면 의존클래스의 private 멤버 데이터가 수정이 어떻게 되던 Bill 클래스는 더 이상 관심을 두지 않아도 된다.

```java
public class Bill {

    private ElectricityUsage electricityUsage;

    private GasUsage gasUsage;

// 개선후 -> 이제 더 이상 각 모듈의 멤버 데이터에 의존하지 않고 메서드를 통해서 데이터를 전달 받는다.
    public double calculateBill() {
        return electricityUsage.getElecticityBill() + gasUsage.getGasBill();
    }

}
```

```java
public class ElectricityUsage {

    private double amount;

    private double pricePerUnit;

    public ElectricityUsage(double amount, double pricePerUnit) {
        this.amount = amount;
        this.pricePerUnit = pricePerUnit;
    }

    public double getAmount() {
        return amount;
    }

    public double getPricePerUnit() {
        return pricePerUnit;
    }
    // 인터페이싱 지점 
    public double getElecticityBill() {
        return this.getAmount() * this.getPricePerUnit();
    }
}

```

```java
public class GasUsage {

    private double amount;

    private double pricePerUnit;

    public GasUsage(double amount, double pricePerUnit) {
        this.amount = amount;
        this.pricePerUnit = pricePerUnit;
    }

    public double getAmount() {
        return amount;
    }

    public double getPricePerUnit() {
        return pricePerUnit;
    }
    
    // 인터페이싱지점 
    public double getGasBill() {
        return this.getAmount() * this.getPricePerUnit();
    }
}
```

