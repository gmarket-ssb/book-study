# 적시에 방어적 본사본을 만들어라

자바는 안전한 언어이다. 자바로 작성한 클래스는 시스템의 다른 부분에서 무슨 짓을 하든 그 불변식이 지켜진다. 하지만 자바라 해도 다른 클래스로부터의 침범을 아무런 노력 없이 다 막을 수 있는 건 아니다. 그러니 클라이언트가 여러분의 불변식을 깨뜨리려고 혈안이 되어 있다고 가정하고 방어적으로 프로그래밍해야 한다.

## 방어적 복사를 통해 불변식을 지키자

- 생성자에서 받은 가변 매개 변수 각각을 방어적으로 복사(defensive copy)해야한다.
    - 매개변수의 유효성을 검사하기 전에 방어적 본사본을 만들고, 이 복사본으로 유효성을 검사해야 한다. (멀티쓰레딩 환경에서 방어)
    - 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 본사본을 만들때 clone 메서드를 사용해서는 안된다.
- 가변 필드의 방어적 복사본을 반환하면 된다.
    - 생성자와 달리 접근자 메서드에서는 방어적 복사에 clone 메서드를 사용해도 되지만 생성자나 정적 팩토리를 쓰는게 좋다(item18)
- 되도록 불변 객체들을 조합해 객체를 구성하면 방어적 복사를 할 일이 줄어든다.
    - e.g. 아래의 경우 Date 대신 불변인 Instant를 사용하면 된다.
- 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때 객체가 변화해도 정상적으로 확신할 수 없다면 복사본을 만들어서 저장해야 한다.
    - e.g. 클라이언트가 건네준 객체를 내부의 Set 인스턴스에 저장하거나 Map 인스턴스의 키로 사용한다면, 추후 그 객체가 변경될 경우 객체를 담고 있는 Set 혹은 Map의 불변식이 깨질 것이다.

```java
public class Period3 {
    private final Date start;
    private final Date end;

    public Period3(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());

        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException();
        }
    }

    public Date getStart() {
        return new Date(this.start.getTime());
    }

    public Date getEnd() {
        return new Date(this.end.getTime());
    }
}

Date start3 = new Date();
Date end3 = new Date();
Period3 period3 = new Period3(start3, end3);
System.out.println("period3.getEnd() = " + period3.getEnd()); // Sun Mar 06 15:53:23 KST 2022
period3.getEnd().setYear(78);
System.out.println("period3.getEnd() = " + period3.getEnd()); // Sun Mar 06 15:53:23 KST 2022
```

## 핵심정리

- 클래스가 클라이언트로 부터 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 그 요소는 반드시 방어적으로 복사해야 한다.
- 복사 비용이 너무 크거나 클라이언트가 그 요소를 잘못 수정할 일이 없음을 신뢰한다면 방어적 복사를 수행하는 대신 해당 구성요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명시하자.