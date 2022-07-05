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

> **AS-IS**
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
