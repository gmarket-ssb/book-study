## 냄새 4. 긴 매개변수 목록

### 개요
- 어떤 함수에 매개변수가 많아짐 -> 함수의 역할을 이해하기 힘들어짐
    - 과연 그 함수가 한가지 일을 하는가?
    - 불필요한 매개변수는 있는가?
    - 하나의 레코드로 뭉칠 수 있는 매개변수 목록은 없는가?
- 어떤 매개변수를 다른 매개변수를 통해 알아낼 수 있다면 **매개변수를 질의 함수로 바꾸기**를 사용할 수 있음
- 기존 자료구조에서 세부적인 데이터를 가져와서 여러 매개변수로 넘기는 대신 **객체 통째로 넘기기**를 사용항 수 있음
- 일부 매개변수들이 대부분 같이 넘겨진다면, **매개변수 객체 만들기**를 사용할 수 있음
- 매개변수가 플래그로 사용된다면, **플래그 인수 제거하기**를 사용할 수 있음
- 여러 함수가 일부 매개변수를 공통적으로 사용한다면 **여러 함수를 클래스로 묶기**를 통해 매개변수를 해당 클래스의 필드로 만들고 메서드에 전달해야 할 매개변수 목록을 줄일 수 있음

### 매개변수를 질의 함수로 바꾸기

- 함수의 매개변수 목록은 함수의 다양성을 대변하며, 짧을수록 이해하기 좋음
- 임의의 매개변수를 다른 매개변수를 통해 알 수 있다면, **중복 매개변수**로 인식될 수 있음
- 매개변수에 값을 전달하는 것은 **함수 호출하는 쪽** 책임. 가능하면 함수 호출하는 쪽 책임을 줄이고 함수 내부에서 책임지도록 노력
- **임시 변수를 질의 함수로 바꾸기**와 **함수 선언 변경하기**를 통해 이 리팩토링을 적용

#### before
```java
public class Order {

    private int quantity;

    private double itemPrice;

    public Order(int quantity, double itemPrice) {
        this.quantity = quantity;
        this.itemPrice = itemPrice;
    }

    public double finalPrice() {
        double basePrice = this.quantity * this.itemPrice;
        int discountLevel = this.quantity > 100 ? 2 : 1;
        return this.discountedPrice(basePrice, discountLevel);
    }

    private double discountedPrice(double basePrice, int discountLevel) {
        return discountLevel == 2 ? basePrice * 0.9 : basePrice * 0.95;
    }
}
```

#### after
```java
public class Order {

    private int quantity;

    private double itemPrice;

    public Order(int quantity, double itemPrice) {
        this.quantity = quantity;
        this.itemPrice = itemPrice;
    }

    public double finalPrice() {
        double basePrice = this.quantity * this.itemPrice;
        return this.discountedPrice(basePrice);
    }

    private int discountLevel() {
        return this.quantity > 100 ? 2 : 1;
    }

    private double discountedPrice(double basePrice) {
        return discountLevel() == 2 ? basePrice * 0.90 : basePrice * 0.95;
    }
}

```


### 플래그 인수 제거하기

- 플래그는 보통 함수에 매개변수로 전달해서, 함수 내부 로직을 분기하는데 사용
- 플래그를 사용한 함수는 차이를 파악하기 힘듦
- **조건문 분해하기**를 활용할 수 있음

#### before
```java
public class Shipment {

    public LocalDate deliveryDate(Order order, boolean isRush) {
        if (isRush) {
            int deliveryTime = switch (order.getDeliveryState()) {
                case "WA", "CA", "OR" -> 1;
                case "TX", "NY", "FL" -> 2;
                default -> 3;
            };
            return order.getPlacedOn().plusDays(deliveryTime);
        } else {
            int deliveryTime = switch (order.getDeliveryState()) {
                case "WA", "CA" -> 2;
                case "OR", "TX", "NY" -> 3;
                default -> 4;
            };
            return order.getPlacedOn().plusDays(deliveryTime);
        }
    }

    @Test
    void deliveryDate() {
        LocalDate placedOn = LocalDate.of(2021, 12, 15);
        Order orderFromWA = new Order(placedOn, "WA");

        Shipment shipment = new Shipment();
        // 두 번쨰 인자가 무엇인지는 해당 함수를 보지 않는 이상 알 수 없음
        assertEquals(placedOn.plusDays(1), shipment.deliveryDate(orderFromWA, true));
        assertEquals(placedOn.plusDays(2), shipment.deliveryDate(orderFromWA, false));
    }

}

```

