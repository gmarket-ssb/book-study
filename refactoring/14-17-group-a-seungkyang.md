## 냄새14. 성의없는 요소

### Why?
- 변수, 메소드, 클래스를 만들때 오히려 효율적이라고 생각하지만 비효율적인 경우도 발생한다.

- 관련리팩토링
  - 함수 인라인 (Inline Function)”
  - “클래스 인라인 (Inline Class)”
  - 불필요한 상속 구조는 “계층 합치기 (Collapse Hierarchy)”를 사용할 수 있다.
 

### 리팩토링 34. 계층합치기
상속 구조를 리팩토링하는 중에 기능을 올리고 내리다 보면 하위클래스와 상위클래스 코드 에 차이가 없는 경우가 발생할 수 있다. 그런 경우에 그 둘을 합칠 수 있다.
하위클래스와 상위클래스 중에 어떤 것을 없애야 하는가? (둘 중에 보다 이름이 적절한 쪽 을 선택하지만, 애매하다면 어느쪽을 선택해도 문제없다.)

-  tips : IDE refactoring move to up or down
<img width="575" alt="image" src="https://user-images.githubusercontent.com/5934737/177666341-a3a42386-18c4-488d-a488-1133cb92f349.png">

> Before(Tip : IDE refactoring move to up or down ) 
<img width="569" alt="image" src="https://user-images.githubusercontent.com/5934737/177666916-2a96ec81-7afe-4851-b11e-0a74a5985050.png">

> After
<img width="579" alt="image" src="https://user-images.githubusercontent.com/5934737/177666939-9a1375b1-b9f4-4f2f-943e-62f5beac32ef.png">

## Section15. 추측성 일반화
- 나중에 이러 저러한 기능이 생길 것으로 예상하여, 여러 경우에 필요로 할만한 기능을 만들어 놨지만 “그런 일은 없었고...”결국에 쓰이지 않는 코드가 발생한 경우.
- XP의 YAGNI (You aren’t gonna need it) 원칙 : 지금당장필요한게 아니면 만들지마!!!!

- 관련 리팩토링
  - 추상 클래스를 만들었지만 크게 유요하지 않다면 “계층 합치기 (Collapse Hierarchy)”
  - 불필요한 위임은 “함수 인라인 (Inline Function)” 또는 “클래스 인라인 (Inline Class)”
  - 사용하지 않는 매개변수를 가진 함수는 “함수 선언 변경하기 (Change Function Declaration)” 오로지 테스트 코드에서만 사용하고 있는 코드는 “죽은 코드 제거하기 (Remove Dead Code)”
### 리팩토링 35. 죽은 코드 제거하기 Remove Dead Code
사용하지 않는 코드가 애플리케이션 성능이나 기능에 영향을 끼치지는 않는다.
하지만, 해당 소프트웨어가 어떻게 동작하는지 이해하려는 사람들에게는 꽤 고통을 줄 수있다.

> Before & After : 해당 코드는 테스트 코드에서만 사용하고 있다. 삭제조치!(Tip : IDE usage 확인 ) 
<img width="799" alt="image" src="https://user-images.githubusercontent.com/5934737/177667498-ec183fe8-7ffc-4cc2-ad09-51a550b76a61.png">


## 냄새16. 임시필드 Temporary Field
- 클래스에 있는 어떤 필드가 특정한 경우에만 값을 갖는 경우.
- 어떤 객체의 필드가 “특정한 경우에만” 값을 가진다는 것을 이해하는 것은 일반적으로 예상하지 못하기때문에 이해하기 어렵다. 

- 관련 리팩토링
  - “클래스 추출하기 (Extract Class)”를 사용해 해당 변수들을 옮길 수 있다.
  - “함수 옮기기 (Move Function)”을 사용해서 해당 변수를 사용하는 함수를 특정 클래스로 옮길 수 있 다.
  - “특이 케이스 추가하기 (Introduce Special Case)”를 적용해 “특정한 경우”에 해당하는 클래스를 만들어 해당 조건을 제거할 수 있다.

### 리팩토링 36. 특이 케이스 추가하기 Introduce Special Case
- 어떤 필드의 특정한 값에 따라 동일하게 동작하는 코드가 반복적으로 나타난다면, 해당 필 드를 감싸는 “특별한 케이스”를 만들어 해당 조건을 표현할 수 있다.
- 이러한 매커니즘을 “특이 케이스 패턴”이라고 부르고 “Null Object 패턴”을 이러한 패턴의 특수한 형태라고 볼 수 있다.


> Before&After1 : customer.getName().equals("unknown") 의 반복, 별도의 클래스로 추출
```java
public class UnknownCustomer extends Customer {

    public UnknownCustomer() {
        super("unknown",null , null);
    }
}
```

