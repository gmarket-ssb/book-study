## [item 16] public 클래스에서는 public 필드가 아닌 접근자(getter) 메서드를 사용하라

<br>

이따금 인스턴스 필드들을 모아놓는 일 외에는 아무 목적도 없는 퇴보한 클래스를 작성하려 할 때가 있다.

```java
// 16-1 이처럼 퇴보한 클래스는 public이어서는 안된다! - bad
class Point {
    public double x;
    public double y;
}
```

이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다. (아이템 15)
불변식을 보장할 수 없고, 외부에서 필드에 접근할 때 부수작업을 수행하지도 못한다.
객체지향에서는 이런 클래스를 싫어해서 필드들을 모두 private로 바꾸고 public 접근자 getter를 추가한다.



```java
// 16-2 접근자(getter)와 변경자(setter) 메서드를 활용해 데이터를 캡슐화 한다. - good
class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() { return x; }
    public double getY() { return y; }
    
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

public class 라면 이 방식이 확실히 맞다. 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함
으로써 클래스 내부 표현 방식을 언제든 바꿀수 있는 유연성을 얻을 수 있다.

```
package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.
그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다. 
이 방식은 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다.
클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하나, 클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다.
따라서 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다. private 중첩 클래스의 경우라면
수정 범위가 더 좁아져서 이 클래스를 포함하는 외부 클래스까지로 제한 된다.
```

자바 플랫폼 라이브러리에도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종 있다.
대표적인 예가 java.awt.package 패키지의 Point와 Dimension 클래스다. 이 클래스들을 흉내 내지 말고, 타산지석으로 삼길 바란다.
아이템 67에서 설명하듯, 내부를 노출한 Dimension 클래스의 심각한 성능 문제는 오늘날까지도 해결되지 못했다.

```java
// 16-3 불변 필드를 노출한 public 클래스 - 과연 좋은가?
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;
    
    public final int hour;
    public final int minute;
    
    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("시간: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("분: " + minute);
        this.hour = hour;
        this.minute = minute;
    }
}
```

public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 여전히 결코 좋은 생각이 아니다.
여전히 API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다.
단 불변식은 보장할 수 있게 된다. 


### 핵심정리
1. public 클래스는 절대 가변 필드를 직접 노출해서는 안된다.   
2. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. (item 15의 배열의 경우처럼)  
3. 하지만 package-private(=default) 클래스나 private 중첩 클래스(nested-private) 에서는 종종 불변이든 가변이든 노출하는 편이 나을 때도 있다.
   (같은 패키지 안에서 어떤 특정 이유 때문에 사용 or 바깥 클래스에서 내부 클래스로만 접근하기 때문에 단점이 보완)


### 결국 최종 요약
- final 변수면 직접 접근해도 불변을 보장 하지만 불편함은 그대로 있으니 getter를 사용하자
- 우리가 사용하는 대부분의 클래스(=public class) getter를 사용해 변수에 접근하자