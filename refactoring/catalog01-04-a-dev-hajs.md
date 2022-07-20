## 카탈로그 1. 기본기술
> 가장 자주 사용하는 리팩토링 기술

- 함수 추출하기 (Extract Function)
- 함수 인라인하기 (Inline Function)
- **변수 추출하기 (Extract Variable)**
  - 표현식이 복잡한 경우 일부를 변수로 추출하여 이름을 주도록 하는 리팩토링 기법
  - 디버깅 시 breakpoint 를 줄 수도 있고, 상태를 출력하는 문장을 추가할 수 있다.
- **변수 인라인하기 (Inline Variable)**
- 함수 선언 변경하기 (Change Function Declaration)
  - 함수의 이름, 매개변수, 리턴 타입 등을 변경하는 리팩토링 기법
- 변수 캡슐화하기 (Encapsulate Variable)
- **변수 이름 바꾸기 (Rename Variable)**
- 매개변수 객체 만들기 (Introduce Parameter Object)
  - 매개변수가 많을 때 사용하는 리팩토링 기법
- 여러 함수를 클래스로 묶기 (Combine  Functions into Class)
  - 공통된 데이터를 중심으로 긴밀하게 엮여 동작하는 함수 무리가 있을 때 수행하는 리팩토링 기법
- 여러 함수를 변환 함수로 묶기 (Combine Functions into Transform)
  - 특정 정보를 도출하는 함수가 많을 때 하나로 합치도록 하는 리팩토링 기법
  - 검색과 갱신을 일관된 장소에서 처리할 수 있고, 중복된 로직도 막을 수 있다. 
- 단계 쪼개기 (Split Phase)
  - 한 함수에서 하고 있는 많은 일들을 세부적으로 분리하는 리팩토링 기법

<br>

## 카탈로그 2. 캡슐화
> 모듈에서 외부 시스템으로 공개하지 않아도 되는 데이터를 숨기는 기술

- 레코드 캡슐화하기 (Encapsulate Record)
  - public 필드를 record 를 통해서 감추는 리팩토링 기법
- **컬렉션 캡슐화하기 (Encapsulate Collection)**
  - 컬렉션에 접근하는 기술 자체를 감추는 리팩토링 기법
  ```javascript
  // before
  class Person {
    get courses() { return this._courses; }
    set courses(aList) { this._courses = aList; }
  }
  
  // after
  class Person {
    get courses() { return this._courses.slice(); } // 아무도 목록을 변경할 수 없도록 복제본을 제공
    addCourse(aCourse)    {...}
    removeCourse(aCourse) {...}
  }
  ```
- 기본형을 객체로 바꾸기 (Replace Primitive with Object)
  ```java
  // before
  orders.filter(o => "high" === o.priority);
  
  // after
  orders.filter(o => o.priority.higherThan(new Priority("normal"));
  ```
- 임시 변수를 질의 함수로 바꾸기 (Replcae Temp with Query)
  - 임시 변수를 아예 함수로 만들어 사용하도록 하는 리팩토링 기법
  - 추출한 함수와 원래 함수의 경계가 더 분명해져서 부자연스러운 의존 관계나 부수 효과를 찾고 제거하는 데 도움이 된다.
- 클래스 추출하기 (Extract Class)
- 클래스 인라인하기 (Inline Class)
- 위임 숨기기 (Hide Delegate)
  - 클래스가 또 다른 클래스와 어떤 연관관계를 가지고 있는지를 감추는 리팩토링 기법
  ```java
  // before
  manager = person.department.manager;
  
  // after
  manager = person.manager;
  
  class Person {
    get manager() { return this.department.manager; }
  }
  ```
- 중재자 제거하기 (Remove Middle Man)
  - '위임 숨기기' 의 반대 개념으로, 너무 많이 캡슐화가 된 경우 사용하기 좋은 리팩토링 기법
- **알고리듬 교체하기 (Substitute Algorithm)**
  - 하나의 함수 내에서 함수가 가지고 있는 특정 알고리즘을 바꾸는 리팩토링 기법
  - 캡슐화를 하는 과정이라기 보다는 캡슐화가 잘 되었을 때 얻을 수 있는 장점에 해당하는 기법

<br>

## 카탈로그 3. 기능 옮기기
> 함수나 필드 또는 문장을 적절한 위치로 옮기는 기술

- 함수 옮기기 (Move Function)
- 필드 옮기기 (Move Field)
- **문장을 함수로 옮기기 (Move Statements into Function)**
  - Statement 와 함수가 있을 때, Statement 를 함수로 넣는 리팩토링 기법
- **문장을 호출한 곳으로 옮기기 (Move Statements to Callers)**
  - '문장을 함수로 옮기기' 와 상반되는 리팩토링 기법
  - 함수를 꺼내서 호출한 쪽으로 옮기는 리팩토링 기법
- **인라인 코드를 함수 호출로 바꾸기 (Replace Inline Code with Function Call)**
  - 인라인 되어 있는 코드에 이름을 부여하는게 더 낫다고 판단되었을 때 수행하는 리팩토링 기법
- 문장 슬라이드하기 (Slide Statements)
- 반복문 쪼개기 (Split Loop)
- 반복문을 파이프라인으로 바꾸기 (Replace Loop with Pipeline)
  - 반복문을 없애고 읽기 좋고 이해하기 쉬운 파이프라인(자바의 경우 스트림)으로 바꾸는 리팩토링 기법
- 죽은 코드 제거하기 (Remove Dead Code)
  - 사용하지 않는 코드를 제거하는 리팩토링 기법

<br>

## 카탈로그 4. 데이터 조직화
> 데이터 구조를 다루는 기술

- 변수 쪼개기 (Split Variable)
  - 여러가지 용도로 사용되는 변수를 쪼개는 리팩토링 기법
- 필드 이름 바꾸기 (Rename Field)
- 파생 변수를 질의 함수로 바꾸기 (Replace Derived Variable with Query)
  - 계산을 해낼 수 있는 변수(파생변수)를 함수를 통해 계산해 올 수 있도록 하는 리팩토링 기법 
- **참조를 값으로 바꾸기 (Change References to Value)**
  - Employee 대신에 employeeName 을 주도록 변경하는 리팩토링 기법
- **값을 참조로 바꾸기 (Change Value to Reference)**
  - employeeName 대신에 Employee 을 주도록 변경하는 리팩토링 기법
  - Reference 자체를 주는게 더 낫다고 판단했을 때 수행하는 리팩토링 기법
