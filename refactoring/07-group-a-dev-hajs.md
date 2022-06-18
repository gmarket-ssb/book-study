## 냄새 7. 뒤엉킨 변경 (Divergent Change)

> 좋은 소프트웨어는 응집도는 높이고 결합도는 낮춰야 한다.

어떤 한 모듈이 여러가지 이유로 다양하게 변경되어야 하는 상황에서는 서로 다른 문제는 서로 다른 모듈에서 해결해야 한다.<br>
관련 리팩토링 기술
  * [단계 쪼개기 (Split Phase)](https://github.com/gmarket-ssb/book-study/new/main/refactoring#%EB%A6%AC%ED%8C%A9%ED%86%A0%EB%A7%81-24-%EB%8B%A8%EA%B3%84-%EC%AA%BC%EA%B0%9C%EA%B8%B0)
  * [함수 옮기기 (Move Function)](https://github.com/gmarket-ssb/book-study/new/main/refactoring#%EB%A6%AC%ED%8C%A9%ED%86%A0%EB%A7%81-25-%ED%95%A8%EC%88%98-%EC%98%AE%EA%B8%B0%EA%B8%B0)
  * 함수 추출하기 (Extract Function)
  * [클래스 추출하기 (Extract Class)](https://github.com/gmarket-ssb/book-study/new/main/refactoring#%EB%A6%AC%ED%8C%A9%ED%86%A0%EB%A7%81-26-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%B6%94%EC%B6%9C%ED%95%98%EA%B8%B0)


---
### 리팩토링 24. 단계 쪼개기

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


### 리팩토링 25. 함수 옮기기


### 리팩토링 26. 클래스 추출하기
