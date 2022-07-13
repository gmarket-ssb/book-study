# 냄새 19. 내부자 거래(Insider Trading)

- 어느 모듈이 다른 모듈의 내부 정보를 지나치게 많이 알고 있는 경우
- 적절한 모듈로 "함수 옮기기"와 "필드 옮기기"로 결합도를 낮춘다.
- 여러 모듈이 자주 사용하는 공통적인 기능은 새로운 모듈을 만들어 관리한다.
- 혹은, "위임 숨기기(Hide Delegate)"를 사용해 중재자 모듈을 사용한다.
- 상속으로 인한 결합도를 줄일 때는 "슈퍼클래스 또는 서브클래스 위음으로 교체하기"를 사용한다.

> ### before
```java
public class Ticket {
    
    private LocalDate purchasedDate;

    private boolean prime;

    public Ticket(LocalDate purchasedDate, boolean prime) {
        this.purchasedDate = purchasedDate;
        this.prime = prime
    }

    public LocalDate getPurchasedDate() {
        return purchasedDate;
    }

    public boolean isPrime() {
        return prime;
    }

}

public class CheckIn {

    // Ticket 안의 내부정보를 많이 참조하고 있다.
    public boolean isFastPass(Ticket ticket) {
        LocalDate earlyBirdDate = LocalDate.of(2022, 1, 1);
        return ticket.isPrime() && ticket.getPurchasedDate().isBefore(earlyBirdDate);
    }
}
```

> ### after
```java
public class Ticket {
    
    private LocalDate purchasedDate;

    private boolean prime;

    public Ticket(LocalDate purchasedDate, boolean prime) {
        this.purchasedDate = purchasedDate;
        this.prime = prime
    }

    public LocalDate getPurchasedDate() {
        return purchasedDate;
    }

    public boolean isPrime() {
        return prime;
    }

    // CheckIn 객체에서 Ticket 객체로 옮긴다. => "함수 옮기기"
    public boolean isFastPass() {
        LocalDate earlyBirdDate = LocalDate.of(2022, 1, 1);
        return isPrime() && getPurchasedDate().isBefore(earlyBirdDate);
    }

}
```

# 냄새20. 거대한 냄새 클래스(Large Class)

- 클래스가 커지기 시작하면, 많은 필드와 중복 코드도 보이기 시작한다.
- 클라이언트의 관점에서 해당 클래스의 기능 중 일부만 사용한다면, 각각 세부 기능을 별도의 클래스로 분리할 수 있다.
  - "클래스 추출하기"를 사용해 관련있는 필드를 한 곳으로 모을 수 있다.
  - 상속 구조를 만들 수 있다면 "슈퍼 클래스 추출하기" 또는 "타입 코드를 서브클래스로 교체하기"를 적용할 수 있다.
- 클래스 내부에 산재하는 중복코드는 메소드를 추출하여 제거할 수 있다.

## 리팩토링 41. 슈퍼클래스 추출하기(Extract Superclass)

- 두개의 클래스에서 비슷한 것들이 보인다면 상속을 적용하고, 슈퍼클래스로 "필드 올리기"와 "메서드 올리기"를 사용한다.
- 대안으로는 "클래스 추출하기"를 적용해 위임을 사용할 수 있다.

> ### before
```java
@Getter
public class Department {
    private String name;
    private List<Employee> staff;
    
    public double totalMonthlyCost() {
        return this.staff.stream().mapToDouble(e -> e.getMonthlyCost()).sum();
    }

    public double totalAnnualCost() {
        return this.totalMonthlyCost() * 12;
    }

    public int headCount() {
        return this.staff.size();
    }
}

@Getter
public class Employee {
    private Integer id;
    private String name;
    private double monthlyCost;

    public double annualCost() {
        return this.monthlyCost * 12;
    }
}
```

> 필드 name과, 함수명은 조금 다르지만 cost를 계산하는 함수가 공통적으로 같은 의미를 나타낸다. 따라서, 두 부분을 슈퍼클래스로 추출한다.

