## Section14. 성의없는 요소

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


> Before : customer.getName().equals("unknown") 의 반복, 별도의 클래스로 추출

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

> After

```java
...
            participants.forEach(p -> {
                String markdownForHomework = getMarkdownForParticipant(totalNumberOfEvents, p);
                writer.print(markdownForHomework);
            });
```
- Refactoring #1 : markdownForHomework에 할당되는 긴영역을 Query함수(질의하는함수)로 추출한다. getMarkdownForParticipant
- Refactoring #2 : count, rate 변수에 할당되는 영역이 특정함수에만 사용되므로 추출한함수내로 이동한다. 

## 리팩토링 8. 매개변수 객체 만들기

### 중복되는 parameter 들을 객채화 
Tips. IDE 빠른메뉴 : Refoctoring 기능 활용( Refactor -> introduce parameter object )

### 중복되는 parameter 들을 field화
여러 함수에서 사용되는 field 가 있다면 클래스 변수(&Constructor) field화 해서 적용

> __Before__ : "int totalNumberOfEvents, Participant p" 매개변수가 중복되어 확인되고 있다.

```java
    private double getRate(int totalNumberOfEvents, Participant p) {
        long count = p.homework().values().stream()
                .filter(v -> v == true)
                .count();
        double rate = count * 100 / totalNumberOfEvents;
        return rate;
    }

    private String getMarkdownForParticipant(int totalNumberOfEvents, Participant p) {
        return String.format("| %s %s | %.2f%% |\n", p.username(), checkMark(p, totalNumberOfEvents), getRate(totalNumberOfEvents, p));
    }
```

> After : function 들의 paramter 좀 더 간소화 되었다.

```java
    //Class varialble 생성
    private final int totalNumberOfEvents;
    //생성자
    public StudyDashboard(int totalNumberOfEvents) {
        this.totalNumberOfEvents = totalNumberOfEvents;
    }
    ...
    // parameter 감소
    private String header(int totalNumberOfParticipants) { 
    ...
  
```
- Refactoring #1 : IDE 빠른메뉴 : Refoctoring 기능 활용( Refactor -> introduce parameter object )
- Refactoring #2 : IDE 빠른메뉴 : Refoctoring 기능 활용( Refactor -> introduce field ), 클래스 변수로 생성
  - Tips. 'this.'를 붙여서 local 변수인지, class 변수인지를 구분하는 습관.

## 리팩토링 9. 객체 통째로 넘기기

- 어떤 한 레코드에서 구할 수 있는 여러 값들을 함수에 전달하는 경우, 해당 매개변수를 레 코드 하나로 교체할 수 있다.
- 매개변수 목록을 줄일 수 있다. (향후에 추가할지도 모를 매개변수까지도...) 
- 이 기술을 적용하기 전에 의존성을 고려해야 한다.
- 어쩌면 해당 메소드의 위치가 적절하지 않을 수도 있다. (기능 편애 “Feature Envy” 냄새에 해당한다.)

> __Before__ : participants p에서 값을 계속 꺼내고 있다. 

```java
    participants.forEach(p -> {
        String markdownForHomework = getMarkdownForParticipant(p.username(), p.homework());
        writer.print(markdownForHomework);
    });
```
> __After__ : paramter list가 간소화 되었다.

```java
     participants.forEach(p -> {
          String markdownForHomework = getMarkdownForParticipant(p);
          writer.print(markdownForHomework);
      });
  ...
  
```

### 추가로 고민할문제
1. 의존성 
   - 함수를 일반화된 데이터 Map<K, V>를 param으로 받는게 맞는가, 아니면 위의 obj를 받는게 맞는가.
   - 다른곳에서도 사용하는가? 이함수를 다른 도메인에도 적용할것인가?에 따라 그냥 놔둘수 있다.
2. 과연이위치가 적절한가? 다른 도메인에도 사용할것인가? ( Refactor => Move Method )
   -  밖으로 추출(util성)한다면 Normalize할 필요가 있다.
   -  
> __Before__ : getRate를 이 클래스안애서만 사용하겠는가? 부분 Normalize
```java
    double getRate(Map<Integer, Boolean> homework) {
        long count = homework.values().stream()
                .filter(v -> v == true)
                .count();
        return (double) (count * 100 / this.totalNumberOfEvents);
    }
 ```
 > __After__ : Participant로 추후 record에서 별도의 Class로 고려가능한 형태로 변경됨
 ```java
    public double getRate(double total) {
        long count = this.homework.values().stream()
                .filter(v -> v == true)
                .count();
        return count * 100 / total;
    }
 ```
 
