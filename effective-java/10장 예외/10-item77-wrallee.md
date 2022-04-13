## 예외를 무시하지 말라

결론: 아래와 같은 `catch` 블록을 만들지 말자.

```java
try {
    ...
} catch (SomeException e) {
}
```

너무 뻔한 조언이지만 사람들이 정말 자주 어기는 항목이라고 한다. 검사 예외든, 비검사 예외든 절대 위 같은 블록을 만들지 말자.

물론 무시해도 되는 상황도 존재한다. `FileInputStream`을 `close()` 하는 상황이 그 예다. 입력 전용 스트림이므로 따로 롤백할거리도 없고, 스트림을 명시적으로 닫는 것 자체가 일을 다 끝냈다는 뜻이니 이후 예외가 발생하더라도 파일을 닫지 못했다 정도의 문제이다.

<br/>

예외를 무시하기로 했다면 `catch` 블록 안에 주석으로 그렇게 결정한 이유를 남기고 변수의 이름을 `ignored`로 바꿔놓도록 하자.

 ```java
 Future<Integer> f = exec.submit(planarMap::chromaticNumber);
 int numColor = 4; // 기본값. 어떤 지도라도 이 값이면 충분하다.
 try {
     numColor = f.get(1L, TimeUnit.SECONDS);
 } catch (TimeoutException | ExecutionException ignored) {
     // 기본값을 사용한다(색상 수를 최소화하면 좋지만, 필수는 아니다).
 }
 ```

<br/>

예외를 무시하지 말고 바깥으로 전파되게만 해도 디버깅 정보를 남긴 채 프로그램이 중단되게 할 수 있다.  
`ControllerAdvice`, `ExceptionHandler` 등을 활용하여 에러를 잘 처리하자. 

하지만 예외 핸들러에서 로깅을 하지 않고 무작정 200만 응답하는것도 사실상 빈 `catch` 블록을 만드는셈이니 주의하자.  
로그는 찍되, 심각성에 따라 `logger level` 을 나누고 응답코드도 적절하게 세팅하자.

[참고](https://github.com/wrallee/jwp-refactoring/blob/wrallee/module-web/src/main/java/kitchenpos/common/GeneralExceptionHandler.java)

<br/>

---

이 아이템을 보니 문득 예전에 다뤘던 시스템이 생각납니다. `dispatcher-servlet.xml` 에 아래 처럼 설정이 되어 있었는데 예외 발생 시 무조건 `error.jsp`로 보내고 로깅도 안하도록 되어있어서 고생을 했었습니다. 형태는 조금 다르지만(?) 하고싶은 말은 `ControllerAdvice`, `ExceptionHandler` 도 꼼꼼하게 잘 챙겨주자는 것 입니다.

```xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="defaultErrorView" value="error/error"/>
    <property name="exceptionMappings">
        <props>
            <prop key="java.lang.Exception">error/error</prop>
            <prop key="org.springframework.dao.DataAccessException">error/dataAccessFailure</prop>
            <prop key="org.springframework.transaction.TransactionException">error/transactionFailure</prop>
        </props>
    </property>
</bean>
```