> Before 2 : customer.getName().equals("unknown") extract method를 이용해서 별도의 클래스로 추출한다. ( IDE refactor : Extract method )

```java
public class CustomerService {

    public String customerName(Site site) {
        Customer customer = site.getCustomer();

        String customerName;
        if (customer.getName().equals("unknown")) {
            customerName = "occupant";
        } else {
            customerName = customer.getName();
        }

        return customerName;
    }

    public BillingPlan billingPlan(Site site) {
        Customer customer = site.getCustomer();
        return customer.getName().equals("unknown") ? new BasicBillingPlan() : customer.getBillingPlan();
    }

    public int weeksDelinquent(Site site) {
        Customer customer = site.getCustomer();
        return customer.getName().equals("unknown") ? 0 : customer.getPaymentHistory().getWeeksDelinquentInLastYear();
    }

}
```

> After 2 

```java
   public String customerName(Site site) {

        Customer customer = site.getCustomer();

        String customerName;
        if (customer.isUnknown()) {
            customerName = "occupant";
        } else {
            customerName = customer.getName();
        }

        return customerName;
    }

    public BillingPlan billingPlan(Site site) {
        Customer customer = site.getCustomer();
        return customer.isUnknown() ? new BasicBillingPlan() : customer.getBillingPlan();
    }

    public int weeksDelinquent(Site site) {
        Customer customer = site.getCustomer();
        return customer.isUnknown() ? 0 : customer.getPaymentHistory().getWeeksDelinquentInLastYear();
    }
    private boolean isUnknown(Customer customer) {
        return customer.getName().equals("unknown");
    }
```

> Before 3 : CustomerService의 isUnknown의 책임은 어느클래스에 잇는것이 맞는가? move to customer class ( IDE refactor : move to instance  )
```java
public class CustomerService {
...
    private boolean isUnknown(Customer customer) {
        return customer.getName().equals("unknown");
    }
...
}
```
> After 3 : CustomerService의 isUnknown의 책임은 어느클래스에 잇는것이 맞는가? move to customer class ( IDE refactor : move to instance  )
```java 
public class Customer {
 public boolean isUnknown() {
        return getName().equals("unknown");
    }
}
```
> Before&After 4 : 이름이 잇을것이기 때문에 반드시 true를 return하도록함
```java 
public class Customer {
 public boolean isUnknown() {
        return false;
    }
}
```

> Before&After 5 :  자 이제 customer가 known인지는 customer 가 판단
```java 
public class UnknownCustomer extends Customer {
   @Override
    public boolean isUnknown() {
        return true;
    }
}
```
> Before 6 :  그럼 customer는 site에서 결정을 지어줘야한다.
```java 
    public Site(Customer customer) {
        this.customer = customer;
    }
```
> After 6 : 
```java 
public class Site {
    public Site(Customer customer) {

        this.customer = customer.getName().equals("unknown") ? new UnknownCustomer() : customer;
    }
}
```
(* Tip : refactoring 중간중간 계속 Test를 수행하여 문제가없는지 확인한다. )

> Before 7 :  customerName = "occupant"; 부분을 각 클래스의 getName을 호출하도록 변경
```java 
public class CustomerService {

    public String customerName(Site site) {
        Customer customer = site.getCustomer();

        String customerName;
        if (customer.getName().equals("unknown")) {
            customerName = "occupant";
        } else {
            customerName = customer.getName();
        }

        return customerName;
    }
    ...
}
```
> After 7 : 
```java 
public class UnknownCustomer extends Customer {
    ...
    @Override
    public String getName() {
        return "occupant";
    }
    ...
}
```

