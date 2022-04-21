## [item 80] 스레드보다는 실행자, 테스크, 스트림을 애용하라

### java.util.concurrent 패키지
실행자 프레임워크(Executor Framework)라고 하는 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있음

```java
// 작업큐를 한줄로 생성할 수 있음
ExecutorService exec = Executors.newSingleThreadExecutor();

// 실행자를 넘기는 방법
exec.execute(runnable);

// 실행자 종료
exec.shutdown();
```

### 실행자 서비스의 주요 기능
- 특정 태스크가 완료되기를 기다림
- 태스크 모음 중 아무것 하나(invokeAny 메서드) 혹은 모든 태스크(invokeAll 메서드)가 완료되기를 기다림
- 실행자 서비스가 종료하기를 기다린다(awaitTermination 메서드)
- 완료된 태스크들의 결과를 차례로 받는다(ExecutorCompletionService 이용)
- 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다(ScheduledThread PoolExecutor 이용)

### 기타
- 큐를 둘 이상의 스레드가 처리하게 하고 싶다 -> 다른 정적 팩터리를 이용, 다른 종류의 실행자 서비스(스레드 풀)를 생성
- 대부분의 실행자는 java.util.concurrent.Executors의 정적 팩토리들로 생성 가능
- 평범하지 않은 실행자를 원한다면 ThreadPoolExecutor 클래스를 직접 사용 -> 스레드 풀 동작을 결정하는 거의 모든 속성을 설정 가능


### 용도
- 작은/가벼운 서버라면 Executors.newCachedThreadPool을 사용 -> 특별히 설정할 게 없고 일반적인 용도에 적합하게 동작함
  - 태스크가 큐에 쌓이지 않고 즉시 실행, 가용한 스레드가 없다면 새로 생성 -> 사용량이 많으면 CPU 100%
- 무거운/프로덕션 서버에서는 스레드 개수를 고정한 Executors.newFixedThreadPool 사용
- 혹은 완전히 통제할 수 있는 ThreadPoolExecutor를 직접 사용


### 실행자 프레임워크의 장점
- 작업 단위와 실행 단위가 분리 됨
- 작업단위 = 태스크 (Runnable, Callable)
- Callable은 Runnable과 비슷하지만 값을 리턴하고 예외를 던질 수 있음
- 태스크를 수행하는 일반적인 매커니즘 -> 실행자 서비스
- 실행자 프레임워크가 작업 수행을 담당




```java
// newFixedThreadPool 사용 예
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ExecutorsExample {
    public static void main(String[] args) {
        System.out.println("Inside : " + Thread.currentThread().getName());

        System.out.println("Creating Executor Service with a thread pool of Size 2");
        ExecutorService executorService = Executors.newFixedThreadPool(2);

        Runnable task1 = () -> {
            System.out.println("Executing Task1 inside : " + Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException ex) {
                throw new IllegalStateException(ex);
            }
        };

        Runnable task2 = () -> {
            System.out.println("Executing Task2 inside : " + Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(4);
            } catch (InterruptedException ex) {
                throw new IllegalStateException(ex);
            }
        };

        Runnable task3 = () -> {
            System.out.println("Executing Task3 inside : " + Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException ex) {
                throw new IllegalStateException(ex);
            }
        };


        System.out.println("Submitting the tasks for execution...");
        executorService.submit(task1);
        executorService.submit(task2);
        executorService.submit(task3);

        executorService.shutdown();
    }
}
```

```java
// Output
Inside : main
Creating Executor Service with a thread pool of Size 2
Submitting the tasks for execution...
Executing Task2 inside : pool-1-thread-2
Executing Task1 inside : pool-1-thread-1
Executing Task3 inside : pool-1-thread-1
```

- newFixedThreadPool를 사용하여 2개의 고정된 thread들을 관리하는 thread pool을 생성
- 고정된 thread pool에서는 executor service가 생성한 pool이 언제나 특정한 thread들이 실행된다는 것을 보장
- thread pool에서는 pool안의 thread가 죽더라도 그 자리를 새로운 thread가 즉시 대체
- 새로운 task가 제출되면 executor service는 현재 할당이 가능한 thread를 pool에서 뽑고 그 task를 뽑은 thread에 할당
- 만약 thread들이 이전에 할당된 task들을 처리해야해서 할당가능한 thread가 존재하지 않으면, 그 이후에 제출된 task들은 queue에서 대기


### 포크-조인(fork-join) 태스크
- 포크조인 풀이라는 실행자 서비스가 실행함
- ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있고, ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리함
- 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 처리
- 직접 작성하고 튜닝하긴 어렵지만 포크조인 풀을 이용한 병렬 스트림을 이용하면 이점을 얻을 수 있음


### 참고
https://engkimbs.tistory.com/589
https://www.callicoder.com/java-executor-service-and-thread-pool-tutorial/
