# 명명 패턴보다 애너테이션을 사용하라

전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다. 예컨대 테스트 프레임워크인 JUnit은 버전 3까지 테스트 메서드 이름을 test로 시작하게끔 했다. 하지만 아래와 같은 단점들이 있다.

- 오타가 나면 안된다.
- 올바른 요소에만 사용되리라 보증할 방법이 없다(메소드, 클래스 등등..)
- 프로그램 요소를 매개변수로 전달할 방법이 없다.

## 애너테이션

애너테이션은 이런 모든 문제들을 해결해준다.

1. 이름에 오타를 내거나 메서드 선언 외의 프로그램 요소에 달면 컴파일 오류를 내준다.

    ```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface Test {
    }
    ```

2. 메타애너테이션을 통해 애너테이션이 언제까지 유지될지 어떤 element type에 달 수 있는지 알려줄 수 있다.
   - `@Retention`: [https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Retention.html](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Retention.html)
   - `@Target` : [https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Target.html](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Target.html)
3. 애너테이션에 매개변수를 사용할 수 있다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

@ExceptionTest(ArithmeticException.class)
public static void m1() {
    int i = 0;
    i = i / i;
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest2 {
    Class<? extends Throwable>[] value();
}

@ExceptionTest2({IndexOutOfBoundsException.class, NullPointerException.class})
public static void doublyBad() {
    List<String> list = new ArrayList<>();
    list.addAll(5, null);
}
```

java 8 이상부터는 `@Repeatable` 메타애너테이션을 사용하여 여러 개의 값을 받는 애너테이션을 정의할 수 있다. 이 방식으로 가독성이 개선할 수 있다면 사용하자. 하지만 애너테이션을 선언하고 처리하는 부분의 코드 양이 늘어나며, 특히 코드가 복잡해져 오류가 날 가능성이 커진다.

```java
// 컨테이너 애노테이션
// @Repeatable을 사용하는 애너테이션보다 @Retention 및 @Target이 더 커야됨
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest3[] value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest3 {
    Class<? extends Throwable> value();
}

@ExceptionTest3(IndexOutOfBoundsException.class)
@ExceptionTest3(NullPointerException.class)
public static void doublyBad() {
    List<String> list = new ArrayList<>();
    list.addAll(5, null);
}

public class RunExceptionTests3 {

    public static void main(String[] args) throws ClassNotFoundException {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName("chapter6.item39.ExceptionTestSample3");

        for (Method m : testClass.getDeclaredMethods()) {
            // ExceptionTest3가 한개만 있을 때 || 여러개 일떄
            if (m.isAnnotationPresent(ExceptionTest3.class) || m.isAnnotationPresent(ExceptionTestContainer.class)) {
                tests++;

                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외 던지지 않음%n", m);
                } catch (InvocationTargetException e) {
                    Throwable throwable = e.getCause();
                    int oldPassed = passed;
                    ExceptionTest3[] exceptionTest3s = m.getAnnotationsByType(ExceptionTest3.class);

                    for (ExceptionTest3 exceptionTest3 : exceptionTest3s) {
                        if (exceptionTest3.value().isInstance(throwable)) {
                            passed++;
                            break;
                        }
                    }

                    if (passed == oldPassed) {
                        System.out.printf("테스트 %s 실패: %s%n", m, throwable);
                    }
                } catch (Exception e) {
                    System.out.println("잘못 사용한 @ExceptionTest3: " + m);
                }
            }
        }

        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
```