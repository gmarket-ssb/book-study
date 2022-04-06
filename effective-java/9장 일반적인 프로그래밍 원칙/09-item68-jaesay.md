# 일반적으로 통용되는 명명 규칙을 따르라.

자바 플랫폼은 명명 규칙이 잘 정립되어 있으며, 그 중 많은 것이 자바 언어 명세 ([JLS, 6.1](https://docs.oracle.com/javase/specs/jls/se7/html/jls-6.html)) 에 기술되어 있다. 자바의 명명규칙은 크게 철자와 문법, 두 범주로 나뉜다. 철자 규칙이나 문법 규칙을 어기면 다른 프로그래머 들이 그 코드를 읽기 번거로울 뿐 아니라 다른 뜻으로 오해할 수도 있고 그로 인해 오류까지 발생할 수 있다.

## 철자 규칙

철자 규칙은 패키지, 클래스, 인터페이스, 메서드, 필드, 타입 변수의 이름을 다룬다. 이 규칙들은 특별한 이유가 없는 한 반드시 따라야 한다. 이 규칙을 어긴 API는 사용하기 어렵고, 유지보수하기 어렵다.

### 패키지와 모듈

패키지와 모듈 이름은 각 요소를 점(.)으로 구분하여 계층적으로 짓는다. 요소들은 모두 소문자 알파벳 혹은 (드물게) 숫자로 이뤄진다.  
⇒ org.junit.jupiter.api, com.google.common.collect

각 요소는 일반적으로 8자 이하의 짧은 단어로 한다.  
⇒ utilities 보다는 util 처럼 의미가 통하는 약어 추천

### 클래스와 인터페이스

하나 이상의 단어로 이뤄지며, 각 단어는 대문자로 시작한다.  
⇒ Stream, FutureTask

여러 다어의 첫 글자만 딴 약자나 max, min처럼 널리 통용되는 줄임말을 제외하고는 단어를 줄여쓰지 않도록 한다.

약자의 경우 첫글자만 대문자로 하는 경우가 많다.  
⇒ HttpUrl

### 메서드와 필드

메서드와 필드 이름은 첫 글자를 소문자로 쓴다는 점만 빼면 클래스 명명 규칙과 같다.  
⇒ remove, ensureCapacity

### 상수 필드

상수 필드를 구성하는 단어는 모두 대문자로 쓰며 단어 사이는 밑줄로 구분한다.  
⇒ VALUE, NEGATIVE_INFINITY

### 지역 변수

지역변수에도 다른 맴버와 비슷한 명명 규칙이 적용되지만 문맥에서 의미를 쉽게 유추할 수 있기 때문에 약어를 써도 좋다.  
⇒ i, denom, houseNum

입력 매개변수도 지역변수의 하나다. 하지만 메서드 설명 문서에까지 등장하는 만큼 일반 지역변수보다는 신경을 써야 한다.

### 타입 매개변수

타입매개변수 이름은 보통 한 문자로 표현한다. 대부분은 다음의 다섯 가지 중 하나다. 임의의 타입엔 T(Type), 컬렉션 원소의 타입은 E(Element)를, 맵의 키와 값에는 K(Key)와 V(Value)를, 예외에는 X(eXception)를, 메서드 반환 타입에는 R(Return)을 사용한다. 그 외에 임의 타입의 시퀀스에는 T, U, V 혹은 T1, T2, T3를 사용한다.

## 문법 규칙

문법 규칙은 철자 규칙과 비교하면 더 유연하고 논란도 많다. 패키지에 대한 규칙은 따로 없다.

### 클래스(열거타입 포함)

보통 단수 명사나 명사구를 사용한다.  
⇒ Thread, PriorityQueue, ChessPiece

객체를 생성할 수 없는 클래스의 이름은 보통 복수형 명사로 짓는다.  
⇒ Collectors, Collections

인터페이스 이름은 클래스와 똑같이 짓거나 able 혹은 ible로 끝나는 형용사로 짓는다.  
⇒ Collection, Comparator  
⇒ Runnable, Iterable, Accessible

애너테이션은 워낙 다양하게 활용되어 지배적인 규칙이 없이 명사, 동사, 전치사 형용사가 두루 쓰인다.  
⇒ BindingAnnotation, Inject, ImplementedBy, Singleton

### 메서드

메서드의 이름은 동사나 (목적어를 포함한) 동사구로 짓는다.  
⇒ append, drawImage

boolean 값을 반환하는 메서드라면 보통 is나 has로 시작하고명사나 명사구, 혹은 형용사로 가능한 아무 단어나 구로 끝나도록 짓는다.  
⇒ isDigit, isProbablePrime, isEmpty, isEnabled, hasSiblings

반환 타입이 boolean이 아니거나 해당 인스턴스의 속성을 반환하는 메서드의 이름은 보통 명사, 명사구, 혹은 get으로 시작하는 동사구로 짓는다.  
⇒ size, hashCode, getTime

자바빈즈 명명규칙을 따르거나 클래스가 한속성의 게터와 세터를 모두 제공할 때는 get/set으로 시작하는 형태를 쓴다.  
⇒ getAttribute, setAttribute

객체의 타입을 바꿔서, 또 다른 객체를 반환하는 인스턴스 메서드의 이름은 보통 toType 형태로 짓는다.  
⇒ toString, toArray

객체의 내용을 다른 뷰로 보여주는 메서드의 이름은 asType 형태로 짓는다.  
⇒ asList

객체의 값을 기본 타입 값으로 반환하는 메서드의 이름은 보통 typeValue 형태로 짓는다.  
⇒ intValue

정적팩터리의 이름은 다양하지만 from, of, valueOf, instance, getInstance, newInstance, getType, newType을 흔히 사용한다(아이템 1 참고).

### 필드 이름

boolean 타입의 필드 이름은 보통 boolean 접근자 메서드에서 앞 단어를 뺀 형태이다.  
⇒ initialized, composite

boolean 이 아닌 다른 필드라면 명사나 명사구를 사용한다.  
⇒ height, digits, bodyStyle

### 지역변수

필드와 비슷하게 지으면 되나 조금 더 느슨하다.

## 핵심 정리

- 표준 명명 규칙을 체화하여 자연스럽게 베어 나오도록 하자.
- 오랫동안 따라온 규칙과 충돌한다면 그 규칙을 맹종해서는 안된다.