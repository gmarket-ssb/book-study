## 클래스와 멤버의 접근 권한을 최소화하라
> 이 아이템에서는 '캡슐화' 와 '정보 은닉' 에 대해서 심도있게 다룹니다.
<br>

### 흔히 말하는 '잘 설계된' 컴포넌트란?
* 모든 내부 구현을 완벽히 숨겨, 구현과 API 를 깔끔히 분리한 컴포넌트.
* 오직 API 를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는 컴포넌트.

=> 이는, `캡슐화` 혹은 `정보 은닉` 이라고 하는 소프트웨어 설계의 근간이 되는 개념 원리
<br><br>

### 미묘한 용어의 차이
사실 `캡슐화` 와 `정보 은닉` 은 미묘한 용어의 차이가 있습니다.
* 캡슐화 : 데이터와 함수를 하나로 **묶는** 것
* 정보 은닉 : 다른 객체에게 자신의 정보를 **숨기고** 자신의 연산만을 통하여 접근을 **허용**하는 것이며, 캡슐화에서 가장 중요한 장점
<br>

### 정보 은닉의 수많은 장점
* 시스템 개발 속도를 높인다.
* 시스템 관리 비용을 낮춘다.
* 성능 최적화에 도움을 준다.
* 소프트웨어 재사용성을 높인다.
* 큰 시스템을 제작하는 난이도를 낮춰준다.
<br><br>

### 자바에서 정보 은닉을 취하는 방법
자바에서 정보 은닉을 위해 제공하는 다양한 장치 중에는 `접근 제어 메커니즘` 이 있다.<br>
이 메커니즘은 **"모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다."** 는 간단한 기본 원칙을 가지고 있다.

![image](https://user-images.githubusercontent.com/57446639/150311833-cf620d2b-648f-493d-b80c-79e09109766b.png)

* 멤버(필드, 메서드, 중첩 클래스, 중첩 인터페이스)에 부여하는 접근 수준
  * `private` : (내부 접근) 멤버를 선언한 클래스에서만 접근할 수 있다.
  * `package-private` : (패키지 접근) 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있다.
  * `protected` : (패키지+하위클래스 접근) default + 멤버를 선언한 클래스의 하위 클래스에서도 접근할 수 있다.
  * `public` : 모든 곳에서 접근할 수 있다.


* 탑클래스(가장 바깥에 있는 클래스)와 인터페이스에 부여하는 접근 수준
  * `package-private` : 내부 구현이 되어 언제든 수정할 수 있다.
  * `public` : 공개 API 가 되어 영원히 관리해줘야만 한다.
<br><br>

### 권장하는 설계 순서
책에서는 아래와 같이 좁게 설계한 후 풀어주는 방식을 권장하고 있다.
1. 클래스의 공개 API 를 세심히 설계한다.
2. 모든 멤버는 `private` 로 만든다.
3. 필요에 따라 다른 클래스가 접근해야 하는 멤버에 한해 `package-private` 로 풀어준다.
4. ~~반복...~~
<br><br>

### 설계시 유의사항
* `Serializable` 을 구현한 클래스에서는 그 필드들도 의도치 않게 공개 API 가 될 수 있음에 유의하자.
  * (아이템 86 스포) 클래스가 `Serializable` 을 구현하면 직렬화 형태도 하나의 공개 API 가 된다.<br><br>
* public 클래스의 인스턴스 필드는 되도록 `public` 이 아니어야 한다.
  * (아이템 16 스포) public 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 방식을 마음대로 바꿀수 없게 된다.<br><br>
* 추상 개념을 완성하는 데 꼭 필요한 구성요소로써의 상수라면 `public static final` 필드로 공개해도 좋다.
  * (아이템 17 스포) 이렇게 하면 다음 릴리스에서 내부 표현을 바꾸지 못하므로 권장하지 않는다.
<br><br>

### 자바9 에서는...
* 패키지들의 묶음인 '모듈' 의 개념이 등장한다.
* 모듈은 자신이 속하는 패키지 중에 공개할 것들을 선언하는데, 공개(`protected`,`public`) 멤버라도 패키지를 공개하지 않는다면 모듈 외부에서는 접근할 수 없게 된다.
* 고로 이 방식은, 위에서 배운 캡슐화와 정보 은닉의 설정을 무력화할 수 있는 방법이다.
* 참고로, 새로 등장한 이 접근 수준을 적극 활용한 대표적인 예가 바로 "JDK" 자체이다.
* (tmi) 모듈의 장점을 제대로 누리려면 설정할게 많기도 하고 아직 이른감이 있다. 그러니 꼭 필요한 경우에만 사용하자.
<br><br>

### 요약
* 프로그램 요소의 접근성은 가능한 한 최소한으로 하라.
* 꼭 필요한 것만 골라 최소한의 공개 API 를 설계하자.
* 그 외에는 클래스, 인터페이스, 멤버가 의도치 않게 API 로 공개되는 일이 없도록 해야 한다.
* public 클래스는 상수용 `public static final` 필드 외에는 어떠한 public 필드를 가져서는 안되며, 해당 필드가 참조하는 객체가 불변인지 확인하라.
