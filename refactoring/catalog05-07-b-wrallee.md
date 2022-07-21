# 리팩토링 카탈로그 - 2

## 카탈로그 5. 조건부 로직 간소화

#### 복잡한 조건문을 다루는 기술

- **조건문 분해하기(Decompose Conditional)**

  if-else로 메서드가 길어질 경우 각각의 내부 로직들을 메서드로 분리하여 읽을 수 있는 코드로 개선할 수 있다.

- **조건식 통합하기(Consolidate Conditional Expression)**

  조건문 자체를 하나의 메서드로 묶어 빼서 이해하기 쉽게 만든다.

- **중첩 조건문을 보호 구문으로 바꾸기(Replace Nested Conditional with Guard Clauses)**

  if와 else가 동등한 상황이라고 판단될때는 if-else 문을 쓰는것이 맞다.  
  하지만 그렇지 않은 상황에서는 보호 구문(early return)을 활용하는것이 좋다.

  *AS-IS*

  ```java
  public class GuardClauses {
    public int getPoint() {
        int result;
        if (isVip()) {
            result = vipPoint();
        } else if (isPlat()) {
            result = platPoint();
        } else {
            result = normalPoint();
        }
  
        return result;
    }
    ...
  }
  ```

  *TO-BE*

  ```java
  public class GuardClauses {
    public int getPoint() {
        if (isVip()) return vipPoint();
        if (isPlat()) return platPoint();
        return normalPoint();
    }
    ...
  }
  ```
  
- 조건부 로직을 다형성으로 바꾸기(Replace Conditional with Polymorphism)

  스위치문같은경우는 다형성을 활용하는 방식으로 대체할 수 있다.
  
- 특이 케이스 추가하기(Introduce Special Case)

  일반적인 로직이 있고 특정한 로직이 있을 경우 상속을 활용하여 특정 로직을 서브 클래스로 두는 방식으로 개선할 수 있다. Null Object Pattern도 이 방식 중 하나이다.
  
- 어서션 추가하기(Introduce Assertion)

  코드가 실행하는 와중 시스템이 가정하는 특정 상태에 대한 검증을 추가하여 더 이해하기 편한 코드를 만들 수 있다.
  ```java
  double getExpenseLimit() {
      Assert.isTrue(expenseLimit != NULL_EXPENSE || primaryProject != null);
  
      return (expenseLimit != NULL_EXPENSE) ?
          expenseLimit : primaryProject.getMemberExpenseLimit();
  }
  ```

 <br/>

## 카탈로그 6. API 리팩토링

#### 쉽고 이해하고 사용할 수 있는 API를 만드는 기술

- **질의 함수와 변경 함수 분리하기(Separate Query from Modifier)**

  Side Effect를 방지하기 위해 커맨드와 쿼리를 분리한다.

- **함수 매개변수화하기(Parameterize Function)**

  비슷한 기능을 가진 함수들이 어떤 매개변수에 의해 동작이 좌우된다면 하나의 함수에 매개변수를 던지는 방식으로 리팩토링 할 수 있다.

  *AS-IS*

  ```java
  class Employee {
      void fivePercentRaise() {...};
      void tenPercentRaise() {...};
      ...
  }
  ```

  *TO-BE*

  ```java
  class Employee {
      void raise(double percentage)
  }
  ```

- **플래그 인수 제거하기(Remove Flag Argument)**

  *AS-IS*

  ```javascript
  function setDimension(name, value) {
    if (name === "height") {
      this._height = value;
      return;
    }
    if (name === "width") {
      this._width = value;
      return;
    }
  }
  ```

  *TO-BE*

  ```javascript
  function setHeight(value) {this._height = value;}
  function setWidth (value) {this._width = value;}
  ```

- **객체 통째로 넘기기(Preserve Whole Object)**

  메소드 매개변수가 너무 많아지면 DTO 객체를 만들어 넘겨서 더 이해하기 쉽도록 리팩토링 할 수 있다.

- **매개변수를 질의 함수로 바꾸기(Replace Parameter with Query)**

  질의함수를 통해 가져올 수 있는 값은 매개변수로 넘기지 말고 메소드 내에서 해당 함수를 호출하도록 변경할 수 있다.

  *AS-IS*

  ```java
  int basePrice = quantity * itemPrice;
  double seasonDiscount = this.getSeasonalDiscount();
  double fees = this.getFees();
  double finalPrice = discountedPrice(basePrice, seasonDiscount, fees);
  ```

  *TO-BE*

  ```java
  int basePrice = quantity * itemPrice;
  double finalPrice = discountedPrice(basePrice);
  ```

- **질의 함수를 매개변수로 바꾸기(Replace Query with Parameter)**

  위 항목의 반대 케이스이다. 함수의 의존성, 역할을 고려하여 선택해야 한다.