> Before&After 8 :  getName만 호출하도록 변경
```java 
public class CustomerService {
    ...
    public String customerName(Site site) {
        return site.getCustomer().getName();
//before
//        Customer customer = site.getCustomer();
//
//        String customerName;
//        if (customer.isUnknown()) {
//            customerName = "occupant";
//        } else {
//            customerName = customer.getName();
//        }
//
//        return customerName;
    }
    ...
}
```
> Before&After 9 :  billingPlan도 로직이 있을 필요가 없다.
```java 
public class CustomerService {
   public BillingPlan billingPlan(Site site) {
        return site.getCustomer().getBillingPlan();
//before
//        Customer customer = site.getCustomer();
//        return customer.isUnknown() ? new BasicBillingPlan() : customer.getBillingPlan();
    }
}

public class UnknownCustomer extends Customer {
    ...
    public UnknownCustomer() {
        super("unknown", new BasicBillingPlan() , new NullPaymentHistory());
    }
    ...
```
> Before 10 :  Null Object pattern의 적용, 0을 리턴하는 부분을 NullPaymentHistory를 적용함
```java 
public class CustomerService {
    public int weeksDelinquent(Site site) {
        Customer customer = site.getCustomer();
        return customer.isUnknown() ? 0 : customer.getPaymentHistory().getWeeksDelinquentInLastYear();
    }
    ...
```
> After 10-1: NullPaymentHistory class 생성
```java 
public class NullPaymentHistory extends PaymentHistory{

    public NullPaymentHistory() {
        super(0);
    }
}

```
> After 10-2: UnknownCustomer 생성자를 변경
```java 

public class UnknownCustomer extends Customer {

    public UnknownCustomer() {
        super("unknown", new BasicBillingPlan() , new NullPaymentHistory());
    }

```
> After 10-3: weeksDelinquent 로직을 단순화
```java 
public class CustomerService {
    public int weeksDelinquent(Site site) {
        return site.getCustomer().getPaymentHistory().getWeeksDelinquentInLastYear();
//before : null object pattern의 적용 : Null Payment HIstory로 추출한다.
//        Customer customer = site.getCustomer();
//        return customer.isUnknown() ? 0 : customer.getPaymentHistory().getWeeksDelinquentInLastYear();
    }
   ...
```

## 냄새 17. 메시지 체인 Message Chains

- 예) this.member.getCredit().getLevel().getDescription()
- 체인 중 일부가 변경된다면 클라이언트의 코드도 변경해야 한다. 
- 관련 리팩토링
  - “위임 숨기기 (Hide Delegate)”를 사용해 메시지 체인을 캡슐화를 할 수 있다.
  - “함수 추출하기 (Extract Function)”로 메시지 체인 일부를 함수로 추출한 뒤, “함수 옮기기 (Move Function)”으로 해당 함수를 적절한 이동할 수 있다.

### 리팩토링 37. 위임 숨기기 Hide Delegate
- 캡슐화 (Encapsulation)란 어떤 모듈이 시스템의 다른 모듈을 최소한으로 알아야 한다는 것이다. ( 최소한으로 알아야한다!!! )
- 그래야 어떤 모듈을 변경할 때, 최소한의 모듈만 그 변경에 영향을 받을 것이고, 그래야 무언 가를 변경하기 쉽다.
- 처음 객체 지향에서 캡슐화를 배울 때 필드를 메소드로 숨기는 것이라 배우지만, 메소드 호출도 숨길 수 있다.

- 처음 객체 지향에서 캡슐화를 배울 때 필드를 메소드로 숨기는 것이라 배우지만, 메소드 호출도 숨길 수 있다.
- person.department().manager(); -> person.getManager()
- 이전의 코드는 Department를 통해 Manager에 접근할 수 있다는 정보를 알아야 하지만, getManager()를 통해 위임을 숨긴다면 클라이언트는 person의 getManager()만 알아도 된다. 
- 나중에 getManager() 내부 구현이 바뀌더라도 getManager()를 사용한 코드는 그대로 유지할 수 있다.

> Before :  Null Object pattern의 적용, 0을 리턴하는 부분을 NullPaymentHistory를 적용함
```java 
class PersonTest {

    @Test
    void manager() {
        Person keesun = new Person("keesun");
        Person nick = new Person("nick");
        keesun.setDepartment(new Department("m365deploy", nick));

        Person manager = keesun.getDepartment().getManager();
        assertEquals(nick, manager);
    }

}
    ...
```
> After 1-1: Extract method 수행
```java 
class PersonTest {

    @Test
    void manager() {
        Person keesun = new Person("keesun");
        Person nick = new Person("nick");
        keesun.setDepartment(new Department("m365deploy", nick));

        Person manager = getManager(keesun);
        assertEquals(nick, manager);
    }

    private Person getManager(Person keesun) {
        return keesun.getDepartment().getManager();
    }

}
```

> After 1-1: Extract method 수행
```java 
class PersonTest {

    @Test
    void manager() {
        Person keesun = new Person("keesun");
        Person nick = new Person("nick");
        keesun.setDepartment(new Department("m365deploy", nick));

        Person manager = getManager(keesun);
        assertEquals(nick, manager);
    }

    private Person getManager(Person keesun) {
        return keesun.getDepartment().getManager();
    }

}
```
> After 1-2: Person class 로 move instance
<img width="800" alt="image" src="https://user-images.githubusercontent.com/5934737/177684656-d4ccb551-24ca-4690-b972-7d0b6a96b541.png">

> After 2 : Person class 로 위임되었다.
```java 
public class Person {
    ...
    public Person getManager() {
        return getDepartment().getManager();
    }
```
