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

### 불변식을 보장하지 못하는 사례 : Period 클래스 유효성 검사
 - readObject() 를 정의하지 않아서, 자바의 기본 직렬화를 수행한다.
 ```java
public final class Period implements Serializable {

    private Date start;
    private Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime()); // 방어적 복사
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0) { // 유효성 검사
            throw new IllegalArgumentException(start + " after " + end);
        }
    }

    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }
}
 ```

 - 아래와 같은 바이트스트림으로 Period 객체로 역직렬화한다면?

```java
public class BogusPeriod {
    // 불변식을 깨뜨리도록 조작된 바이트 스트림
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
        0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8,
        0x2b, 0x4f, 0x46, (byte)0xc0, (byte)0xf4, 0x02, 0x00, 0x02,
        0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c,
        0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f,
        0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74,
        0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70,
        0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75,
        0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a,
        (byte)0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
        0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte)0xdf,
        0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03,
        0x77, 0x08, 0x00, 0x00, 0x00, (byte)0xd5, 0x17, 0x69, 0x22,
        0x00, 0x78
    };

    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p.start);
        System.out.println(p.end);
    }
    
    static Object deserialize(byte[] sf) {
        try {
            return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```

 - 위 바이트스트림의 정보는, start 의 시각이 end 의 시각보다 느리게 조작했다.
 - 즉, 불변식을 꺠뜨린 객체로 역직렬화하도록 조작되었다.
 
 ```
 Fri Jan 01 12:00:00 PST 1999 // start 가 더 느리다.
 Sun Jan 01 12:00:00 PST 1984 // end 가 더 이르다.
```

#### 해결방법
 - readObject 를 정의하고, 유효성 검사를 실시한다.
 - Period 클래스에 다음의 메서드를 추가한다.
 ```java
 private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
     s.defaultReadObject(); // 기본 직렬화 수행
     if (start.compareTo(end) > 0) { // 유효성 검사
         throw new InvalidObjectException(start + " 가 " + end + " 보다 늦을 수 없습니다.");
     }
 }
 ```


<br>
<br>

### 불변식을 보장하지 못하는 사례 : Period 클래스 방어적 복사
 - 직렬화된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들 수 있다.
 ```java
public class MutablePeriod {
    public final Period period;

    public final Date start;

    public final Date end;

    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(bos);

            // 불변식을 유지하는 Period 를 직렬화.
            out.writeObject(new Period(new Date(), new Date()));

            /*
             * 악의적인 start, end 로의 참조를 추가.
             */
            byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // 악의적인 참조
            bos.write(ref); // 시작 필드
            ref[4] = 4; // 악의적인 참조
            bos.write(ref); // 종료 필드

            // 역직렬화 과정에서 Period 객체의 Date 참조를 훔친다.
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

```java
    public static void main(String[] args) {
        MutablePeriod mp = new MutablePeriod();
        Period mutablePeriod = mp.period; // 불변 객체로 생성한 Period
        Date pEnd = mp.end; // MutablePeriod 클래스의 end 필드
        
        pEnd.setYear(78); // MutablePeriod 의 end 를 바꿨는데 ?
        System.out.println(mutablePeriod.end()); // Period 의 값이 바뀐다.
        
        pEnd.setYear(69);
        System.out.println(mutablePeriod.end());
    }
```
 - 결과
 ```
Fri Apr 07 19:59:32 KST 1978
Mon Apr 07 19:59:32 KST 1969
 ```

 - 불변 객체 Period 를 직렬화 / 역직렬화한다고 생각할 수 있지만,
 - 위의 방법으로 불변식을 깨뜨릴 수 있다.
 - 실제로 String 이 불변이라는 사실에 기댄 보안 문제들이 존재한다.
 
#### 해결법
 - 객체를 역직렬화할 때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 방어적 복사한다.
 - 불변 클래스 안의 모든 private 가변 요소를 방어적 복사한다.
 ```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 방어적 복사를 통해 인스턴스의 필드값 초기화
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 유효성 검사
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start +" after "+ end);
}
```
 - 유효성 검사보다 먼저 방어적 복사
    - 반대라면, 유효성 검사 이후 방어적 복사 이전에 불변식을 깨뜨릴 틈이 생긴다. (item 50)
 - final 필드는 방어적 복사가 불가능하므로, 필드를 final 이 아니게 해야함.
 
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

https://velog.io/@guswlsapdlf/Java%EC%9D%98-Mutable%EA%B3%BC-Immutable
https://velog.io/@max9106/Java-%EB%B0%A9%EC%96%B4%EC%A0%81-%EB%B3%B5%EC%82%ACDefensive-copy
https://stackoverflow.com/questions/9979982/should-i-use-the-final-modifier-when-creating-date-objects
https://blog.yevgnenll.me/posts/implement-serializable-with-great-caution-effective-java-86
https://doridorigang.tistory.com/2

Mutable vs Imutable
<img width="834" alt="image" src="https://user-images.githubusercontent.com/5934737/165678520-c0c54f72-dace-47cc-a1d7-b84c79794d3a.png">
