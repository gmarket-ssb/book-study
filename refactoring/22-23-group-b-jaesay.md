# 냄새 23. 데이터 클래스

## 데이터 클래스란?

- 데이터 필드와 게터/세터로만 구성된 클래스

## 악취

- 데이터 클래스는 필요한 동작이 엉뚱한 곳에 정의돼 있다는 신호일 수 있다.
- 물론 예외도 있다. 대표적인 예로 단계 쪼개기의 결과로 나온 중간데이터(데이터를 전송할 용도) 구조가 있다.
    - 리팩토링 기법 간의 상충되는 부분이 있으니 잘 판단하자.

## 적용할 수 있는 리팩토링

- 필드가 public이면 ***레코드 캡슐화하기***
- 변경이 안되는 필드는 ***세터 제거하기***
- 다른 클래스에서 데이터 클래스의 게터나 세터를 사용하는 메서드가 있다면 ***함수 옮기기***로 그 메서드를 데이터 클래스로 옮길 수 있는지 확인

## 레코드 캡슐화하기

- 여기서 레코드란 public 필드만으로 구성된 데이터 클래스를 말한다.
- ~~가변 데이터를 다룰때는 레코드보다는 객체를 선호한다.~~

### **장점**

- 사용자는 무엇이 저장된 값이고 무엇이 계산된 값인지 알필요가 없다.
- 필드 이름을 바꿔도 기존 이름과 새 이름 모두를 각각의 메서드로 제공할 수 있어서 사용자가 새로운 메서드로 옮겨갈때까지 점진적으로 수정할 수 있다.

### **예제**

1. before

    ```java
    public class Organization {
        public String name;
        public String country;
    }
    ```

2. after

    ```java
    // 자바17을 사용하고 불변 객체라면
    public record Organization(String name, String country) {
    }
    
    public class Organization {
        private String name;
        private String country;
    
            // getter, setter
        // ...
    }
    ```

# 냄새 23. 상속 포기

## 악취

- 서브클래스가 슈퍼클래스에서 제공하는 메소드나 데이터를 잘 활용하지 않을때
    - ***메서드 내리기***
    - ***필드 내리기***
- 서브클래스가 부모의 동작은 필요로하지만  인터페이스를 따르고 싶지 않을때
    - ***서브클래스를 위임으로 바꾸기***
    - ***슈퍼클래스를 위임으로 바꾸기***

## 메서드 내리기/필드 내리기

특정 서브 클래스 하나(혹은 소수)와만 관련된 메서드는 슈퍼클래스에서 제거하고 해당 서브 클래스들에 추가하는 편이 깔끔하다. 다만, 이 리팩터링은 해당 기능을 제공하는 서브클래스가 정확히 무엇인지를 호출자가 알고 있을 때만 적용할 수 있다. 그렇지 못한 상황이라면 서브클래스에 따라 다르게 동작하는 슈퍼클래스의 기만적인 조건부 로직을 다형성으로 바꿔야 한다.

### 절차

1. 대상 메서드를 모든 서브클래스에 복사한다.
2. 슈퍼클래스에서 그 메서드를 제거한다.
3. 테스트한다.
4. 이 메서드를 사용하지 않는 모든 서브 클래스에서 제거한다.
5. 테스트한다.

### 예제

1. before

    ```java
    public class Employee {
        protected Quota quota;
    
        protected Quota getQuota() {
            return new Quota();
        }
    }
    
    public class Engineer extends Employee {
    }
    
    public class Salesman extends Employee {
    }
    ```

2. after

    ```java
    public class Employee {
    }
    
    public class Engineer extends Employee {
    }
    
    public class Salesman extends Employee {
        protected Quota quota;
    
        protected Quota getQuota() {
            return new Quota();
        }
    }
    ```

# 냄새 24. 주석

## 악취

- 주석이 장황하게 달린 원인이 코드를 잘못 작성했기 때문인 경우가 의외로 많다.

## 리팩토링 기법

- 특정 코드 블록이 하는 일에 주석을 남기고 싶다면 ***함수 추출하기***
- 이미 추출되어 있는 함수임에도 여전히 설명이 필요하다면 ***함수 선언 바꾸기***
- 시스템이 동작하기 위한 선행조건을 명시하고 싶다면 ***어서션 추가하기***

## 어서션 추가하기

- 어서션은 항상 참이라고 가정하는 조건부 문장으로, 어서션이 있고 없고가 프로그랭 기능의 정상 동작에 아무런 영향을 주지 않도록 작성해야 한다.
    - 자바에서는 컴파일할때 assert문은 없어짐
    - intellij 같은 IDE는 테스트 코드를 실행할때는 assertion을 확인해주는 옵션이 켜져있음 (-ea : enable assertion)
    - 프로그램에서 반드시 체크해야되는것은 if문이나 switch문 사용
- 어서션은 프로그램이 어떤 상태임을 가정한 채 실행되는지를 다른 개발자에게 알려주는 훌륭한 소통 도구

### 절차

- 참이라고 가정하는 조건이 보이면 그 조건을 명시하는 어서션을 추가한다.

### 예시

1. before

    ```java
    public class Customer {
        private Double discountRate;
        public double applyDiscount(double amount) {
            return (this.discountRate != null) ? amount - (this.discountRate * amount) : amount;
        }
        public Double getDiscountRate() {
            return discountRate;
        }
    
        public void setDiscountRate(Double discountRate) {
            this.discountRate = discountRate;
        }
    }
    ```

2. after

    ```java
    public class Customer {
        private Double discountRate;
        public double applyDiscount(double amount) {
            return (this.discountRate != null) ? amount - (this.discountRate * amount) : amount;
        }
        public Double getDiscountRate() {
            return discountRate;
        }
    
        public void setDiscountRate(Double discountRate) {
            assert discountRate != null && discountRate > 0;
            this.discountRate = discountRate;
        }
    }
    
    ```