> ### after
```java
@Getter
@AllArgsConstructor
public abstract class Party {
    protected String name;

    public double annualCost() {
        return monthlyCost() * 12;
    }

    abstract protected double monthlyCost();
}

@Getter
public class Department extends Party {
    private List<Employee> staff;

    public Department(String name) {
        super(name);
    }

    @Override
    public double monthlyCost() {
        return this.staff.stream().mapToDouble(e -> e.getMonthlyCost()).sum();
    }

    public int headCount() {
        return this.staff.size();
    }
}

@Getter
public class Employee extends Party {
    private Integer id;
    private double monthlyCost;

    public Employee(String name) {
        super(name);
    }

    @Override
    public double monthlyCost() {
        return monthlyCost;
    }
}
```

# 냄새 21. 서로 다른 인터페이스의 대안 클래스들(Alternative Classes with Different Interfaces)

- 비슷한 일을 여러 곳에서 서로 다른 규약을 사용해 지원하고 있는 코드 냄새
- 대안 클래스로 사용하려면 동일한 인터페이스를 구현해야 한다.
- "함수 선언 변경하기"와 "함수 옮기기"를 사용해서 서로 동일한 인터페이스를 구현하게끔 코드를 수정할 수 있다.
- 두 클래스에서 일부 코드가 중복되는 경우에는 "슈퍼클래스 추출하기"를 사용해 중복된 코드를 슈퍼클래스로 옮기고 두 클래스를 새로운 슈퍼클래스의 서브클래스로 만들 수 있다.

> ### before
```java
public interface EmailService {
    void sendEmail(EmailMessage emailMessage);
}

public interface AlertService {
    void add(AlertMessage alertMessage);
}

public class OrderProcessor {
    private EmailService emailService;

    public void notifyShipping(Shipping shipping) {
        EmailMessage emailMessage = new EmailMessage();
        emailMessage.setTitle(shipping.getOrder() + " is shipped");
        emailMessage.setTo(shipping.getEmail());
        emailMessage.setFrom("no-reply@whiteship.com");
        
        emailService.sendEmail(emailMessage);
    }
}

public class OrderAlerts {
    private AlertService alertService;

    public void alertShipped(Order order) {
        AlertMessage alertMessage = new AlertMessage();
        alertMessage.setMessage(order.toString() + " is shipped");
        alertMessage.setFor(order.getEmail());

        alertService.add(alertMessage);
    }
}
```

> EmailService와 AlertService의 인터페이스는 서로 다른 규약이며, 고치기 난감하다.
> > 따라서, 상위 인터페이스로 한번 더 감싸 리팩토링을 진행한다.

> ### after

```java
@Builder
@Getter
public class Notification {
    private String title;
    private String receiver;
    private String sender;
}

// EmailService와 AlertService를 감싸기 위한 상위 규약
public interface NotificationService {
    void sendNotification(Notification notification);
}

public interface EmailService {
    void sendEmail(EmailMessage emailMessage);
}

public interface AlertService {
    void add(AlertMessage alertMessage);
}

public class EmailNotificationService implements NotificationService {
    private EmailService emailService;

    @Override
    public void sendNotification(Notification notification) {
        EmailMessage emailMessage = new EmailMessage();
        emailMessage.setTitle(notification.getTitle());
        emailMessage.setTo(notification.getReceiver());
        emailMessage.setFrom(notification.getSender());
        
        emailService.sendEmail(emailMessage);
    }
}

public class AlertNotificationService implements NotificationService {
    private AlertService alertService;

    @Override
    public void sendNotification(Notification notification) {
        AlertMessage alertMessage = new AlertMessage();
        alertMessage.setMessage(notification.getTitle());
        alertMessage.setFor(notification.getReceiver());

        alertService.add(alertMessage);
    }
}

public class OrderProcessor {
    private NotificationService notificationService;

    public OrderProcessor(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void notifyShipping(Shipping shipping) {
        Notification notification = Notification.builder()
            .title(shipping.getOrder() + " is shipped")
            .receiver(shipping.getEmail())
            .sender("no-reply@whiteship.com")
            .build();

        notificationService.sendNotification(notification);
    }
}

public class OrderAlerts {
    private NotificationService notificationService;
 
    public OrderAlerts(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    private void alertShipped(Order order) {
        Notification notification = Notification.builder()
            .title(order.toString() + " is shipped")
            .for(order.getEmail())
            .build();
        notificationService.sendNotifiaction(notification);
    }
}
```