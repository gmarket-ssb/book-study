# readObject 메서드는 방어적으로 작성하라.

* 방어적복사( Defensive copy)
  - 생성자를 통해 초기화 할 때, 새로운 객체로 감싸서 복사해주는 방법이다. 외부와 내부에서 주소값을 공유하는 인스턴스의 관계를 끊어주기 위함
* mutable vs imutable
  - stringBuilder의 값을 abc에서 abcdef로 변경하였을 때, 메모리 주소 값이 변경되지 않았다. 이는 곧 1785210046의 메모리 주소에 할당된 abc란 값이 abcdef로 변한 것임을 나타낸다. 따라서 StringBuffer class는 Mutable하게 동작
  - str의 값을 abc에서 abcdef로 변경하였을 때, 메모리 주소 값도 같이 변경되었다. 이는 곧 1785210046의 메모리 주소에 할당된 abc란 값이 abcdef로 변한 것이 아니라 1151020327의 메모리 주소에 abcdef란 값을 가진 String 객체가 새로 생성된 것임을 나타낸다. 따라서 String class는 Immutable하게 동작
 
## readObject 메서드
 - readObject는 Public 생성자이므로 방어적 복사를 고려해야한다.
 ```java
public class BogusPeriod {
    
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06....
    }
    
    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p);
    }
    
    static Object deserialize(byte[] sf) {
        try {
            return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
        } catch(IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```
- 이 코드의 serializedForm에서 상위 비트가 1인 바이트 값들은 byte로 형변환했는데,
- 이는 자바가 바이트 리터럴을 지원하지 않고 byte 타입은 부호가 있는 (signed) 타입이기 때문이다.
- 위의 프로그램을 실행하면
Fri Jan 01 12:00:00 PST 1999 - Sun Jan 01 12:00:00 PST 1984를 출력한다.
- Period를 직렬화할 수 있도록 선언한 것 만으로도 불변식을 깨뜨리는 객체를 만들 수 있다.

<br>
<br>

## 그럼 어떻게? - 유효성검사수행
역직렬화 된 이후 유효성검사를 추가로 수행한다.
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    // 불변식을 만족하는지 검사한다.
    if(start.compareTo(end) > 0) {
       throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```
- 위의 if문 추가로 허용되지 않는 Period 인스턴스가 생성되는 일을 막을 수 있지만, 아직도 미묘한 문제가 숨어있다.
정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들 수 있다
 
```java
 public class MutablePeriod {
    //Period 인스턴스
    public final Period period;
    
    //시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date start;
    //종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date end;
    
    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectArrayOutputStream out = new ObjectArrayOutputStream(bos);
            
            //유효한 Period 인스턴스를 직렬화한다.
            out.writeObject(new Period(new Date(), new Date()));
            
            /**
             * 악의적인 '이전 객체 참조', 즉 내부 Date 필드로의 참조를 추가한다.
             * 상세 내용은 자바 객체 직렬화 명세의 6.4절을 참고
             */
            byte[] ref = {0x71, 0, 0x7e, 0, 5}; // 참조 #5
            bos.write(ref); // 시작 start 필드 참조 추가
            ref[4] = 4; //참조 #4
            bos.write(ref); // 종료(end) 필드 참조 추가
            
            // Period 역직렬화 후 Date 참조를 훔친다.
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }
}
```
다음 공격 코드를 실행하면 이 공격이 실제로 이뤄지는 모습을 확인할 수 있다.

```java
public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period p = mp.period;
    Date pEnd = mp.end;
    
    //시간 되돌리기
    pEnd.setYear(78);
    System.out.println(p); // Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1978
    
    //60년대로 회귀
    pEnd.setYear(60);
    System.out.println(p); // Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1969
}
```
문제는 정상적으로 직렬화된 Period 인스턴스의 바이트 스트림 끝에
private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어 낼 수가 있다.
공격자는 ObjectInputStream에서 Period 인스턴스를 읽은 후, 스트림 끝에 추가되어 있는'악의적인 객체 참조'를 읽어 Period 객체의 내부 정보를 얻을 수 있다. 
그리고 이 참조로 얻은 Date 인스턴스들을 검사 없이 수정해버릴 수도 있으니, Period 인스턴스의 필드는 더 이상 검사되지 않는다.

즉,이 예에서 Period 인스턴스는 불변식을 유지한 채 생성됐지만, 이렇게 의도적으로 내부의 값을 수정할 수 있다. 
이처럼 변경할 수 있는 Period 인스턴스를 획득한 공격자는 이 인스턴스가 불변이라고 가정하는 클래스에 넘겨 엄청난 보안 문제를 일으킬 수 있다.

```java
The example in the question, serialization layout can regard as:

