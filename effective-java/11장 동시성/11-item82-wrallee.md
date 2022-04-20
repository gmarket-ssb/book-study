## 스레드 안정성 수준을 문서화하라

클래스/메서드를 구현 할 때는 멀티스레드 환경에서 어떻게 동작하는지 스레드 안전성 수준을 문서화하는것이 좋다. 그렇지 않는다면 클라이언트는 클래스를 가정을하고 분석을 해야하며, 때로는 틀린 가정으로 인해 동기화를 충분히 못하게 되거나, 지나치게 하여 심각한 오류를 유발할 수도 있다.

<br/>

스레드 안정성을 표현하는 방식은 다음과 같다.

- 불변(immutable): 상수와 같이 불변하기 때문에 동기화조차 필요 없음.
- 무조건적 스레드 안전(unconditionally thread-safe): 인스턴스는 가변일 수 있지만 내부에서 충분히 동기화가 잘 되어있음.
- 조건부 스레드 안전(conditionally thread-safe): 일부 메서드는 외부 동기화가 필요하기도 함.
- 스레드 안전하지 않음(not thread-safe): 이 클래스의 메서드는 동기화 처리를 해 주어야 thread-safe하다.
- 스레드 적대적(thread-hostile): 이 클래스는 동기화 처리를 하더라도 thread-safe를 보장할 수 없다.

나머지는 대부분 이해 되겠지만 조건부 스레드 안전 부분만 추가 설명하자면 대표적으로 Collections.synchronizedList(...) 를 들여다 보면 다음과 같은 문서가 존재한다. 주의사항을 요약하면 List는 thread-safe 하지만, Iterator, Spliterator, Stream 같은 항목을 통해 탐색할때는 별도의 동기화 처리를 해야한다는 말이다. forEach 메서드를 호출할때도 동기화해야 한다.

```java
    /**
     * Returns a synchronized (thread-safe) list backed by the specified
     * list.  In order to guarantee serial access, it is critical that
     * <strong>all</strong> access to the backing list is accomplished
     * through the returned list.<p>
     *
     * It is imperative that the user manually synchronize on the returned
     * list when traversing it via {@link Iterator}, {@link Spliterator}
     * or {@link Stream}:
     * <pre>
     *  List list = Collections.synchronizedList(new ArrayList());
     *      ...
     *  synchronized (list) {
     *      Iterator i = list.iterator(); // Must be in synchronized block
     *      while (i.hasNext())
     *          foo(i.next());
     *  }
     * </pre>
     * ...
     */
    public static <T> List<T> synchronizedList(List<T> list) {
        ...
    }
```

추가참고: https://www.baeldung.com/java-synchronized-collections#the-synchronizedList-method

<br/>

또한 문서 대신 JCIP Annotation을 활용하여 표기할 수도 있다. @GuardedBy, @Immutable, @NotThreadSafe, @ThreadSafe 가 존재하며, Apache HttpClient의 HttpGet, DefaultHttpClient 클래스 등에서 JCIP Annotation을 활용하고 있다. 애너테이션을 활용하면 스레드 안정성을 프로그래밍적으로 확인할 수 있어 버그 자동화 툴을 적용하거나 동기화 필요 여부를 컴파일 단계에서 파악하는 등의 방향으로도 활용할 수 있을 것이다.

<br/>

#### 결론

- 모든 클래스가 자신의 스레드 안전성 정보를 명확히 문서화해야 한다.
- 정확한 언어로 명확히 설명하거나 스레드 안전성 에너테이션을 사용할 수 있다.
- 조건부 스레드 안전 클래스는 메서드를 어떤 순서로 호출할 때 외부 동기화가 요구되고, 그때 어떤 락을 얻어야 하는지도 알려줘야 한다.