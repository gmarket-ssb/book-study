## 비검사 경고를 제거하자
모든 비검사 경고를 제거한다면 그 코드는 타입 안정성이 보장된다. 즉, 런타임에 ClassCastException이 발생할 일 없고, 의도한 대로 잘 동작하리라 확신할 수 있다.

### 경고 예제와 해결방법
1. 컴파일러가 알려준 대로 수정한다.

    ```java
    Set<Lark> exaltation = new HashSet(); // warning: unchecked conversion
    Set<Lark> exaltation = new HashSet<>();
    ```

1. 경고를 제거할 수는 없지만 타입이 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchecked")`을 달아 경고를 숨긴다.
    - 가능한 범위를 좁혀 숨긴다.
    - 심각한 경고를 놓칠 수 있으니 절대로 클래스 전체에 적용해서는 안된다.
    - 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.

    ```java
   @Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.CONSTRUCTOR, ElementType.LOCAL_VARIABLE, ElementType.MODULE})
   @Retention(RetentionPolicy.SOURCE)
   public @interface SuppressWarnings {
       String[] value();
   }
   
    public <T> T[] toArray(T[] a) {
        if (a.length < size) {
            // 생성한 배열과 매개변수로 받는 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
            @SuppressWarnings("unchecked")
            T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
            return result;
        }
        // ...
    }
    ```

### 핵심 정리
할 수 있는 한 모든 비검사 경고를 제거하여 타입 안정성을 보장하자