#0 Period Class desc
#1 String Class 
#2 Period instance - new 
#3 Date Class desc
#4 Date instance - end (name by ascending order)
#5 Date instance - start 
More detail link: The Java serialization algorithm revealed. There are some differences between different compiler level, because the serialization version may be different.

Then in deserialization all above resolved result are stored in a object array by order - called entries(hold reference for improving performance, etc). So reading a object from "{0x71, 0, 0x7e, 0, 5}" is equivalent to getting entries[5], it's a reference to "start".
When I study in java serialization, I also read the《Effective Java》and wrote the serialization demo. You can probe like this:

byte[] ref = {0x71, 0x0, 0x7E, 0x0, 0}; // ref #0, #1, #2... 
bos.write(ref);
ObjectInputStream ois = new ObjectInputStream(new 
                        ByteArrayInputStream(bos.toByteArray()));
period = (Period)ois.readObject();
Object obj = ois.readObject();
System.out.println(obj);
if(obj instanceof ObjectStreamClass)
    System.out.println("fields: " + Arrays.toString(((ObjectStreamClass) obj).getFields()));
else 
    System.out.println(obj.getClass());

```

### readObject 메서드에서는 private 가변요소를 방어 복사하라
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());
    
    // 불변식을 만족하는지 검사한다.
    if(start.compareTo(end) > 0) {
       throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```
방어적 복사를 유효성 검사보다 앞서 수행하며, Date의 clone 메서드는 사용하지 않았음에 주목하자.
두 조치 모두 Period를 공격으로 부터 보호하는데 필요하다.
또한 final 필드는 방어적 복사가 불가능 하니 주의하자
그래서 이 readObject를 사용하려면 start와 end필드에서 final 한정자를 제거해야 한다.
 
<br>
<br>

## 기본 readObject() vs readObject 직접 정의 ?
 `transient 필드를 제외한 모든 필드의 값을 매개변수`로 받아 `유효성 검사 없이` 필드에 대입하는 `public 생성자` 를 추가해도 괜찮은가 ?
 - Yes -> 기본 readObject
 - No -> readObject 직접 정의 후 유효성 검사와 방어적 복사 수행. or 직렬화 프록시 패턴사용.
 
### 마지막 팁
 - readObject 메서드에서 재정의 가능 메서드를 호출하면 안된다. (item 19)
    - 클래스가 final 이 아닌 경우에만 해당
    - 이 클래스의 하위 클래스가 불리기 이전에 생성자의 재정의된 메서드가 실행되므로 오류를 뱉게 될 것이다.

### 참고
https://stackoverflow.com/questions/40331800/how-does-referencing-work-in-java
https://velog.io/@guswlsapdlf/Java%EC%9D%98-Mutable%EA%B3%BC-Immutable
https://velog.io/@max9106/Java-%EB%B0%A9%EC%96%B4%EC%A0%81-%EB%B3%B5%EC%82%ACDefensive-copy
https://stackoverflow.com/questions/9979982/should-i-use-the-final-modifier-when-creating-date-objects
https://blog.yevgnenll.me/posts/implement-serializable-with-great-caution-effective-java-86
https://doridorigang.tistory.com/2

Mutable vs Imutable
<img width="834" alt="image" src="https://user-images.githubusercontent.com/5934737/165678520-c0c54f72-dace-47cc-a1d7-b84c79794d3a.png">
