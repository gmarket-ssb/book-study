## [item73] 추상화 수준에 맞는 예외를 던지라

### 예외 번역

> 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 던져야 한다 (예외번역 exception translation)<br>
> 저수준 예외를 처리하지 않고 잘못 전파하면, 윗 레벨 API를 오염시킬 수 있다.

```java
try {

} catch (LowerLevelException e) {
    throw new HigherLevelException(...);
}
```

AbstractSequentialList
```java
public E get(int index) {
    ListIterator<E> i = listIterator(index);
    try {
        return i.next();
    } catch (NoSuchElementException e) {
        throw new IndexOutOfBoundsException("index: " + index);
    }
}
```

### 예외 연쇄
> 예외 연쇄란 문제의 근분인 원인(cause)인 저수준 예외를 고수준 예외에 실어 보내는 방식

```java
try {

} catch (LowerLevelException cause) {
    throw new HigherLevelException(cause);
}
```

예외 연쇄용 생성자
```java
class HigherLevelException extends Exception {
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}
```


### 예외번역을 남용하지 말자
무턱대고 예외를 전파하는 것보다야 예외 번역이 우수한 방법이지만, 그렇다고 남용해서는 곤란하다.
1) 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선
2) 아래 계층에서의 예외를 피할 수 없다면, 상위 계층에서 그 예외를 조용히 처리하여 문제를 API 클라이언트에게 전파하지 않는 방법
   (이럴땐 따로 로깅을 활용해서 기록해두면 좋다)

--- 

## 핵심 정리

  아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 힘들다면 예외 번역을 사용하라.<br>
  이때 예외 연쇄를 이용하면, 상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기에 좋다.
