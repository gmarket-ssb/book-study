## 냄새 4. 긴 매개변수 목록

### 개요
- 프로그래밍을 시작했을 때에는 전역 변수 사용을 막기 위해, 필요한 모든 데이터를 매개변수로 전달하곤 했음
- 어떤 함수에 매개변수가 많아짐 -> 함수의 역할을 이해하기 힘들어짐
- 클린코드에서도 가장 이상적인 매개변수 갯수는 0개
    - 3 ~ 4개 이상부터는 특별한 이유가 없는 이상 사용하면 안됨

### 매개변수를 질의 함수로 바꾸기

- 함수의 매개변수 목록은 함수의 다양성을 대변하며, 짧을수록 이해하기 좋음
- 임의의 매개변수를 다른 매개변수를 통해 알 수 있다면, **중복 매개변수**로 인식될 수 있음
- 매개변수에 값을 전달하는 것은 **함수 호출하는 쪽** 책임. 가능하면 함수 호출하는 쪽 책임을 줄이고 함수 내부에서 책임지도록 노력
    - 히지만, 매개변수를 줄이면서 새로운 의존성이 생긴다면 다르게 고민할 필요가 있음
- **임시 변수를 질의 함수로 바꾸기**와 **함수 선언 변경하기**를 통해 이 리팩토링을 적용

> **AS-IS**: `discountedPrice`의 매개변수인 `discountLevel`는 필드인 `quantity`를 사용
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

> **TO-BE**: `discountLevel` 매개변수 대신, 질의함수인 `discountLevel()` 생성. 따라서 `discountLevel` 매개변수 삭제
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
- `basePrice`도 필드인 `quantity`와 `itemPrice`를 사용. 최적화한다면 `basePrice`까지도 가능

### 플래그 인수 제거하기

- 플래그는 보통 함수에 매개변수로 전달해 함수 내부 로직을 분기하는데 사용
    - boolean, enum, ...
- 플래그를 사용한 함수는 차이를 파악하기 힘듦
- 플래그 매개변수가 많다 -> 해당 함수의 역할이 많다는 뜻
- 함수를 호출하는 쪽에서는 함수를 사용하기 위해 플래그의 의미를 알아야 함
    - IDE 덕에 쉽게 알 수 있지만, 함수의 구현부를 살펴봐야 함
    - IntelliJ에선 macos 기준 `Option + Command + B`를 통해 구현부로 이동할 수 있음
- **조건문 분해하기**를 활용할 수 있음

> **AS-IS**: `deliveryDate`내 `isRush` 플래그를 기준으로 다른 로직이 존재
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

> **TO-BE**: 플래그 대신 조건문을 기준으로 함수로 나눔. Caller 쪽에선 플래그 없이 함수 이름을 가지고 구분 가능
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
        assertEquals(placedOn.plusDays(2), shipment.regularDeliveryDate(orderFromWA));
    }
}

```

### 여러 함수를 클래스로 묶기

- 동일한 매개변수를 사용하는 함수라면 옮기기가 쉬움
- 비슷한 매개변수 목록을 여러 함수에서 사용하고 있다면, 이 역시 해당 메소드를 모아 클래스를 만들 수 있음
    - **클래스 내부로 메소드를 옮기고, 데이터를 필드로 만들면** 메소드에 전달해야하는 매개변수 목록도 줄일 수 있음


> **AS-IS**: `print`, `header`, `getMarkdownForParticipant` 함수의 매개변수는 `participants`에서 뽑은 값으로 구성
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

> **TO-BE**: 즉, `participants`를 필드로 구성. 해당 함수에선 필드에 직접 접근하여 사용 가능
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
