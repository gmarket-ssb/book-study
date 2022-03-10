## 가변인수는 신중히 사용하라

가변인수 메서드는 명시한 타입의 **인수를 0개 이상** 받을 수 있다.  
또한 가변인수 메서드는 런타임에 **배열을 자동 생성**하여 사용한다.

이러한 배경에 따라 고려해야 할 점이 몇 가지 있는데 아래 두 케이스에 대해 다뤄보자.

- 반드시 1개 이상의 원소를 받아야 하는 경우
- 배열을 생성하는 동작이 성능에 영향을 끼칠 때



#### 1) 반드시 1개 이상의 원소를 받아야 하는 경우

아래와 같이 런타임에서 체크하도록 구현할 수 있으나, 좋은 방법은 아니다.

```java
// The WRONG way to use varargs to pass one or more arguments! (Page 245)
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("Too few arguments");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
```

다음과 같이 첫 번째로는 일반 변수를 받도록 수정하자.  
컴파일타임에서 체크가 될 뿐만 아니라 for-each문까지 사용할 수 있게 된다.

```java
// The right way to use varargs to pass one or more arguments (Page 246)
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
```

<br>

#### 2) 배열을 생성하는 동작이 성능에 영향을 끼칠 때

가변인수 메서드는 호출 될 때마다 배열을 할당하고 초기화 한다.  
만약 어떤 가변인수 메서드의 호출 케이스의 95%가 인수를 3개 이하로 받는다고 할 때 다음과 같이 정의하자.

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

이렇게 정의하면 단 5%의 케이스만이 5번째 메서드를 호출하게 되고 이 경우에만 배열이 생성된다.  
코드는 좀 길어지겠지만 특수한 상황에선 `foo(int... rest)` 하나만 만드는 것 보다 효율적으로 메모리를 활용하게 된다.



### 결론

가변인수는 <ins>변수를 0개부터 받을 수 있고</ins>, <ins>호출 때마다 배열을 생성한다</ins>는점에 유의하여 사용하자.
