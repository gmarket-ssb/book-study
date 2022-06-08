## 리팩토링 7. 임시변수를 질의 함수로 바꾸기

### 임시변수를 찾는다. 그리고 질의 함수로 바꾼다. 결국 함수추출인 것이다.

#### 짧은 함수 vs 긴 함수
- 함수가 길 수록 더 이해하기 어렵다. vs 짧은 함수는 더 많은 문맥 전환을 필요로 한다.
- “과거에는” 작은 함수를 사용하는 경우에 더 많은 서브루틴 호출로 인한 오버헤드가 있었다.
- 작은 함수에 “좋은 이름”을 사용했다면 해당 함수의 코드를 보지 않고도 이해할 수 있다.
- 어떤 코드에 “주석”을 남기고 싶다면, 주석 대신 함수를 만들고 함수의 이름으로 “의도”를 표현해보자.

#### 사용할 수 있는 리팩토링 기술
- 99%는, “함수 추출하기 (Extract Function)”로 해결할 수 있다.
- 함수로 분리하면서 해당 함수로 전달해야 할 매개변수가 많아진다면 다음과 같은 리팩토링을 고려해볼 수 있다.
- 임시 변수를 질의 함수로 바꾸기 (Replace Temp with Query)
- 매개변수 객체 만들기 (Introduce Parameter Object)
- 객체 통째로 넘기기 (Preserve Whole Object)
- “조건문 분해하기 (Decompose Conditional)”를 사용해 조건문을 분리할 수 있다.
- 같은 조건으로 여러개의 Swtich 문이 있다면, “조건문을 다형성으로 바꾸기 (Replace Conditional with Polymorphism)”을 사용할 수 있다.
- 반복문 안에서 여러 작업을 하고 있어서 하나의 메소드로 추출하기 어렵다면, “반복문 쪼개기 (Split Loop)”를 적용할 수 있다.

> before

```java
package me.whiteship.refactoring._01_smell_mysterious_name._01_before;

import org.kohsuke.github.*;
```

> after

```java
package me.whiteship.refactoring._01_smell_mysterious_name._01_before;

import org.kohsuke.github.*;
```

## 리팩토링 8. 매개변수 객체 만들기

### 중복되는 parameter 들을 객채화
Tips. IDE 빠른메뉴 : Refoctoring 기능 활용( Refactor -> introduce parameter object )

### 중복되는 parameter 들을 field화
여러 함수에서 사용되는 field 가 있다면 field화 해서 적용
Tips. 'this.'를 붙여서 local 변수인지, class 변수인지를 구분

> __문제__ : before

```java
package me.whiteship.refactoring._01_smell_mysterious_name._01_before;

import org.kohsuke.github.*;
```

> after

```java
package me.whiteship.refactoring._01_smell_mysterious_name._01_before;

import org.kohsuke.github.*;
```

## 리팩토링 9. 매개변수 객체 만들기
### 고찰
어떤 한 레코드에서 구할 수 있는 여러 값들을 함수에 전달하는 경우, 해당 매개변수를 레 코드 하나로 교체할 수 있다.
매개변수 목록을 줄일 수 있다. (향후에 추가할지도 모를 매개변수까지도...) 
이 기술을 적용하기 전에 의존성을 고려해야 한다.
어쩌면 해당 메소드의 위치가 적절하지 않을 수도 있다. (기능 편애 “Feature Envy” 냄새에 해당한다.)

### 적용
1. 객체에서 get으로 꺼내는게 아니라(obj.getter()) 객체를 그대로(obj)

### 고민할문제
1. 함수를 일반화된 데이터 Map<K, V>를 param으로 받는게 맞는가, 아니면 위의 obj를 받는게 맞는가.
=
다른곳에서도 사용하는가? 이함수를 다른 도메인ㅇ에도 적용할것인가?에 따라 그냥 놔둘수 있다.
= 
과연이위치가 적절한가? 다른 도메인에도 사용할것인가? ( Refactor => Move Method )
만약 밖으로 추출(util성)한다면 Normalize할 필요가 있다.

## 리펙도링 10. 함수를 명령으로 바꾸기
### 고찰
함수를 독립적인 객체인, Command로 만들어 사용할 수 있다. 
커맨드 패턴을 적용하면 다음과 같은 장점을 취할 수 있다.( 참고 : https://gmlwjd9405.github.io/2018/07/07/command-pattern.html )
- 부가적인 기능으로 undo 기능을 만들 수도 있다.
- 복잡한 기능을 구현해야한다면.....더 복잡한 기능을 구현하는데 필요한 여러 메소드를 추가할 수 있다. 상속이나 템플릿을 활용할 수도 있다.(check)
- 복잡한 메소드를 여러 메소드나 필드를 활용해 쪼갤 수도 있다.
- 대부분의 경우에 “커맨드” 보다는 “함수”를 사용하지만, 커맨드 말고 다른 방법이 없는 경우에만 사용 한다.

### 적용

## 리팩토링 11. 조건문 분해하기

###고찰
조건문이 복잡해지는 경우 조건 자체가 복잡한경우, true or false가 해야할일이 길어질경우 고려하는 것이다.

여러 조건에 따라 달라지는 코드를 작성하다보면 종종 긴 함수가 만들어지는 것을 목격할 수 있다.
“조건”과 “액션” 모두 “의도”를 표현해야한다.
기술적으로는 “함수 추출하기”와 동일한 리팩토링이지만 의도만 다를 뿐이다.

### 적용
if( A ) else (B) => A ? B : C
decompose conditional을  통해서 명확하게 함수이름을 통해서 어떤 동작을하고싶은지 가독성이 높아진다.


## 리팩토링 12. 반복문 쪼개기

### 고찰
코드를 수정할때 반복문이 길면 condition을 고려해야함.
각각의 반복문마다 쪼개놓고 정리하자
O(N)의 입장에서도 N or N^2이냐는 다른문제이다.
하나의 반복문에서 여러 다른 작업을 하는 코드를 쉽게 찾아볼 수 있다. 해당 반복문을 수정할 때 여러 작업을 모두 고려하며 코딩을 해야한다. 반복문을 여러개로 쪼개면 보다 쉽게 이해하고 수정할 수 있다.
성능 문제를 야기할 수 있지만, “리팩토링”은 “성능 최적화”와 별개의 작업이다. 리팩토링을 마친 이후에 성능 최적화 시도할 수 있다.

