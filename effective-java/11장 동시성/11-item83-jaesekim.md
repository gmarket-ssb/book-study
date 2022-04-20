# 지연 초기화는 신중히 사용하라

(다른 수많은 최적화와 마찬가지로) 실제로는 성능을 느려지게 할 수도 있고..

멀티쓰레드 환경에서는 지연 초기화를 하기가 까다롭다. 지연 초기화하는 필드를 둘 이상의 쓰레드가 공유한다면 어떤 형태로든 반드시 동기화해야 한다. 그렇지 않으면 심각한 버그로 이어질 것이다.

## 쓰레드 세이프한 초기화 기법

1) 인스턴스를 초기화하는 일반적인 방법 (eager initialization)

```java
private final FieldType field = computeFieldValue();
```

2) 인스턴스 필드의 필드의 지연 초기화 - synchronized 접근자 방식

    지연 초기화가 초기화 순환성(initialization circularity)을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용하자.

    ```java
    private final FieldType field;
    
    private synchronized FieldType getField() {
        if (field == null) {
            field = computeFieldValue();
        }
        return field
    }
    ```

3) 정적 필드용 지연 초기화 홀더 클래스 관용구

    성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스(lazy initialization holder class) 관용구를 사용하자.

    ```java
    // inner class
    private static class FieldHolder {
        static final FieldType field = computeFieldValue();
    }
    
    private static FieldType getField() {
        return FieldHolder.field;
    }
    ```

    동작방식: 클래스는 클래스가 처음 쓰일 때 비로소 초기화된다는 특성을 이용한 관용구다. getField가 처음 호출되는 순간 FieldHolder.field가 처음 읽히면서, 비로소 FieldHolder 클래스 초기화를 촉발한다. 이 관용구의 멋진 점은 getField 메서드가 필드에 접근하면서 동기화를 전혀 하지 않으니 성능이 느려질 거리가 전혀 없다는 것이다. 일반적인 VM은 오직 클래스를 초기화할 때만 필드 접근을 동기화할 것이다. 클래스가 끝난 후에는 VM이 동기화 코드를 제거하여, 그 다음부터는 아무런 검사나 동기화 없이 필드에 접근하게 된다.

4) 인스턴스 필드 지연 초기화용 이중검사 관용구

    성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사(double-check) 관용구를 사용하라. 이 관용구는 초기화된 필드에 접근할 때의 동기화 비용을 없애준다.
    
    ```java
    private volatile FieldType field;
    
    private FieldType getField() {
        // 왜 result라는 지역 변수를 사용할까?
        // 이미 초기화된 상황에서 딱 한번만 읽도록 보장하는 역할을 한다.
        // 아마..field는 volatile이기 때문에 동기화 비용이 든다(메인메모리에서 직접 가져오기 때문에)
        FieldType result = field;
        if (result != null) { // 첫 번째 검사 (락 사용 안함)
            return result; // 이때 한번 더 안읽어도 됨
        }
    
        synchronized(this) {
            if (field == null) {
                field = computeFieldValue();
            }
            return field;
        }
    }
    ```

    이중검사에는 언급해둘만한 두가지 변종이 있다.

   - 단일 검사(single-check) 관용구

     이따금 초기화해도 상관없는 인스턴스필드를 지연초기화해야 할 때가 있는데, 이런 경우라면 이중검사에서 두 번째 검사를 생략할 수 있다.

       ```java
       // 여전히 volatile을 사용해야 한다. 
       private volatile FieldType field;
    
       private FieldType getField() {
           FieldType result = field; // 이중검사 관용구와 같은 이유로 로컬변수를 사용한 것 같음
           if (result == null) {
               result = computeFieldValue();
               field = result;
           }
           return result;
       }
       ```

   - 짜릿한 단일 검사(racy single-check) 관용구  
     모든 쓰레드가 필드의 값을 다시 계산해도 상관없고 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 단일 검사의 필드선언에서 volatile 한정자를 없애도 된다(아이템 78 참고). 이 관용구는 어떤 환경에서는 필드 접근 속도를 높여주지만, 초기화가 쓰레드 당 최대 한 번 더 이뤄질 수 있다. 아주 이례적인 기법으로 보통은 거의 쓰지 않는다.

## 핵심정리

- 대부분의 필드는 지연시키지 말고 곧바로 초기화해야 한다. (아이템 67 참고)
- 성능 떄문에 혹은 위험한 초기화 순환을 막기 위해 꼭 지연 초기화를 써야 한다면 올바른 초기화 기법을 사용하자.
    - 인스턴스 필드
        - 이중검사관용구
        - 반복해 초기화해도 괜찮은 경우 단일검사 관용구도 고려
    - 정적필드 - 지연 초기화 홀더 관용구