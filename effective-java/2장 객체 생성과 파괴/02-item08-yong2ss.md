## [Item8] finalizer와 cleaner 사용을 피하라 

java에서는 두 가지의 객체 소멸자를 제공한다.
> 1. Object 클래스에 포함된 finalize 메서드
> 2. Java 9에서 추가된 java.lang.ref 패키지에 포함된 Cleanner 클래스

이 두 가지는 JVM에서 가비지 콜렉터가 이루어질 때 실행되는 구문이다.<br>
하지만 Finallizer는 예측 불가능하고 위험하며, 불필요하다. 또한 자바 9부터는 Deprecated되었다.<br>
또한 Cleanner는 별도의 스레드에서 동작하기 때문에 괜찮아 보이지만, 예측할 수 없고, 느리고, 사용이 권장되지 않는다.


### Finalizer와 Cleaner의 치명적인 단점
1) 신뢰성 이슈
    * **사용 했을 때 실행 시점을 보장할 수 없다. (언제 실행 될 지 모른다.)**<br>
      전적으로 GC 알고리즘에 달렸으며, 이는 GC 구현마다 천차만별이므로 JVM의 버전과 옵션에 따라 달라질 수 있다.
    * **인스턴스 반납을 지연 시킬 수도 있다.**<br>
      별도의 gc 쓰레드가 finalize를 실행시키는데, finalize는 우선순위가 낮아서 작업이 많으면 뒤로 미뤄질 수 있다. (자원반납이 늦어져 JVM에 객체가 쌓여서 out of memory exception이 일어날 수 있다.)
    * **실행이 되지 않을 수도 있다.**<br>
      회수해야 할 자원에 대한 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수도 있다는 얘기다.<br>
      따라서 데이터베이스 같은 공유 자원의 영구 락 해제와 같이 상태를 수정하는 작업에서는 절대 finalizer 나 cleaner에 의존하지 말고 직접 자원을 close해줘야 한다.


2) 성능 이슈 
    * finalizer와 cleaner는 가비지 컬렉터의 효율을 떨어뜨린다.<br> AutoCloseable 객체를 생성하고 close메서드를 사용하는 방법보다 약 50배가 느리다.
    

3) 보안 이슈
    * A라는 클래스가 있고 B는 A를 상속 받는다, B에서 인스턴스 생성시에 예외 던지고 finalize를 오버라이딩 한다면
인스턴스 생성 시 에러가 발생하면 예외를 던지고 죽어야 정상인데, 죽을 때 finalize가 실행이 된다. 이때 인스턴스가 갖고 있는 메서드나 스태틱 필드에 (사용)접근이 가능하다.

    
> **(Tip) Finalizer Attack을 방어하는 방법**
> * final 클래스로 만들어 상속하지 못하게 하거나<br>
> * finalize 메소드 자체를 final로 만들자. (아무일도 하지 않는 finalize 메서드를 만들고 final로 선언하자)
   

-------------------------------------

### finalizer와 cleaner의 대안! (java 8부터 사용가능) 
> 1. AutoCloseable 인터페이스를 implements하고 close를 오버라이딩해서 자원반납을 한다.
> 2. Item 9에서 나오는 try-with-resource를 사용

### Finalizer와 Cleaner를 사용해야 한다면...
**1. 안전망 역할**

자원반납을 해야되는 클래스는 원래 클라이언트(호출)하는 쪽에서 자원 반납 (ex:close())를 명시적으로 해줘야한다.<br>
하지만 클라이언트가 자원반납에 쓸 close를 호출하지 않았다는 가정하에는, Finalizer나 Cleaner가에서 안전망으로 close를 한 번 더 호출할 수 있다.<br>
예를들어 자바에서 FileInputStream, FileOutputStream, ThreadPoolExecutor, java.sql.Connection에는 안전망으로 동작하는 finalizer(finalizer안에서 자원을 반납하는)가 코딩되어 있다고 한다.

* 예시 (AutoCloseable와 안전망을 이용한 방법)
```java
public class SampleClass implements AutoCloseable {

    private boolean closed;

    @Override
    public void close() throws RuntimeException {
        if (this.closed) {
            throw new IllegalStateException();
        }
        closed = true;
    }

    @Override
    protected void finalize() throws Throwable {
        if (!this.closed)
            close();
    }
}
```

**2. 네이티브 자원을 정리할때 쓴다**

네이티브 객체는 일반적인 객체가 아니라서 GC가 그 존재를 모른다.
중요하지 않거나 영향이 크지 않으면 Cleaner가 Finalizer를 사용해서 해당 자원을 반납해도 되지만, 중요하다면 close 메소드를 사용하는 것이 좋다.

### * 결론 
> 되도록 AutoCloseable을 구현하고 try-with-resource을 사용 !!!

