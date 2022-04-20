## wait 와 notify 보다는 동시성 유틸리티를 애용하라
> wait : 스레드가 어떤 조건이 충족되기를 기다릴 때 사용하는 메서드 (in Object)
> notify : 기다리는 스레드를 깨울 때 사용하는 메서드 (in Object)

<br>

### `java.util.concurrent` 패키지의 범주
1. 실행자 프레임워크 (Executor Framework)
   * Item80
2. 동시성 컬렉션 (Concurrent Collection)
   * 동시성 컬렉션은 `List, Queue, Map` 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다.
   * 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다.
   * 따라서 동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.
   * 추가로 동시성 컬렉션의 등장은 동기화한 컬렉션을 낡은 유산으로 만들어버렸다.
     * ex) `Collections.synchronizedMap` 보다는 `ConcurrentHashMap` 을 사용하는 게 훨씬 좋다.
3. **동기화 장치 (Synchronizer)**
   * 동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다.
     * 가장 자주 쓰이는 동기화 장치 : CountDownLatch. Semaphore
     * 덜 쓰이는 동기화 장치 : CyclicBarrier, Exchanger
     * 가장 강력한 동기화 장치 : Phaser

<br>

### CountDawnLatch
일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때 까지 기다리게 한다.
* 어떤 동작들을 동시에 시작해 모두 완료하기까지의 시간을 재는 간단한 프레임워크를 만든다고 했을 때, CountDownLatch 프레임워크를 사용하면 놀랍도록 간단해진다.
* **이를 wait 와 notify 로 구현하려면 아주 난해해지고 지저분한 코드가 탄생한다.**
  ```java
  public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
      CountDownLatch ready = new CountDownLatch(concurrency);
      CountDownLatch start = new CountDownLatch(1);
      CountDownLatch done = new CountDownLatch(concurrency);

      for (int i=0; i<concurrency; i++) {
          executor.execute(() -> {
              // 타이머에게 준비를 마쳤음을 알린다.
              ready.countDown();

              try {
                  // 모든 작업자 스레드가 준비될 때까지 기다린다.
                  start.wait();
                  action.run();
              } catch (InterruptedException e) {
                  Thread.currentThread().interrupt();
              } finally {
                  // 타이머에게 작업을 마쳤음을 알린다.
                  done.countDown();
              }
          });
      }

      ready.await(); // 모든 작업자가 준비될 때 까지 기다린다.
      long startNanos = System.nanoTime(); // 시간 간격을 잴 때는 항상 Syatem.nanoTime 을 사용하자.
      start.countDown(); // 작업자들을 깨운다.
      done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.

      return System.nanoTime() - startNanos;
  }
  ```

<br>

### 정리
* wait 와 notify 보다는 동시성 유틸리티를 사용하자.
* 만일 wait 와 notify 가 있는 레거시 코드를 유지보수해야 한다면...
 * wait 는 항상 표준 관용구에 따라 while 문 안에서 호출하도록 하고
 * 일반적으로 notify 보다는 notifyAll 을 사용하자.