- **세터 제거하기(Remove Setting Method)**

  변경이 되어도 되는 데이터인가를 고려하여 불필요한 세터는 제거하도록 한다. 또한 단순한 세터는 단순히 값을 세팅한다는것 이외에 메소드가 어떻게 쓰이는지에 대한 의미를 드러내주지 않기 때문에 프레임워크에서 강제되지 않는 한 지양하도록 하자.

- **생성자를 팩토리 함수로 바꾸기(Replace Constructor with Factory Function)**

  생성자와는 다르게, 팩토리 함수를 사용하면 객체 생성하는 메소드에 이름을 부여할 수 있다. 또한 리턴타입을 다양하게(하위타입까지도) 선언할 수 있다.

- **함수를 명령으로 바꾸기(Replace Function with Command)**

  함수를 쓸 것이냐, 함수를 클래스로 표현할 것이냐(커맨드 패턴)에 대한 문제.  
  어떤 기능에 대한 Undo 기능을 제공한다거나, 함수의 매개변수를 빌더 형태로 제공하고싶다거나, 함수의 동작을 다형성이나 상속구조같은것을 적용하고 싶다면 함수를 클래스(커맨드)로 구현한다.

  *AS-IS*

  ```java
  class Order {
    // ...
    public double price() {
      double primaryBasePrice;
      double secondaryBasePrice;
      double tertiaryBasePrice;
      // Perform long computation.
    }
  }
  ```

  *TO-BE*

  ```java
  class Order {
    // ...
    public double price() {
      return new PriceCalculator(this).compute();
    }
  }
  
  class PriceCalculator {
    private double primaryBasePrice;
    private double secondaryBasePrice;
    private double tertiaryBasePrice;
    
    public PriceCalculator(Order order) {
      // Copy relevant information from the
      // order object.
    }
    
    public double compute() {
      // Perform long computation.
    }
  }
  ```

- **명령을 함수로 바꾸기(Replace Command with Function)**

  위 항목의 반대 케이스이다. 위에서 설명한 내용이 불필요하다고 판단되면 명령을 함수로 변경한다.

<br/>

## 카탈로그 7. 상속 다루기

#### 상속을 제대로 사용하는 기술

- **메소드 올리기(Pull Up Method)**

- **필드 올리기(Pull Up Field)**

- **생성자 본문 올리기(Pull Up Constructor Body)**

  *AS-IS*

  ```java
  class Manager extends Employee {
    public Manager(String name, String id, int grade) {
      this.name = name;
      this.id = id;
      this.grade = grade;
    }
    // ...
  }
  ```

  *TO-BE*

  ```java
  class Manager extends Employee {
    public Manager(String name, String id, int grade) {
      super(name, id);
      this.grade = grade;
    }
    // ...
  }
  ```

- **메서드 내리기(Push Down Method)**

- **필드 내리기(Push Down Field)**

- **타입 코드를 서브클래스로 바꾸기(Replace Type Code with Subclasses)**

  *AS-IS*

  ```javascript
  function createEmployee(name, type) {
    return new Employee(name, type);
  }
  ```

  *TO-BE*

  ```javascript
  function createEmployee(name, type) {
    switch (type) {
      case "engineer": return new Engineer(name);
      case "salesman": return new Salesman(name);
      case "manager":  return new Manager (name);
    }
  ```

- **서브클래스 제거하기(Remove Subclass)**

  위 항목의 반대 케이스이다. 필드/메소드 올리기를 하다보면 서브클래스가 죽은클래스가 될 수 있다. 이 때 서브클래스를 제거하면 된다. 주의할점은 해당 클래스를 사용하는곳이 없나 확인 후 만약 있다면 상위 클래스를 사용하도록 변경해준다.

- **슈퍼클래스 추출하기(Exctract Superclass)**

  전혀 다른 클래스들간에 비슷한 기능을 하는 코드가 있다면 슈퍼클래스를 만들어서 상속구조를 만드는 방식의 리팩토링이다.

- **계층 합치기(Collapse Hierachy)**

  리팩토링을 진행하다보면 의미 없어지는 계층의 클래스들이 생길 수 있는데 이 클래스들을 제거하여 상속 구조를 단축시키는 방식의 리팩토링이다.

- **서브클래스를 위임으로 바꾸기(Replace Sublcass with Delegate)**

  상속 구조가 적절하지 않다고 판단된다면 위임으로 변경한다.  
  (이펙티브 자바 아이템 18. 상속보다는 컴포지션을 사용하라)

- **슈퍼클래스를 위임으로 바꾸기(Replace Superclass with Delegate)**

  위 항목의 반대 케이스이다.

<br/>

### 참고자료

- https://refactoring.com/catalog
- https://refactoring.guru/refactoring/catalog
