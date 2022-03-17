## [item 57] 지역변수의 범위를 최소화하라

지역변수의 유효 범위를 최소로 줄이면
1. 코드 가독성과 유지보수성이 높아짐
2. 오류 가능성은 낮아짐

### 지역변수의 범위를 줄이는 가장 강력한 기법
1. 가장 처음 쓰일 때 선언하기  
사용하려면 멀었는데, 미리 선언부터 해두면 코드가 어수선해져 가독성이 떨어짐

2. 거의 모든 지역변수는 선언과 동시에 초기화 해야 함
try-catch 문은 이 규칙에서 예외  
-> try 블록 바깥에서도 사용해야한다면 try 블록 앞에서 초기화

3. 반복 변수의 값을 반복문이 종료된 후에도 써야하는 것이 아니면 while 대신 for 사용
for, for-each 모두 반복 변수의 범위가 for 키워드와 몸체 사이로 제한 됨

```java
// 57-1 컬렉션이나 배열을 순회하는 권장 관용구
for (Element e : c) {
    ... // e로 무언가를 한다
}
```
반복문의 인덱스가 필요한 상황에선 전통적인 for문을 사용하는 것이 낫다.

```java
// 57-2 반복자가 필요할 때의 관용구
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // e와 i로 무언가를 한다
}
```

```java
// 복사 붙여 넣기 오류 - 버그가 숨어 있는 while 문
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
    doSomething(i.next());
}
        ...

Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) { // 버그: 이전 i 사용
    doSomethingElse(i2.next());
}
```
i의 유효범위가 끝나지 않아 컴파일도 잘되고 실행시 예외도 없음

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ...
}

// i를 찾을 수 없다 컴파일 오류
for (Iterator<Element> i2 = c2.iterator(); i.hasNext(); ) {
    Element e2 = i2.next();
    ...
}
```
for 문을 사용 할 경우 복사 붙여 넣기 오류를 컴파일 타임에 잡을 수 있음  
게다가 변수 유효범위가 for문의 범위아 일치하여 똑같은 이름의 변수를 여러 반복문에서 써도 영향을 주지 않음   

```java
// 지역변수의 범위를 최소화 하는 관용구
for (int i = 0, n = expensiveComputation(); i < n; i++) {
    ...
}
```
변수 i의 한계값을 변수 n에 저장하여, 반복때마다 n을 계산해야 하는 비용을 없앴음   
같은 값을 반환하는 메서드를 매번 호출한다면 이 관용구를 사용

4. 메서드를 작게 유지하고 한가지 기능에 집중
한 메서드에서 여러가지 기능을 처리한다면 그 중 한 기능만 관련된 지역변수라도   
다른 기능을 수행하는 코드에서 접근할 수 있음   
해결책 -> 기능별로 쪼개자


### 핵심 정리
- 아이템 15장(클래스와 멤버의 접근 권한을 최소화하라)와 취지가 비슷하다
- 지역변수의 유효범위를 최소한으로 줄이자 - 가독성, 유지보수성 UP, 오류 가능성 DOWN
  - 가장 처음 쓰일 때 선언하기
  - 선언과 동시에 초기화 해야 함 (try-catch는 예외)
  - while 대신 for 사용
  - 메서드를 작게 유지하고 한가지 기능에 집중