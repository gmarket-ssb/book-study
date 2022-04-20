## wait 와 notify 보다는 동시성 유틸리티를 애용하라
> wait : 스레드가 어떤 조건이 충족되기를 기다릴 때 사용하는 메서드 (in Object)
> notify : 기다리는 스레드를 깨울 때 사용하는 메서드 (in Object)


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
  * `CountDownLatch`
    * 일회성 장벽으로 