#### after
```java
public class Shipment {

    public LocalDate regularDeliveryDate(Order order) {
        int deliveryTime = switch (order.getDeliveryState()) {
            case "WA", "CA" -> 2;
            case "OR", "TX", "NY" -> 3;
            default -> 4;
        };
        return order.getPlacedOn().plusDays(deliveryTime);
    }

    public LocalDate rushDeliveryDate(Order order) {
        int deliveryTime = switch (order.getDeliveryState()) {
            case "WA", "CA", "OR" -> 1;
            case "TX", "NY", "FL" -> 2;
            default -> 3;
        };
        return order.getPlacedOn().plusDays(deliveryTime);
    }

    @Test
    void deliveryDate() {
        LocalDate placedOn = LocalDate.of(2021, 12, 15);
        Order orderFromWA = new Order(placedOn, "WA");

        Shipment shipment = new Shipment();
        // 조건문을 함수레벨로 쪼개서 플래그 함수 없이 어떤 역할을 하는 함수인지 알 수 있음
        assertEquals(placedOn.plusDays(1), shipment.rushDeliveryDate(orderFromWA));
        assertEquals(placedOn.plusDays(2), shipment.regularDeliveryDate(orderFromWA, false));
    }
}

```

### 여러 함수를 클래스로 묶기

- 비슷한 매개변수 목록을 여러 함수에서 사용하고 있다면 해당 메소드를 모아 클래스를 만들 수 있음
- **클래스 내부로 메소드를 옮기고, 데이터를 필드로 만들면** 메소드에 전달해야하는 매개변수 목록도 줄일 수 있음


#### before
```java
public class StudyDashboard {

    private final int totalNumberOfEvents;

    private void print() throws IOException, InterruptedException {
        GitHub gitHub = GitHub.connect();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        List<Participant> participants = new CopyOnWriteArrayList<>();

        /* ... */

        print(participants);
    }

    private void print(List<Participant> participants) throws IOException {     // participants 매개변수 사용
        try (FileWriter fileWriter = new FileWriter("participants.md");
             PrintWriter writer = new PrintWriter(fileWriter)) {
            participants.sort(Comparator.comparing(Participant::username));

            writer.print(header(participants.size()));

            participants.forEach(p -> {
                String markdownForHomework = getMarkdownForParticipant(p.username(), p.homework());
                writer.print(markdownForHomework);
            });
        }
    }

    private String header(int totalNumberOfParticipants) {                  // participants.size() 매개변수 사용
        StringBuilder header = new StringBuilder(String.format("| 참여자 (%d) |", totalNumberOfParticipants));

        for (int index = 1; index <= this.totalNumberOfEvents; index++) {
            header.append(String.format(" %d주차 |", index));
        }
        header.append(" 참석율 |\n");

        header.append("| --- ".repeat(Math.max(0, this.totalNumberOfEvents + 2)));
        header.append("|\n");

        return header.toString();
    }

    private String getMarkdownForParticipant(String username, Map<Integer, Boolean> homework) { // participants 필드값 사용
        return String.format("| %s %s | %.2f%% |\n", username, checkMark(homework), getRate(homework));
    }
}
```
- `print`, `header`, `getMarkdownForParticipant` 함수는 `participants` 리스트의 값으로 이뤄진 매개변수를 사용
- 즉, `participants` 객체를 매개변수를 대신 사용해도 되고, 이 함수들은 하나의 클래스로 묶을 수 있음

#### after
```java
public class StudyDashboard {

    private final int totalNumberOfEvents;

    private void print() throws IOException, InterruptedException {
        GitHub gitHub = GitHub.connect();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        List<Participant> participants = new CopyOnWriteArrayList<>();

        /* ... */

        new StudyPrinter(this.totalNumberOfEvents, participants).print();
    }
}

public class StudyPrinter {

    private int totalNumberOfEvents;

    private List<Participant> participants;

    public StudyPrinter(int totalNumberOfEvents, List<Participant> participants) {
        this.totalNumberOfEvents = totalNumberOfEvents;
        this.participants = participants;
    }

    public void print() throws IOException {
        try (FileWriter fileWriter = new FileWriter("participants.md");
             PrintWriter writer = new PrintWriter(fileWriter)) {
            participants.sort(Comparator.comparing(Participant::username));

            writer.print(header());

            participants.forEach(p -> {
                String markdownForHomework = getMarkdownForParticipant(p.username(), p.homework());
                writer.print(markdownForHomework);
            });
        }
    }

    /**
     * | 참여자 (420) | 1주차 | 2주차 | 3주차 | 참석율 |
     * | --- | --- | --- | --- | --- |
     */
    private String header() {
        StringBuilder header = new StringBuilder(String.format("| 참여자 (%d) |", this.participants.size()));

        for (int index = 1; index <= this.totalNumberOfEvents; index++) {
            header.append(String.format(" %d주차 |", index));
        }
        header.append(" 참석율 |\n");

        header.append("| --- ".repeat(Math.max(0, this.totalNumberOfEvents + 2)));
        header.append("|\n");

        return header.toString();
    }


    private String getMarkdownForParticipant(String username, Map<Integer, Boolean> homework) {
        return String.format("| %s %s | %.2f%% |\n", username, checkMark(homework), getRate(homework));
    }
}
```