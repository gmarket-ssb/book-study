## [Item07] 다 쓴 객체 참조를 해제하라. ( ing )
> 1. 자기 메모리를 직접관리하는 클래스라면 반드시 누수를 주의
> 2. 객체를 유효범위 밖으로 밀어내는 것이 기본 : 객체참조를 null 처리하는 것은 예외적인 경우로 한다.
> 3. 캐시를 사용를 사용하여 객체 참조하는 경우 : WeakHashMap을 사용 고려 ( Strong References, Soft References, Weak References )


## **상세설명**

### 1. 메모리 누수 예**
``` java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
- ensureCapacity() 는 어느순간 outofmemory를 일으킨다.
</br></br>

### 2. 객체를 유효범위 밖으로 밀어내기

- 지역변수를 선언할 때, 해당 scope에서만 선언하도록 만들어라.
- for문 내부에, 사용하는 변수는 그 외부에 선언을 해놓으면 GC가 자동으로 정리해주지 못한다.
</br></br>

### 3. 캐시를 사용를 사용하여 객체 참조하는 경우
> GC vs non-GC
- GC 하지 않는 경우
  - Strong References
      ``` java
      > Integer prime = 1;
      ```
  - Soft References
      ``` java
      > Integer prime = 1;  
      > SoftReference<Integer> soft = new SoftReference<Integer>(prime); 
      > prime = null;
      ```
- GC 하는 경우
  - Weak References
      ``` java
      > Integer prime = 1;  
      > WeakReference<Integer> soft = new WeakReference<Integer>(prime); 
      > prime = null;
      ```
- WeakHashMap 활용의 예
  ``` java   
    > WeakHashMap<UniqueImageName, BigImage> map = new WeakHashMap<>();
    > BigImage bigImage = new BigImage("image_id");
    > UniqueImageName imageName = new UniqueImageName("name_of_big_image");
    > 
    > map.put(imageName, bigImage);
    > assertTrue(map.containsKey(imageName));
    > 
    > imageName = null;
    > System.gc();
    > 
    > await().atMost(10, TimeUnit.SECONDS).until(map::isEmpty);
  ```
 
