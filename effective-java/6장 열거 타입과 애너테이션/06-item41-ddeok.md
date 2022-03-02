## [item 41] 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

<br>

### 마커인터페이스
- 아무 메서드도 선언하지 않은 인터페이스
- ex) Serializable
  
```java
public interface Serializable {
    
}
```

ObjectOutputStream의 writeObject() 메서드 안에는 Serializable의 인스턴스인지 체크하는 메서드가 있음

```java
if (obj instanceof String) {
    writeString((String) obj, unshared);
} else if (cl.isArray()) {
    writeArray(obj, desc, unshared);
} else if (obj instanceof Enum) {
    writeEnum((Enum<?>) obj, desc, unshared);
} else if (obj instanceof Serializable) {
    writeOrdinaryObject(obj, desc, unshared);
} else {
    if (extendedDebugInfo) {
        throw new NotSerializableException(
            cl.getName() + "\n" + debugInfoStack.toString());
    } else {
        throw new NotSerializableException(cl.getName());
    }
}
```


### 마커 인터페이스의 장점
- 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있다.
- 마커 인터페이스는 어엿한 타입이기 때문에 오류를 컴파일타임에 잡을 수 있다.
- 마킹하고 싶은 클래스에서 그 인터페이스를 구현하면 마킹된 타입은 자동으로 그 인터페이스의 하위 타입임이 보장된다.


### 마커 어노테이션의 장점
- 반대로 마커 어노테이션이 마커 인터페이스보다 나은 점으로는 거대한 어노테이션 시스템의 지원을 받는다는 점을 들 수 있다.
- 따라서 어노테이션을 적극 활용하는 프레임워크 에서는 마커 어노테이션을 쓰는 쪽이 일관성을 지키는 데 유리할 것이다.
- 클래스와 인터페이스 외의 프로그램 요소(모듈, 패키지, 지역변수 등)에 마킹해야 할 때 어노테이션을 쓸 수 밖에 없다.


### 마커 인터페이스 vs 어노테이션
- 이 마킹이 된 객체를 매개변수로 받는 메서드를 작성할 일이 있을까? 
- 답이 "그렇다"이면 마커 인터페이스를 써야 한다. 
- 이렇게 하면 그 마커 인터페이스를 해당 메서드의 매개변수 타입으로 사용하여 컴파일 타임에 오류를 잡아낼 수 있다.
- 이런 메서드를 작성할 일은 절대 없다고 확신한다면 아마도 마커 어노테이션이 나은 선택일 것이다.


### 핵심정리
- 새로 추가하는 메서드 없이 단지 타입 정의가 목적 -> 인터페이스
- 클래스나 인터페이스 외의 프로그램 요소에 마킹해야 하거나 
- 어노테이션을 적극 활용하는 프레임워크의 일부로 그 마커를 편입시키고자 한다 -> 어노테이션
- 어노테이션을 활발히 활용하는 프레임워크 -> 어노테이션