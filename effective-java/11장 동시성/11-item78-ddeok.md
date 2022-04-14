## [item 78] 공유 중인 가변 데이터는 동기화해 사용하라

### synchronized 키워드
해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다.

### synchronized 사용 이유
**1. 배타적(독점적) 실행 보장**  
객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다.  
**2. 스레드 사이의 안정적인 통신을 보장**  
동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.

### 원자적인 데이터를 읽고 쓸 때는 동기화하지 않아도 되는가?
long과 double 외의 변수를 읽고 쓰는 동작은 원자적(atomic)이다.    
즉, 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 뜻이다.  
이 말을 듣고, "성능을 높이려면 원자적 데이터를 읽고 쓸 때는 동기화하지 말아야겠다"고 생각하기 쉬운데, 아주 위험한 발상이다!   
스레드가 필드를 읽을 때 항상 '수정이 완전히 반영된' 값을 얻는다고 보장하지만, 한 스레드가 저장한 값이 다른 스레드에게 '보이는가'는 보장하지 않는다.  
-> 저장된 값을 읽어오나 그게 원하는 스레드에서 읽어 온 값이 아닐 수 있다.    
-> 즉, 스레드를 사용할 경우 동기화를 해야한다

### 다른 스레드를 멈추는 올바른 방법방법
Thread.stop 메서드는 데이터 훼손으로 안전하지 않아 deprecated API로 지정 됨

다른 스레드를 멈추는 올바른 방법은  
첫 번째 스레드는 자신의 boolean 필드를 폴링하면서 그 값이 true가 되면 멈춘다.  
이 필드를 false로 초기화해놓고, 다른 스레드에서 이 스레드를 멈추고자 할 때 true로 변경하는 식이다.  
boolean 필드를 읽고 쓰는 작업은 원자적이라 어떤 프로그래머는 이런 필드에 접근할 때 동기화를 제거하기도 한다.

```java
// 잘못된 코드 - 이 프로그램은 얼마나 오래 실행될까?
public class StopThread {
    private static boolean stopRequested;
 
    public static void main(String[] args) throws InterruptedException{
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) i++;
        });
        backgroundThread.start();
 
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

이 프로그램이 1초 후에 종료되리라 생각하는가?  
메인 스레드가 1초 후 stopRequest를 true로 설정하면 backgroundThread는 반복문을 빠져나올 것처럼 보일 것이다.  
하지만 내 컴퓨터에서는 도통 끝날 줄 모르고 영원히 수행되었다.

원인은 동기화에 있다.  
동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 될지 보증할 수 없다.  
동기화가 빠지면 가상머신이 다음과 같은 최적화를 수행할 수도 있는 것이다.   


```java
// 원래 코드
while(!stopRequested)
  i++;

// 최적화한 코드
if(!stopRequested)
  while(true)
      i++;
```

OpenJDK 서버 VM이 실제로 적용하는 끌어올리기(hoisting)라는 최적화 기법이다.  
이 결과 프로그램은 응답 불가(liveness failure) 상태가 되어 더 이상 진전이 없다.  
stopRequested 필드를 동기화해 접근하면 이 문제를 해결할 수 있다.  
그래서 다음처럼 바꾸면 기대한 대로 1초 후에 종료된다.

*hoisting 최적화: 이 루프내에서 행할 필요가 연산들은 루프밖으로 끄집어 내는 최적화다.
참고: https://zbvs.tistory.com/25 [블로그]


```java
public class SyncStopThread {
   private static boolean stopRequested;

   private static synchronized void requestStop() {
      stopRequested = true;
   }

   private static synchronized boolean stopRequested() {
      return stopRequested;
   }