## 리펙도링 10. 함수를 명령으로 바꾸기

- 함수를 독립적인 객체인, Command로 만들어 사용할 수 있다. 
- 커맨드 패턴을 적용하면 다음과 같은 장점을 취할 수 있다.( 참고 : https://gmlwjd9405.github.io/2018/07/07/command-pattern.html )
- 부가적인 기능으로 undo 기능을 만들 수도 있다.
- 복잡한 기능을 구현해야한다면.....더 복잡한 기능을 구현하는데 필요한 여러 메소드를 추가할 수 있다. 상속이나 템플릿을 활용할 수도 있다.(check)
- 복잡한 메소드를 여러 메소드나 필드를 활용해 쪼갤 수도 있다.
- 대부분의 경우에 “커맨드” 보다는 “함수”를 사용하지만, 커맨드 말고 다른 방법이 없는 경우에만 사용 한다.

> __Before__ : 앞으로 여러기능을 추가될 여지가 있는 코드를 Command Pattern을 이용하여 추출.( Markdown, Console, Excel등 앙한 출력을 할수 있게 만들수 있다. )

```java
      try (FileWriter fileWriter = new FileWriter("participants.md");
          PrintWriter writer = new PrintWriter(fileWriter)) {
          participants.sort(Comparator.comparing(Participant::username));

          writer.print(header(participants.size()));

          participants.forEach(p -> {
              String markdownForHomework = getMarkdownForParticipant(p);
              writer.print(markdownForHomework);
          });
      }
```
> __After__ : 별도의 Class StudyPrinter를 만들어 이동함

```java
      new StudyPrinter(this.totalNumberOfEvents, participants).execute();
  
```
- More refactoring
> __Before__ :  함수로 추출 + inline refactoring
```java
    for (GHIssueComment comment : comments) {
        //## refactoring 영역 ##########
        String username = comment.getUserName();  //#2. inline refactoring
        // #1. 함수로 추출
        boolean isNewUser = participants.stream().noneMatch(p -> p.username().equals(username));
        Participant participant = null;
        if (isNewUser) {
            participant = new Participant(username);
            participants.add(participant);
        } else {
            participant = participants.stream().filter(p -> p.username().equals(username)).findFirst().orElseThrow();
        }
        //## refactoring 영역 ##########
        participant.setHomeworkDone(eventId);
    }
```
> __After__ 
```java
    for (GHIssueComment comment : comments) {
        Participant participant = findParticipant(comment.getUserName(), participants);
        participant.setHomeworkDone(eventId);
    }
```

## 리팩토링 11. 조건문 분해하기

- 조건문이 복잡해지는 경우 또는 조건 자체가 복잡한경우, true or false가 해야할일이 길어질경우 고려하는 것이다.
- 여러 조건에 따라 달라지는 코드를 작성하다보면 종종 긴 함수가 만들어지는 것을 목격할 수 있다.
- “조건”과 “액션” 모두 “의도”를 표현해야한다.
-  기술적으로는 “함수 추출하기”와 동일한 리팩토링이지만 의도만 다를 뿐이다.

> __Before__ :  무슨코드인지 이해가  어렵다.
```java
    private Participant findParticipant(String username, List<Participant> participants) {
        Participant participant;
        if (participants.stream().noneMatch(p -> p.username().equals(username))) {
            participant = new Participant(username);
            participants.add(participant);
        } else {
            participant = participants.stream().filter(p -> p.username().equals(username)).findFirst().orElseThrow();
        }

        return participant;
    }
```
> __Middle__ : 함수로 추출 및 함수명을 통해서 이해하기 쉽도록 refactoring, 명확하게 함수이름을 통해서 어떤 동작을하고싶은지 가독성이 높아진다.
```java : 
    private Participant findParticipant(String username, List<Participant> participants) {
        Participant participant;
        if (isNewParticipant(username, participants))) {
            participant = createNewParticipant(username, participants)
        } else {
            participant = findExistingParticipant(username, participants);
        }

        return participant;
    }
```
> __After__ : 추가 refactoring if( A ) else (B) => A ? B : C
```java : 
      return isNewParticipant(username, participants) ?
              createNewParticipant(username, participants) :
              findExistingParticipant(username, participants);
```

## 리팩토링 12. 반복문 쪼개기

- O(N)의 입장에서도 N or N^2이냐는 다른문제이다.
- 하나의 반복문에서 여러 다른 작업을 하는 코드를 쉽게 찾아볼 수 있다. 
- 해당 반복문을 수정할 때 여러 작업을 모두 고려하며 코딩을 해야한다. 
- 반복문을 여러개로 쪼개면 보다 쉽게 이해하고 수정할 수 있다.
- 성능 문제를 야기할 수 있지만, “리팩토링”은 “성능 최적화”와 별개의 작업이다. 리팩토링을 마친 이후에 성능 최적화 시도할 수 있다.
- 코드를 수정할때 반복문이 길면 for문안에서 일어나는 여러가지 condition을 고려해야함.( 우리 code패턴.... )
- 각각의 반복문마다 쪼개놓고 정리하자

> __Before__
```java
     for (GHIssueComment comment : comments) {
              Participant participant = findParticipant(comment.getUserName(), participants);
              participant.setHomeworkDone(eventId);

              if (firstCreatedAt == null || comment.getCreatedAt().before(firstCreatedAt)) {
                  firstCreatedAt = comment.getCreatedAt();
                  first = participant;
              }
          }
```
> __Middle__ : 하나의 for 문당 하나의 작업만 하도록 변경 
```java : 
     for (GHIssueComment comment : comments) {
              Participant participant = findParticipant(comment.getUserName(), participants);
              participant.setHomeworkDone(eventId);

          }
          
     for (GHIssueComment comment : comments) {
              Participant participant = findParticipant(comment.getUserName(), participants);

              if (firstCreatedAt == null || comment.getCreatedAt().before(firstCreatedAt)) {
                  firstCreatedAt = comment.getCreatedAt();
                  first = participant;
              }
          }
```

> __After__ : 함수로 추출함 + introduce field, class 변수로 이동 
```java : 
      checkHomework(issue.getComments(), participants, eventId);
      findFirst(comments, participants)
```

### Refactoing Tips
하나의 for문에서 하나의 작업만하도록 쪼갠다. 그리고 각각을 함수로 쪼갠다. (intellij의 함수만들어주는 기능 활용, 인라인리펙토링활용 )
introduce field를 활용

## 리팩토링 13. 조건문을 다형성으로 바꾸기

- 여러 타입에 따라 각기 다른 로직으로 처리해야 하는 경우에 다형성을 적용해서 조건문을 보다 명확하게 분리할 수 있다. (예, 책, 음악, 음식 등...) 반복되는 switch문을 각기 다른 클래스를 만들어 제거할 수 있다.
- 공통으로 사용되는 로직은 상위클래스에 두고 달라지는 부분만 하위클래스에 둠으로써, 달라지는 부분만 강조할 수 있다.
- 모든 조건문을 다형성으로 바꿔야 하는 것은 아니다.
- 다형성 overriding!!

> __Before__ : StudyPrinter class 내의 Switch case 문
```java : 
public void execute() throws IOException {
        switch (printerMode) {
            case CVS -> {
                  ...
            }
            case CONSOLE -> {
                  ...
            }
            case MARKDOWN -> {
                  ...
                }
            }
        }
    }
```

> __After1__ : StudyPrinter 를 추상클래스로 선언, 추상 Method로 변경,
```java : StudyPrinter 생성
  public abstract class StudyPrinter {
    public abstract void execute() throws IOException;
  }
```
> __After2__ : StudyPrinter 를 추상클래스를 상속받는 MarkdownPrinter, CSVPrinter, ConsolePrinter등을 생성
```java : StudyPrinter 생성
    private void print() throws IOException, InterruptedException {
        checkGithubIssues(getGhRepository());
        new MarkdownPrinter(this.totalNumberOfEvents, this.participants, PrinterMode.MARKDOWN).execute(); // or new CSVPrinter() or ...
    }
```
### More Refactoing
factory method pattern을 사용하는 방법도 동적으로 바꿔줄수 있기 때문에 좋은 방법이다.
하지만 어차피 switch case문을 어디선가 작성해야하기 때문에 request 기반으로 동적으로 변경 시에는 이게 더 맞고,
지금 같은 정적인 코드는 직접 생성해서 코드하는게 맞다. 

