## 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라
> 직렬화는 생성자를 이용하지 않고도 인스턴스를 생성하는 기능을 제공하므로, 버그와 보안 문제가 생길 가능성이 커진다.<br>
> 하지만 **직렬화 프록시 패턴** 을 사용하면 이런 언어 도단적 특성을 상당 부분 제거할 수 있다.
<br><br>

### 직렬화 프록시 패턴(serialization proxy pattern)
![image](https://user-images.githubusercontent.com/57446639/165578676-4b510927-0fe6-475b-939a-a2c1108fd3f4.png)
```java
public class Period implements Serializable {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = start;
        this.end = end;
    }

    // proxy
    private static class SerializationProxy implements Serializable {
        private final Date start;
        private final Date end;

        private static final long serialVersionUID = 123123L;

        public SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

        private Object readResolve() {
            return new Period(start, end);
        }
    }

    // 직렬화 프록시 패턴용 writePlace 메서드
    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    // 직렬화 프록시 패턴용 readObject 메서드
    private void readObject(ObjectInputStream ois) throws IOException {
        throw new InvalidObjectException("Proxy required!");
    }
}
```
* 바깥 클래스의 논리적 상태를 표현하는 중첩 클래스를 설계하여 `private static` 으로 선언한다.
  * -> 여기서 중첩 클래스가 직렬화 프록시다.
* 그리고 바깥 클래스와 직렬화 프록시 모두 Serializable 을 구현한다고 선언해야 한다.
<br><br>

### 직렬화 프록시 패턴의 강점
직렬화 프록시 패턴이 `.readObject()` 에서의 방어적 복사보다 강력한 경우가 하나 더 있다.<br>
직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다.
<br><br>

### 직렬화 프록시 패턴의 한계
* 클라이언트가 멋대로 확장할 수 있는 클래스에는 적용할 수 없다.
* 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다.
