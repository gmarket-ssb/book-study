## 표준 예외를 사용하라

코드를 재사용하듯 예외도 재사용하는것이 좋다. 그리고 자바 라이브러리는 대부분 API에서 쓰기에 충분한 수의 예외를 제공한다.

아래 표는 대표적으로 자주 사용되는 표준 예외이다.

| 예외                            | 주요 쓰임                                                    |
| ------------------------------- | ------------------------------------------------------------ |
| IllegalArgumentException        | 허용하지 않는 값이 인수로 건네졌을 때(null은 따로 NullPointerException으로 처리) |
| IllegalStateException           | 객체가 메서드를 수행하기 적절하지 않은 상태일 때             |
| NullPointerException            | null을 허용하지 않는 메서드에 null을 건넷을 때               |
| IndexOutOfBoundsException       | 인덱스가 범위를 넘어섰을 때                                  |
| ConcurrentModificationException | 허용하지 않는 동시 수정이 발견됐을 때                        |
| UnsupportedOperationException   | 호출한 메서드를 지원하지 않을 때                             |

이외에도 복소수/유리수를 다룰 때 ArithmeticException, NumberFormatException 같은 예외도 재사용 할 수 있을 것이다.

주의 할 점은 **Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말자.** 이 예외들은 여러 예외를 포괄한다는 개념으로 추상 클래스라고 생각하는게 좋다.

또한 API 문서를 참고해 그 예외가 어떤 상황에서 던져지는지 꼭 확인하고 사용하자. 예외의 이름뿐 아니라 예외가 던져지는 맥락도 부합할 때만 재사용한다.

흔히 IllegalArgumentException과 IllegalStateException을 선택하기 어려울 때가 있는데, 인수 값이 무엇이었든 어차피 실패했을 거라면 IllegalStateException을, 그렇지 않다면 IllegalArgumentException을 사용하자.

더 많은 정보를 제공하길 원한다면 표준 예외를 확장해도 좋다. 단, 예외는 직렬화 할 수 있다는 사실을 기억하자. 이 사실만으로도 나만의 예외를 새로 만들지 않아야 할 근거로 충분하다.

<br/>

### 요약

1. 표준 예외를 재사용하자.
2. API 문서를 참고해 예외를 던지는 맥락을 이해하고 사용하자.
3. 표준예외를 확장해도 좋다. 단, 예외 직렬화 비용을 고려하자.
