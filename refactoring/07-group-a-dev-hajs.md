## 냄새 7. 뒤엉킨 변경 (Divergent Change)

> 뒤엉킨 변경은 단일 책임 원칙(SRP)이 제대로 지켜지지 않을 때 나타난다.

어떤 한 모듈이 여러가지 이유로 다양하게 변경되어야 하는 상황에서는 서로 다른 문제는 서로 다른 모듈에서 해결해야 한다.
<br><br>
**관련 리팩토링 기술**
  * [단계 쪼개기 (Split Phase)](https://github.com/gmarket-ssb/book-study/blob/main/refactoring/07-group-a-dev-hajs.md#%EB%A6%AC%ED%8C%A9%ED%86%A0%EB%A7%81-24-%EB%8B%A8%EA%B3%84-%EC%AA%BC%EA%B0%9C%EA%B8%B0)
  * [함수 옮기기 (Move Function)](https://github.com/gmarket-ssb/book-study/blob/main/refactoring/07-group-a-dev-hajs.md#%EB%A6%AC%ED%8C%A9%ED%86%A0%EB%A7%81-25-%ED%95%A8%EC%88%98-%EC%98%AE%EA%B8%B0%EA%B8%B0)
  * 함수 추출하기 (Extract Function)
  * [클래스 추출하기 (Extract Class)](https://github.com/gmarket-ssb/book-study/blob/main/refactoring/07-group-a-dev-hajs.md#%EB%A6%AC%ED%8C%A9%ED%86%A0%EB%A7%81-26-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%B6%94%EC%B6%9C%ED%95%98%EA%B8%B0)


---

### 리팩토링 24. 단계 쪼개기
> 모듈성
서로 다른 일을 하는 코드를 발견하명 각기 다른 모듈로 분리하는 방법을 모색하자.<br>
모듈이 잘 분리되어 있다면 다른 모듈의 상세 내용은 전혀 기억하지 못해도 원하는대로 수정을 끝마칠 수 있다. (이 차이는 코드에서 훨씬 분명하게 드러낼 수 있다.)
* before
  ```java
  public class PriceOrder {
      public double priceOrder(Product product, int quantity, ShippingMethod shippingMethod) {
          final double basePrice = product.basePrice() * quantity;
          final double discount = Math.max(quantity - product.discountThreshold(), 0) * product.basePrice() * product.discountRate();
          
          final double shippingPerCase = (basePrice > shippingMethod.discountThreshold()) ? shippingMethod.discountedFee() : shippingMethod.feePerCase();
          final double shippingCost = quantity * shippingPerCase;
          final double price = basePrice - discount + shippingCost;
          return price;
      }
  }
  ```
  * 문제1. 한 메소드 안에서 여러 일을 하고 있음 (기본 단가 계산, 배송비 포함 단가 계산)
  * 문제2. 메소드의 역할을 한 눈에 이해하기 어려움 (좋지 못한 코드)


* after
  ```java
  public record PriceData(double basePrice, double discount, int quantity) {}
  
  public class PriceOrder {
      public double priceOrder(Product product, int quantity, ShippingMethod shippingMethod) {
          final PriceData priceData = calculatePriceData(product, quantity);
          return applyShipping(priceData, shippingMethod);
      }

      private PriceData calculatePriceData(Product product, int quantity) {
          final double basePrice = product.basePrice() * quantity;
          final double discount = Math.max(quantity - product.discountThreshold(), 0) * product.basePrice() * product.discountRate();
          return new PriceData(basePrice, discount, quantity);
      }

      private double applyShipping(PriceData priceData, ShippingMethod shippingMethod) {
          final double shippingPerCase = (priceData.basePrice() > shippingMethod.discountThreshold()) ? shippingMethod.discountedFee() : shippingMethod.feePerCase();
          final double shippingCost = priceData.quantity() * shippingPerCase;
          return priceData.basePrice() - priceData.discount() + shippingCost;
      }
  }
  ```
  * 개선1. 우선 메소드가 하는 일이 뭔지 이해하기 쉬워짐
  * 개선2. 기본 단가를 계산하는 메소드와 배송비 포함 단가를 계산하는 메소드를 분리함 (관점에 따라 이 또한 책임이 분리되었다고 할 수 있음)
  * 개선3. 분리한 두 메소드 사이를 이어주는 `PriceData` record 를 만들어 필요한 값에 대한 구분을 명확히 분리함
<br><br>

### 리팩토링 25. 함수 옮기기
> 모듈성, 
좋은 소프트웨어 설계의 핵심은 모듈화가 얼마나 잘 되어 있느냐를 뜻하는 모듈성이다.<br>
어떤 함수가 자신이 속한 모듈 A의 요소들보다 다른 모듈 B의 요소들을 더 많이 참조한다면 모듈 B로 옮겨줘야 마땅하다.<br>
이렇게 하면 캡슐화가 좋아져서, 이 소프트웨어의 나머지 부분은 모듈 B의 세부사상에 덜 의존하게 된다.
* before
  ```java
  public class Account {
     private int daysOverdrawn;
     private AccountType type;

     public double getBankCharge() {
         double result = 4.5;
         if (this.daysOverdrawn > 0) {
             result += this.type.overdraftCharge(this.daysOverdrawn);
         }
         return result;
     }

     private int daysOverdrawn() {
         return this.daysOverdrawn;
     }
  }
  
  public class AccountType {
      private boolean premium;
      ...

      double overdraftCharge(int daysOverdrawn) {
          if (this.isPremium()) {
              final int baseCharge = 10;
              if (daysOverdrawn <= 7) {
                  return baseCharge;
              } else {
                  return baseCharge + (daysOverdrawn - 7) * 0.85;
              }
          } else {
              return daysOverdrawn * 1.75;
          }
      }
  }
  ```
  * 문제1. `.overdraftCharge()` 메소드안에서 다른 모듈의 요소(`this.type.isPremium()`)를 참조하고 있음
   * 이 예시에서는 다른 모듈의 요소를 하나만 참조하고 있지만, 일단 옮겨보기
  
  
* after
  ```java
  public class Account {
     private int daysOverdrawn;
     private AccountType type;

     public double getBankCharge() {
         double result = 4.5;
         if (this.daysOverdrawn > 0) {
             result += this.type.overdraftCharge(this.daysOverdrawn);
         }
         return result;
     }

     private int daysOverdrawn() {
         return this.daysOverdrawn;
     }
   
     private double overdraftCharge() {
         if (this.type.isPremium()) {
             final int baseCharge = 10;
             if (this.daysOverdrawn <= 7) {
                 return baseCharge;
             } else {
                 return baseCharge + (this.daysOverdrawn - 7) * 0.85;
             }
         } else {
             return this.daysOverdrawn * 1.75;
         }
      }
  }
  
  public class AccountType {
      private boolean premium;
      ...

      double overdraftCharge(int daysOverdrawn) {
          if (this.isPremium()) {
              final int baseCharge = 10;
              if (daysOverdrawn <= 7) {
                  return baseCharge;
              } else {
                  return baseCharge + (daysOverdrawn - 7) * 0.85;
              }
          } else {
              return daysOverdrawn * 1.75;
          }
      }
  }
  ```
  * 옮기면서 안에서 사용하고 있는 `daysOverdrawn` 만을 인자로 받게 수정했지만, 사용하는 항목이 많다면 `Account` 객체를 받아도 된다.
  * 하지만, `Account` 객체를 받는 일이 생긴다면 이 메소드는 `Account` 로 다시 옮기는 걸 고려해보는 게 좋다.
<br><br>

### 리팩토링 26. 클래스 추출하기
> 추상화
메서드와 데이터가 너무 많은 클래스는 이해하기가 쉽지 않으니 잘 살펴보고 적절히 분리하자.
* before
  ```java
  public class Person {
     private String name;
     private String officeAreaCode;
     private String officeNumber;

     public String telephoneNumber() {
         return this.officeAreaCode + " " + this.officeNumber;
     }

     // getter, setter
  }
  ```
  * 문제1. 사무실 번호 관련 필드들은 `Person` 과 분리할 수 있음
  
  
* after
  ```java
  public class Person {
      private String name;
      private TelePhoneNumber telePhoneNumber;

      public Person(TelePhoneNumber telePhoneNumber, String name) {
          this.telePhoneNumber = telePhoneNumber;
          this.name = name;
      }

      // getter, setter
  }
  
  public class TelePhoneNumber {
      private String areaCode;
      private String number;

      public TelePhoneNumber(String areaCode, String number) {
          this.areaCode = areaCode;
          this.number = number;
      }

      // getter, setter

      @Override
      public String toString() {
          return this.areaCode + " " + this.number;
      }
  }
  ```
  * 개선1. 사무실 번호 관련 필드들을 `TelePhoneNumber` 클래스로 분리함
