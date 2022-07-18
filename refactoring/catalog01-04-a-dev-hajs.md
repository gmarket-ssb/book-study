## 카탈로그 1. 기본기술
> 가장 자주 사용하는 리팩토링 기술

- 함수 추출하기 (Extract Function)
- 함수 인라인하기 (Inline Function)
- **변수 추출하기 (Extract Variable)**
- **변수 인라인하기 (Inline Variable)**
- 함수 선언 변경하기 (Change Function Declaration)
  - 함수의 이름, 매개변수, 리턴 타입 등을 변경하는 리팩토링 기법
- 변수 캡슐화하기 (Encapsulate Variable)
- **변수 이름 바꾸기 (Rename Variable)**
- 매개변수 객체 만들기 (Introduce Parameter Object)
  - 매개변수가 많을 때 사용하는 리팩토링 기법
- 여러 함수를 클래스로 묶기 (Combine  Functions into Class)
- 여러 함수를 변환 함수로 묶기 (Combine Functions into Transform)
- 단계 쪼개기 (Split Phase)
  - 한 함수에서 하고 있는 많은 일들을 세부적으로 분리하는 리팩토링 기법



## 카탈로그 2. 캡슐화
> 모듈에서 외부 시스템으로 공개하지 않아도 되는 데이터를 숨기는 기술

- 레코드 캡슐화하기 (Encapsulate Record)
  - public 필드를 record 를 통해서 감추는 리팩토링 기법
- **컬렉션 캡슐화하기 (Encapsulate Collection)**
  - 컬렉션에 접근하는 기술 자체를 감추는 리팩토링 기법
- 기본형을 객체로 바꾸기 (Replace Primitive with Object)
- 임시 변수를 질의 함수로 바꾸기 (Replcae Temp with Query)
- 클래스 추출하기 (Extract Class)
- 클래스 인라인하기 (Inline Class)
- 위임 숨기기 (Hide Delegate)
  - 클래스가 또 다른 클래스와 어떤 연관관계를 가지고 있는지를 감추는 리팩토링 기법
- 중재자 제거하기 (Remove Middle Man)
  - '위임 숨기기' 의 반대 개념으로, 너무 많이 캡슐화가 된 경우 사용하기 좋은 리팩토링 기법
- **알고리듬 교체하기 (Substitute Algorithm)**
  - 하나의 함수 내에서 함수가 가지고 있는 특정 알고리즘을 바꾸는 리팩토링 기법
  - 캡슐화를 하는 과정이라기 보다는 캡슐화가 잘 되었을 때 얻을 수 있는 장점에 해당하는 기법



## 카탈로그 3. 기능 옮기기



## 카탈로그 4. 데이터 조직화
