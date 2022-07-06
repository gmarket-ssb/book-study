## 냄새 18. 중재자(Middle Man)

### 개요

- 캡슐화를 통해 구체적인 정보를 감출 수 있지만, 너무 감추는 것도 문제.
- 메시지 체인에 반대가 되는 냄새
  - **위임 숨기기**를 통해 메시지 체인에 해당하는 부분을 감출 수 있음
- 캡슐화를 지나치게 하다보니, 원하는 데이터를 바로 가져오는 것이 아닌 객체(중재자)를 거쳐서 가져와야 하는 번거로움
- 클라이언트에게 데이터를 전달할 때 누군가를 거쳐가는 느낌이 든다면 중재자 냄새로 판단
  - 단순히 느낌으로만 판단

### 리팩토링 38. 중재자 제거하기

- **위임 숨기기**에 반대에 해당하는 리팩토링
- 캡슐화 정도를 **중재자 제거하기**, **위임 숨기기** 리팩토링을 통해 조절할 수 있음
- 위임하는 객체를 getter로 제공, 클라이언트에서 메시지 체인을 사용하도록 코드를 수정
- [Law Of Demeter](https://en.wikipedia.org/wiki/Law_of_Demeter)를 따르기 보다 상황에 맞게 활용
  - **Don't talk to Strangers.** **Principle of least knowledge(최소 지식 원칙)**

> **AS-IS**: `Person`에서 `manager`를 가져올려면 `Department`위임 를 거쳐야함
```java
public class Department {

  private Person manager;

  public Department(Person manager) {
    this.manager = manager;
  }

  public Person getManager() {
    return manager;
  }
}

public class Person {

  private Department department;

  private String name;

  public Person(String name, Department department) {
    this.name = name;
    this.department = department;
  }

  public Person getManager() {
    return this.department.getManager();
  }
  
}

class PersonTest {

  @Test
  void getManager() {
    Person nick = new Person("nick", null);
    Person keesun = new Person("keesun", new Department(nick));
    assertEquals(nick, keesun.getManager());
  }

}
```
- Department에 필드가 늘어난다면, 모든 정보를 Person을 거쳐서 제공해야함
- Person에선 단순히 Department에게 bypass하는 용도
> **TO-BE**: `Person`에서 `Department`의 getter를 생성. 클라이언트에서 메시지 체인을 통해 가져감
```java
public class Person {

  private Department department;

  private String name;

  public Person(String name, Department department) {
    this.name = name;
    this.department = department;
  }

  // 위임 객체를 직접 전달
  public Department getDepartment() {
    return department;
  }

}

class PersonTest {

  @Test
  void getManager() {
    Person nick = new Person("nick", null);
    Person keesun = new Person("keesun", new Department(nick));
    assertEquals(nick, keesun.getDepartment().getManager());  // 클라이언트에선 메시지 체인을 통해 접근할 수 있음
  }

}
```

### 리팩토링 39. 슈퍼클래스를 위임으로 바꾸기

- 객체지향에선 **상속**은 기존 기능을 재사용하는 쉬우면서 강력한 방법. 때로는 적잘하지 않기도 함
- 서브클래스에선 슈퍼클래스의 모든 기능을 지원해야함
- 서브클래스는 슈퍼클래스 자리를 대체하더라도 잘 동작해야함
  - [리스코프 치환 원칙](https://ko.wikipedia.org/wiki/%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84_%EC%B9%98%ED%99%98_%EC%9B%90%EC%B9%99)
- 서브클래스는 슈퍼클래스 변경에 취약
- 그럼 상속을 사용하지 않는 것이 좋을까?
  - 적절한 경우에 사용한다면 쉽고 효율적인 방법
  - 상속 사용이 적절치 않다고 생각되면 해당 리팩토링을 적용

> **AS-IS**: `Scroll` 클래스가 `CategoryItem`이라는 클래스를 상속
```java
public class Scroll extends CategoryItem {

  private LocalDate dateLastCleaned;

  public Scroll(Integer id, String title, List<String> tags, LocalDate dateLastCleaned) {
    super(id, title, tags);
    this.dateLastCleaned = dateLastCleaned;
  }

  public long daysSinceLastCleaning(LocalDate targetDate) {
    return this.dateLastCleaned.until(targetDate, ChronoUnit.DAYS);
  }
}
```
- 상속구조로 유지보수하기 어렵다면, 해당 슈퍼클래스를 위임 구조로 변경
> **TO-BE**: 슈퍼클래스 상속 대신 위임 구조로 변경
```java
public class Scroll {

  private LocalDate dateLastCleaned;

  private CategoryItem categoryItem;

  public Scroll(Integer id, String title, List<String> tags, LocalDate dateLastCleaned) {
    // 슈퍼클래스 생성에 필요한 데이터는 이미 있으니 그래도 사용하면 됨
    categoryItem = new CategoryItem(id, title, tags);
    this.dateLastCleaned = dateLastCleaned;
  }

  public long daysSinceLastCleaning(LocalDate targetDate) {
    return this.dateLastCleaned.until(targetDate, ChronoUnit.DAYS);
  }
}
```

### 리팩토링 40. 서브클래스를 위임으로 바꾸기

- 행동이 카테고리에 따라 바뀐다면, 일반적인 로직을 슈퍼클래스로 두고 특이케이스는 서브클래스를 사용해 표현
- 명확한 단점이라고 하면 **한 번만 사용**할 수 있음
  - 카테고리의 기준이 **하나**여야 한다는 뜻
  - `Person`의 나누는 기준이 `Age(나이)` 혹은 `Income-Level(소득수준)`이어야지, 둘 다일 수 없음
- 하지만 위임을 통해 여러 기준으로 다른 객체에게 위임할 수 있음
- 그리고 슈퍼클래스가 변경되면 서브클래스에도 영향
  - 만약 서브 클래스가 다른 모듈에 존재한다면??
  - 위임을 사용해 중간에 인터페이스를 만들어 의존성을 줄일 수 있음 -> Loose Couple(느슨한 결합)
- `상속 대신 위임을 선호하라`라는 말은 상속을 적용하고 언제든지 이런 리팩토링을 사용해 위임으로 전환할 수 있음을 의미
<details>
  <summary>테스트 코드</summary>
  
```java
class BookingTest {
  @Test
  void basePrice() {
    Show lionKing = new Show(List.of(), 120);
    LocalDateTime weekday = LocalDateTime.of(2022, 1, 20, 19, 0);

    Booking booking = new Booking(lionKing, weekday);
    assertEquals(120, booking.basePrice());

    Booking premium = new PremiumBooking(lionKing, weekday, new PremiumExtra(List.of(), 50));
    assertEquals(170, premium.basePrice());
  }

  @Test
  void basePrice_on_peakDay() {
    Show lionKing = new Show(List.of(), 120);
    LocalDateTime weekend = LocalDateTime.of(2022, 1, 15, 19, 0);

    Booking booking = new Booking(lionKing, weekend);
    assertEquals(138, booking.basePrice());

    Booking premium = new PremiumBooking(lionKing, weekend, new PremiumExtra(List.of(), 50));
    assertEquals(188, premium.basePrice());
  }

  @Test
  void talkback() {
    Show noTalkbackShow = new Show(List.of(), 120);
    Show talkbackShow = new Show(List.of("talkback"), 120);
    LocalDateTime nonPeekday = LocalDateTime.of(2022, 1, 20, 19, 0);
    LocalDateTime peekday = LocalDateTime.of(2022, 1, 15, 19, 0);

    assertFalse(new Booking(noTalkbackShow, nonPeekday).hasTalkback());
    assertTrue(new Booking(talkbackShow, nonPeekday).hasTalkback());
    assertFalse(new Booking(talkbackShow, peekday).hasTalkback());

    PremiumExtra premiumExtra = new PremiumExtra(List.of(), 50);
    assertTrue(new PremiumBooking(talkbackShow, peekday, premiumExtra).hasTalkback());
    assertFalse(new PremiumBooking(noTalkbackShow, peekday, premiumExtra).hasTalkback());
  }

  @Test
  void hasDinner() {
    Show lionKing = new Show(List.of(), 120);
    LocalDateTime weekday = LocalDateTime.of(2022, 1, 20, 19, 0);
    LocalDateTime weekend = LocalDateTime.of(2022, 1, 15, 19, 0);
    PremiumExtra premiumExtra = new PremiumExtra(List.of("dinner"), 50);

    assertTrue(new PremiumBooking(lionKing, weekday, premiumExtra).hasDinner());
    assertFalse(new PremiumBooking(lionKing, weekend, premiumExtra).hasDinner());
  }
}
```

</details>

> **AS-IS**: `Booking` 클래스를 상속하는 `PremiumBooking`
```java
public class Booking {

  protected Show show;

  protected LocalDateTime time;

  public Booking(Show show, LocalDateTime time) {
    this.show = show;
    this.time = time;
  }

  public boolean hasTalkback() {
    return this.show.hasOwnProperty("talkback") && !this.isPeakDay();
  }

  protected boolean isPeakDay() {
    DayOfWeek dayOfWeek = this.time.getDayOfWeek();
    return dayOfWeek == DayOfWeek.SATURDAY || dayOfWeek == DayOfWeek.SUNDAY;
  }

  public double basePrice() {
    double result = this.show.getPrice();
    if (this.isPeakDay()) result += Math.round(result * 0.15);
    return result;
  }

}

public class PremiumBooking extends Booking {

  private PremiumExtra extra;

  public PremiumBooking(Show show, LocalDateTime time, PremiumExtra extra) {
    super(show, time);
    this.extra = extra;
  }

  @Override
  public boolean hasTalkback() {
    return this.show.hasOwnProperty("talkback");
  }

  @Override
  public double basePrice() {
    return Math.round(super.basePrice() + this.extra.getPremiumFee());
  }

  public boolean hasDinner() {
    return this.extra.hasOwnProperty("dinner") && !this.isPeakDay();
  }
}
```
- `PremiumBooking`을 Delegate를 통해 서브클래스를 없앨 수 있음

> **TO-BE(1)**: `PremiumDelegate` 생성 및 `Booking` 객체 생성 팩토리 매소드 생성

```java
public class PremiumDelegate {

  private Booking host;
  private PremiumExtra extra;

  public PremiumDelegate(Booking host, PremiumExtra extra) {
    this.host = host;
    this.extra = extra;
  }
}

public class Booking {

  protected PremiumDelegate premiumDelegate;

  public static Booking createBooking(Show show, LocalDateTime time) {
    return new Booking(show, time);
  }

  public static PremiumBooking createPremiumBooking(Show show, LocalDateTime time, PremiumExtra extra) {
    PremiumBooking premiumBooking = new PremiumBooking(show, time, extra);
    premiumBooking.premiumDelegate = new PremiumDelegate(premiumBooking, extra);
    return premiumBooking;
  }
  
  ...
}
```

- 팩토리 메서드를 만든 이유는 이름명명의 이유도 있겠지만, 리턴객체의 타입이 자유로워짐
- `createPremiumBooking`에서 `premiumBooking.premiumDelegate = new PremiumDelegate(premiumBooking, extra);` 작업을 통해 서브클래스와 Delegate를 연결지음
  - 추후에 Delegate로 서브클래스에 있는 코드를 옮김

> **TO-BE(2)**: `hasTalkback` 코드 옮기기
```java
public class Booking {
  public boolean hasTalkback() {
    return (this.premiumDelegate != null)
          ? this.premiumDelegate.hasTalkback()
          : this.show.hasOwnProperty("talkback") && !this.isPeakDay();
  }
}

public class PremiumDelegate {

  public boolean hasTalkback() {
    return this.host.show.hasOwnProperty("talkback");
  }
}

public class PremiumBooking extends Booking {

  // @Override
  // public boolean hasTalkback() {
  //   return this.show.hasOwnProperty("talkback");
  // }
}
```
- `PremiumBooking::hasTalkback`의 역할을 `PremiumDelegate::hasTalkback`이 대신 위임받음
- 그래서 `Booking`의 `hasTalkback`만 호출해줘도 됨

> **TO-BE(3)**: `basePrice` 코드 옮기기
```java
public class PremiumDelegate {
  public double extendBasePrice(double basePrice) {
    return Math.round(basePrice + this.extra.getPremiumFee());
  }
}

public class Booking {
  public double basePrice() {
    double result = this.show.getPrice();
    if (this.isPeakDay()) result += Math.round(result * 0.15);
    return (this.premiumDelegate != null) ? this.premiumDelegate.extendBasePrice(result) : result;
  }
}

public class PremiumBooking extends Booking {

  // @Override
  // public double basePrice() {
  //   return Math.round(super.basePrice() + this.extra.getPremiumFee());
  // }
}
```
- `PremiumBooking::basePrice`는 기존 `Booking::basePrice`에 `Premium Fee`만큼 더해진 값
- 이 역시 `PremiumDelegate::extendBasePrice`에서 계산하도록 수정

> **TO-BE(4)**: `hasDinner` 코드 옮기기 및 `PremiumBooking` 삭제
```java
public class PremiumDelegate {
  public boolean hasDinner() {
    return this.extra.hasOwnProperty("dinner") && !this.host.isPeakDay();
  }
}

public class Booking {
  public boolean hasDinner() {
    return this.premiumDelegate != null && this.premiumDelegate.hasDinner();
  }
}

public class PremiumBooking extends Booking {

  // public boolean hasDinner() {
  //   return this.extra.hasOwnProperty("dinner") && !this.isPeakDay();
  // }
}
```
- `hasDinner`는 `PremiumBooking`에만 있던 메소드 -> `PremiumBooking`만의 행동
  - 즉, `Booking`에선 처리할 필요가 없음
  - 그럼에도 `Booking`에선 구현해줘야함 -> 이는 해당 리팩토링의 사이드 이펙트라고 판단됨
- `PremiumDelegate`로 위임시켜서 메소드 호출
- 이로서 `PremiumBooking`은 서브클래스로서 의미가 없기 떄문에 삭제되어도 됨.