   public static void main(String[] args) throws InterruptedException {
      Thread backgroundThread = new Thread(() -> {
         int i = 0;
         while (!stopRequested()) i++;
      });
      backgroundThread.start();
      TimeUnit.SECONDS.sleep(1);
      requestStop();
   }
}
```


### 쓰기 메서드(requestStop)와 읽기 메서드(stopReqeust) 둘다 동기화하자   
쓰기와 읽기 모두가 동기화되지 않으면 동작을 보장하지 않는다.
어떤 기기에서는 둘 중 하나만 동기화해도 동작하는 듯 보이지만, 겉모습에 속아서는 안 된다.  
사실 이 두 메서드는 단순해서 동기화 없이도 원자적으로 동작한다.  
앞서 이야기했듯이 동기화는 배타적 수행과 스레드 간 통신이라는 두 가지 기능을 수행하는데,  
이 코드에서는 그중 통신 목적으로만 사용된 것이다.


### 동기화 문제를 피하는 방법
- 애초에 가변 데이터를 공유하지 않는 것
- 불변 데이터만 공유하거나 아무것도 공유하지 말자 
- 즉,가변 데이터는 단일 스레드에서만 쓰도록 하자.  

이 정책을 받아들였다면 그 사실을 문서에 남겨 유지보수 과정에서도 정책이 계속 지켜지도록 하는 게 중요하다.  
이런 외부 코드가 여러분이 인지하지 못한 스레드를 수행하는 복병으로 작용하는 경우도 있기 때문이다.

한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다.  
그러면 그 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들을 동기화 없이 자유롭게 값을 읽어갈 수 있다.  
이런 객체를 사실상 불변(effectively immutable)이라 하고 다른 스레드에 이런 객체를 건네는 행위를 안전 발행(safe publication)이라 한다.  



### 핵심 정리
- 여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화해야 한다.
- 동기화하지 않으면 한 스레드가 수행한 변경을 다른 스레드가 보지 못할 수도 있다.
- 공유되는 가변 데이터를 동기화하는 데 실패하면 응답 불가 상태에 빠지거나 안전 실패로 이어질 수 있다. 이는 디버깅 난이도가 가장 높은 문제에 속한다. 간헐적이거나 특정 타이밍에만 발생할 수도 있고, VM에 따라 현상이 달라지기도 한다.
- 배타적 실행은 필요 없고 스레드끼리의 통신만 필요하다면 volatile 한정자만으로 동기화할 수 있다. 다만 올바로 사용하기가 까다롭다.


---


**volatile 한정자**   
반복문에서 매번 동기화하는 비용이 크진 않지만 속도가 더 빠른 대안을 소개하겠다.  
stopRequested 필드를 volatile 로 선언하면 동기화를 생략해도 된다.  
volatile 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.

```java
public class VolatileStopThread {
   private static volatile boolean stopRequested;

   public static void main(String[] args) throws InterruptedException {
      Thread backgroundThread = new Thread(() -> {
         int i = 0;
         while (!stopRequested)
            i++;
      });
      backgroundThread.start();
      TimeUnit.SECONDS.sleep(1);
      stopRequested = true;
   }
}
```


volatile은 주의해서 사용해야 한다.  
예를 들어 다음은 일련번호를 생성할 의도로 작성한 메서드다.

```java
// 잘못된 코드 - 동기화가 필요하다!
private static volatile int nextSerialNumber = 0;
public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

이 메서드는 매번 고유한 값을 반환할 의도로 만들어졌다.  
이 메서드의 상태는 nextSerialNumber라는 단 하나의 필드로 결정되는데, 원자적으로 접근할 수 있고 어떤 값이든 허용한다.  
따라서 굳이 동기화하지 않더라도 불변식을 보호할 수 있어 보인다.  
하지만 이 역시 동기화 없이는 올바로 동작하지 않는다.

문제는 증가 연산자(++)다.  
이 연산자는 코드상으로는 하나지만 실제로는 nextSerialNumber 필드에 두 번 접근한다.  
먼저 값을 읽고, 그런 다음 (1 증가한) 새로운 값을 저장하는 것이다.  
만약 두 번째 스레드가 이 두 접근 사이를 비집고 들어와 값을 읽어가면 첫 번째 스레드와 똑같은 값을 돌려받게 된다.  
프로그램이 잘못된 결과를 계산해내는 이런 오류를 안전 실패(safety failure)라고 한다.

generateSerialNumber 메서드에 synchronized 한정자를 붙이면 이 문제가 해결된다.  
동시에 호출되도 서로 간섭하지 않으며 이전 호출이 변경한 값을 읽게 된다는 뜻이다.  
메서드에 synchronized를 붙였다면 nextSerialNumber 필드에서는 volatile을 제거해야 한다.  
이 메서드를 더 견 고하게 하려면 int 대신 long을 사용하거나 nextSerialNumber가 최댓값에 도달하면 예외를 던지게 하자.

아직 끝이 아니다.  
아이템 59의 조언을 따라 java.util.concurrent.atomic 패키지의 AtomicLong을 사용해보자.  
이 패키지는 락 없이도(lock-free; 락-프리) 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다.  
volatile은 동기화의 두 효과 중 통신 쪽만 지원하지만 이 패키지는 원자성(배타적 실행)까지 지원한다.   
우리가 generateSerialNumber에 원하는 바로 그 기능이다.  
더구나 성능도 동기화 버전보다 우수하다.

```java
// java.util.concurrent.atomic을 이용한 락-프리 동기화
private static final AtomicLong nextSerialNum = new AtomicLong();
public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```