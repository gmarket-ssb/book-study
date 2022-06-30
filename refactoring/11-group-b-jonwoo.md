# 냄새11. 기본형 집착

-   도메인에 필요한 기본 타입을 만들지 않고 프로그래밍 언어가 제공하는 기본 타입을 사용하는 경우가 많음  
    예. 전화번호, 좌표, 돈, 범위, 수량 등
-   기본형으론 단위 표기법을 표현하기 어려움
-   관련 리팩토링 기술
    -   기본형을 객체로 변경
    -   타입 코드를 서브클래스로 바꾸기
    -   조건부 로직을 다형성으로 바꾸기
    -   클래스 추출하기
    -   배개변수 객체 만들기

## 기본형을 객체로 바꾸기

-   개발 초기에는 기본형으로 표현한 데이터가 나중엔 다양한 기능을 필요로 하는 경우가 발생함
    -   문자열로 표현하던 전화번호의 지역 코드가 필요하거나 다양한 포맷을 지원하는 경우
    -   숫자로 표현하던 온도의 단위를 변환하는 경우
-   기본형을 사용한 데이터를 감싸 줄 클래스로 만들면, 필요한 기능을 추가할 수 있다.

```
class Machine {
  int temperature;
  ...
}

// after
class Machine {
  Temperature temperature;
  ...
}

class Temperature {
  int value;

  public Temperature(int value) {
    check(value);  // 체크 로직 등 필요한 로직을 삽입할 수 있다.
  }
}
```

## 타입 코드를 서브클래스로 바꾸기

* 비슷하지만 다른 것들을 표현해야 하는 경우 string, enum, number 등으로 표현하기도 함
  예. 주문 : '일반주문', '빠른주문'    직원타입 : '엔지니어', '매니저', '세일즈'
* 타입을 서브클래스로 바꾸는 계기
  * 조건문으로 다형성을 표현할 수 있을 때 서브클래스를 만들고 '조건부 로직을 다형성으로 바꾸기'를 적용
  * 특정 타입에만 유효한 필드가 있을 때 서브클래스를 만들고 '필드 내리기'를 사용

```java
// before
class ClassMember {
  Type type;
}

enum Type {
  STUDENT, TEACHER
}

// after
abstract class Type {
   // 필수 스펙, 구현체
}

class Student extends Type { /* student 한정 기능, 구현체 */ }
class Teacher extends Type { /* teacher 한정 기능, 구현체 */ }

class ClassMember {
  Type type
}
```

## 조건부 로직을 다형성으로 바꾸기

* 복잡한 조건식을 상속과 다형성을 사용해 코드를 보다 명확하게 분리할 수 있다
* swtich/if 문을 사용해서 타입에 따라 각기 다른 로직을 사용하는 코드
* 기본 동작과 타입에 따른 특수한 기능이 섞여있는 경우에 상속 구조를 만들어 기본 동작을 상위 클래스에 두고 특수 기능을 하위 클래스로 옴겨 '차이점'을 강조할 수 있다
* 모든 조건문을 다형성으로 옮겨야하는가? 단순한 조건문은 그대로 두어도 좋다. -> 과용을 조심하자

> 템플릿 메소드 패턴, 전략 패턴 등을 활용

* 전략 패턴을 사용하여 swtich 문을 없앰
    ```java
    // before
    int result
    if (a==1) {
      result = do1();
    } else if (a==2) (
      result = do2();
    }
    
    // after
    Calculator calculator = new Calculator(a);
    result = calculator.do(); 
    ```
