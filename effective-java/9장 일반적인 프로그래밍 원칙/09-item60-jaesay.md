# 정확한 답이 필요하다면 float와 double은 피하라

float와 double은 이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 ‘근사치'로 계산하도록 설계되었다. 따라서 정확한 결과가 필요할 때는 사용하면 안된다. 특히 금융 계산에는 BigDecima, int, 혹은 long을 사용해야 한다.

## 예제

주머니에는 1달러가 있고, 선반에 10센트, 20센트, 30센트, ..., 1달러짜리의 맛있는 사탕이 놓여 있다고 해보자. 10센트짜리부터 하나씩, 살 수 있을 때까지 사보자. 사탕을 몇개까지 살수 있고, 잔돈은 얼마가 남을까?

### double 사용시 오류 발생

```java
private static void example1() {
    double funds = 1.00;
    int itemsBought= 0;
    for (double price = 0.10; funds >= price; price += 0.10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println("itemsBought = " + itemsBought); // itemsBought = 3
    System.out.println("funds = " + funds); // funds = 0.3999999999999999
}
```

### BigDecimal을 사용한 해법, 속도가 느리고 쓰기 불편하다.

```java
private static void example2() {
    final BigDecimal TEN_CENTS = new BigDecimal(".10");

    int itemBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >=0; price = price.add(TEN_CENTS)) {
        funds = funds.subtract(price);
        itemBought++;
    }
    System.out.println("itemBought = " + itemBought); // itemBought = 4
    System.out.println("funds = " + funds); // funds = 0.00
}
```

### 정수 타입을 사용한 해법

```java
private static void example3() {
    int itemsBought= 0;
    int funds = 100;
    for (double price = 10; funds >= price; price += 10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println("itemsBought = " + itemsBought); // itemsBought = 4
    System.out.println("funds = " + funds); // funds = 0
}
```

## 핵심정리

- 정확한 답이 필요한 계산에는 float나 double을 피하라.
- 코딩 시의 불편함이나 성능 저하를 신경 쓰지 않겠다면 BigDecimal을 사용하라.
- 반면, 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하라. 열여덟 자리를 넘어가면 BigDecimal을 사용해야 한다.