## [item 79] 과도한 동기화는 피하라

### 핵심
> 과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 예측할 수 없는 동작을 낳는다.


* 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안된다.
    1. 재정의할 수 있는 메서드는 호출 하면 안 된다.
    2. 클라이언트가 넘겨준 함수 객체를 호출해선 안 된다.

> 위에 해당하는 메서드는 '무슨 일을 할지 알지 못하며 통제할 수 없는' 외계인 메서드<br>
> 외계인 메서드를 사용하면 동기화된 영역은 *1)* 예외를 일으키거나 *2)* 교착상태에 빠지거나 *3)* 데이터를 훼손 할 수 있다.

--- 

### 동기화 블록 안에서 외계인 메서드를 호출 
(동기화 블록 안에서 외부 메서드를 호출하는 잘못된 코드)
```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers
            = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for (SetObserver<E> observer : observers) {
                observer.added(this, element);
            }
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);
        return result;
    }
}
```

```java
@FunctionalInterface
public interface SetObserver<E> {
    void added(ObservableSet<E> set, E element); 
    //ObservableSet에 원소가 추가되면 호출
}
```
addObserver / removeObserver 메서드 호출시에 위와 같은 콜백 인터페이스의 인스턴스를 넘긴다.

```java
public static void main(String[] args) {
    ObservableSet<Integer> set =
            new ObservableSet<>(new HashSet<>());

    set.addObserver((s, e) -> System.out.println(e));

    for (int i = 0; i < 100; i++) {
        set.add(i);
    }
}
```
위와 같은 경우는 0부터 99까지 잘 출력한다.

```java
public static void main(String[] args) {
    ObservableSet<Integer> set =
            new ObservableSet<>(new HashSet<>());

    set.addObserver(new SetObserver<Integer>() {
        @Override
        public void added(ObservableSet<Integer> set, Integer element) {
            System.out.println(element);
            if (element == 23)
                set.removeObserver(this);
        }
    });

    for (int i = 0; i < 100; i++) {
        set.add(i);
    }
}
```
0부터 23까지 출력한 후 자신을 remove 한 후에 종료할 것 같으나 실제로 실행해보면 0~23까지 출력한 후 ConcurrentModificationException가 발생
<br>added 메서드 호출이 일어난 시점이 notifyElementAdded가 Observer들의 리스트를 순회하는 도중이기 때문이다.

---

### 쓸데없이 백그라운드 스레드를 사용하는 코드
    removeObserver를 직접 호출하지 않고 실행자 서비스를 통해 다른 스레드를 이용
```java
public static void main(String[] args) {
    ObservableSet<Integer> set =
            new ObservableSet<>(new HashSet<>());

    set.addObserver(new SetObserver<Integer>() {
        @Override
        public void added(ObservableSet<Integer> set, Integer element) {
            System.out.println(element);
            if (element == 23) {
                ExecutorService exec =
                        Executors.newSingleThreadExecutor();

                try {
                    exec.submit(() -> set.removeObserver(this)).get();
                } catch (ExecutionException | InterruptedException ex) {
                    throw new AssertionError(ex);
                } finally {
                    exec.shutdown();
                }
            }
        }
    });

    for (int i = 0; i < 100; i++) {
        set.add(i);
    }
}
```

위의 경우는 예외는 발생하지 않지만 교착상태(Deadlock)에 빠진다. 
<br>백그라운드 스레드가 s.removeObserver 메서드를 호출하면 Observer를 잠그려 시도하지만 락을 얻을 수 없다. 메인 스레드가 이미 락을 잡고 있기 때문이다.
<br>외계인 메서드 예제를 정의한 코드를 보면 removeObserver 메서드에는 synchronized 키워드가 있기 때문에 실행 시 락이 걸린다. 
<br>동시에 메인 스레드는 백그라운드 스레드가 remove해주기를 기다리는 중!

---

## 예외, 교착상태, 재진입 가능한 락의 문제들을 해결할 수 있는 방법

### 외계인 메서드를 동기화 블록 바깥으로 옮긴다.

* 관찰자 리스트를 복사해서 사용
```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot)
        observer.added(this, element);
}
```

### 동시성 컬렉션 라이브러리 사용
```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
```
CopyOnWriteArrayList은 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현되어있다.
(수정할일은 적고 순회할일이 많은 관찰자 리스트 용도로 좋다)

---

### 성능을 고려하자
> 과도한 동기화는 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용이다.(경쟁하며 낭비하는 시간)
> <br>또한 과도한 동기화는 JVM의 코드 최적화를 제한한다는 것도 고려해야 한다.
