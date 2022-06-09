## 냄새 2. 중복코드 (Duplicated Code)

동일한, 비슷한 일을 하는 코드를 잘 들여보면 중복된 부분이 보이고, 이를 개선해 볼 수 있음 -> DAO 등

중복 코드의 단점

- 비슷한지, 완전히 동일한 코드인지 주의 깊게 봐야한다.
- 코드를 변경할 때, 동일한 모든 곳의 코드를 변경해야 한다. 사용할 수 있는 리팩토링 기술
- 동일한 코드를 여러 메소드에서 사용하는 경우, 함수 추출하기 (Extract Function)
- 코드가 비슷하게 생겼지만 완전히 같지는 않은 경우, 코드 분리하기 (Slide Statements)
- 여러 하위 클래스에 동일한 코드가 있다면, 메소드 올리기 (Pull Up Method)

## 함수 추출하기 (Extract Function)

"의도"와 "구현" 분리하기

- 책을 읽듯이 잘 읽히면 -> 구현 -> 예) JDBC Connection 맺기
- 어떤 일을 하는지 잘 이해가 되면 -> 의도 -> 예)도메인 타입으로 변환하기 무슨 일을 하는 코드인지 알아내려고 노력해야 하는 코드(“구현”)라면 해당 코드를 함수로 분리하고 함수 이름으로 "무슨 일을 하는지(
  의도)” 표현할 수 있다. 한줄 짜리 메소드도 괜찮은가? -> 괜찮음 거대한 함수 안에 들어있는 주석은 추출한 함수를 찾는데 있어서 좋은 단서가 될 수 있다.
- > 클린코드에서 나옴

### InteliJ를 이용한 리팩토링 방법
- [extract-method](https://www.jetbrains.com/idea/guide/tips/extract-method/#:~:text=Highlight%20the%20code%20you%20want,the%20readability%20of%20your%20code)

### Before

```java

public class StudyDashboard {

    @Value("token")
    private static String token;

    private void printParticipants(int eventId) throws IOException {

        // TODO: 구현을 읽기 어렵고, 의도를 분리해야함
        // Get github issue to check homework
        GitHub gitHub = new GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(eventId);

        // // TODO: 구현을 읽기 어렵고, 의도를 분리해야함
        // Get participants
        Set<String> participants = new HashSet<>();
        issue.getComments().forEach(c -> participants.add(c.getUserName()));

        // // TODO: 구현을 읽기 어렵고, 의도를 분리해야함
        // Print participants
        participants.forEach(System.out::println);
    }

    private void printReviewers() throws IOException {
        // Get github issue to check homework
        GitHub gitHub = new GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(30);

        // Get reviewers
        Set<String> reviewers = new HashSet<>();
        issue.getComments().forEach(c -> reviewers.add(c.getUserName()));

        // Print reviewers
        reviewers.forEach(System.out::println);
    }

    public static void main(String[] args) throws IOException {

        StudyDashboard studyDashboard = new StudyDashboard();
        studyDashboard.printReviewers();
        studyDashboard.printParticipants(15);

    }

}

```

주석이 설명하는 의도를 담은 메서드로 분리 후 적용

### After - 1

```java
public class StudyDashboard {

    @Value("token")
    private static String token;

    private void printParticipants(int eventId) throws IOException {

        GHIssue issue = getGhIssue(eventId);
        Set<String> participants = getUsernames(issue);
        print(participants);
    }

    private void printReviewers() throws IOException {

        GHIssue issue = getGhIssue(30);
        Set<String> reviewers = getUsernames(issue);
        PrintParticipants(reviewers);
    }

    private void print(Set<String> participants) {

        participants.forEach(System.out::println);
    }

    private Set<String> getUsernames(GHIssue issue) throws IOException {

        Set<String> participants = new HashSet<>();
        issue.getComments().forEach(c -> participants.add(c.getUserName()));
        return participants;
    }

    private GHIssue getGhIssue(int eventId) throws IOException {

        GitHub gitHub = new GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        return repository.getIssue(eventId);
    }

    public static void main(String[] args) throws IOException {

        StudyDashboard studyDashboard = new StudyDashboard();
        studyDashboard.printReviewers();
        studyDashboard.printParticipants(15);

    }
}

```

----

## 코드 정리하기 (Slide Statements)

- 다른 리팩토링을 하기 위한 전처리 과정으로 많이 사용됨

관련있는 코드끼리 묶여있어야 코드를 더 쉽게 이해할 수 있다.

함수에서 사용할 변수를 상단에 미리 정의하기 보다는, 
해당 변수를 사용하는 코드 바로 위에 선언하자. 관련있는 코드끼리 묶은 다음, 함수 추출하기 (
Extract Function)를 사용해서 더 깔끔하게 분리할 수도 있다

```java
public class StudyDashboard {

    @Value("token")
    private static String token;

    private void printParticipants(int eventId) throws IOException {
        // Get github issue to check homework
        Set<String> participants = new HashSet<>();
        GitHub gitHub = new GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(eventId);

        // Get participants
        issue.getComments().forEach(c -> participants.add(c.getUserName()));

        // Print participants
        participants.forEach(System.out::println);
    }

    private void printReviewers() throws IOException {
        // Get github issue to check homework
        // TODO: reviewers를 사용하는 곳과 멀리 떨어져있다. 바꾸자.
        Set<String> reviewers = new HashSet<>();
        GitHub gitHub = new GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(30);

        // TODO: reviewers를 찾기 위해서는 윗쪽으로 올라가야한다. 바꾸자
        // Get reviewers
        issue.getComments().forEach(c -> reviewers.add(c.getUserName()));

        // Print reviewers
        reviewers.forEach(System.out::println);
    }

    public static void main(String[] args) throws IOException {

        StudyDashboard studyDashboard = new StudyDashboard();
        studyDashboard.printReviewers();
        studyDashboard.printParticipants(15);
    }

}
```

`issue.getComments().forEach(c -> reviewers.add(c.getUserName()));` 코드를 읽는 경우 reviewers 가 멀리 떨어져있음. => 붙혀줌

```java

public class StudyDashboard {

    @Value("token")
    private static String token;

    private void printParticipants(int eventId) throws IOException {
        // Get github issue to check homework
        GitHub gitHub = new GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(eventId);

        // Get participants
        Set<String> participants = new HashSet<>();
        issue.getComments().forEach(c -> participants.add(c.getUserName()));

        // Print participants
        participants.forEach(System.out::println);
    }

    private void printReviewers() throws IOException {
        // Get github issue to check homework
        GitHub gitHub = new GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(30);

        // Get reviewers
        Set<String> reviewers = new HashSet<>();
        issue.getComments().forEach(c -> reviewers.add(c.getUserName()));

        // Print reviewers
        reviewers.forEach(System.out::println);
    }

    public static void main(String[] args) throws IOException {

        StudyDashboard studyDashboard = new StudyDashboard();
        studyDashboard.printReviewers();
        studyDashboard.printParticipants(15);

    }

}

```

----

## 메소드 올리기 (Pull Up Method)

중복 코드는 당장은 잘 동작하더라도 미래에 버그를 만들어 낼 빌미를 제공한다.

- 예) A에서 코드를 고치고, B에는 반영하지 않은 경우

여러 하위 클래스에 동일한 코드가 있다면, 손쉽게 이 방법을 적용할 수 있다. 
비슷하지만 일부 값만 다른 경우라면, “함수 매개변수화하기 (Parameterize Function)” 리팩토링을 적용한 이후에, 이
방법을 사용할 수 있다. 

하위 클래스에 있는 코드가 상위 클래스가 아닌 하위 클래스 기능에 의존하고 있다면, “필드 올리기 (Pull Up Field)”를 적용한 이후에 이 방법을 적용할 수 있다. 두 메소드가
비슷한 절차를 따르고 있다면, “템플릿 메소드 패턴 (Template Method Pattern)” 적용을 고려할 수 있다.

### IntelliJ

- [pull-members-up](https://www.jetbrains.com/help/idea/pull-members-up.html)

### Before

```java
public class Dashboard {

    @Value("token")
    protected static String token;

    public static void main(String[] args) throws IOException {

        ReviewerDashboard reviewerDashboard = new ReviewerDashboard();
        reviewerDashboard.printReviewers();

        ParticipantDashboard participantDashboard = new ParticipantDashboard();
        participantDashboard.printParticipants(15);
    }

}


public class ReviewerDashboard extends Dashboard {

    public void printReviewers() throws IOException {

        printUsernames(30);
    }

    //TODO: ParticipantDashboard에도 유사한 메서드가 존재함
    protected void printUsernames(int eventId) throws IOException {
        // Get github issue to check homework
        GitHub gitHub = new GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(eventId);

        // Get usernames
        Set<String> usernames = new HashSet<>();
        issue.getComments().forEach(c -> usernames.add(c.getUserName()));

        // Print usernames
        usernames.forEach(System.out::println);
    }

}


public class ParticipantDashboard extends Dashboard {

    public void printParticipants(int eventId) throws IOException {

        printUsernames(eventId);
    }

    //TODO: ReviewerDashboard에도 유사한 메서드가 존재함
    protected void printUsernames(int eventId) throws IOException {
        // Get github issue to check homework
        GitHub gitHub = new GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(eventId);

        // Get usernames
        Set<String> usernames = new HashSet<>();
        issue.getComments().forEach(c -> usernames.add(c.getUserName()));

        // Print usernames
        usernames.forEach(System.out::println);
    }

}

```

### After

같은 내용의 코드를 Super Class - Dashboard에서 구현 후, Sub Class에서는 이를 호출하여 이용함

```java

public class Dashboard {

    @Value("token")
    protected static String token;

    public static void main(String[] args) throws IOException {

        ReviewerDashboard reviewerDashboard = new ReviewerDashboard();
        reviewerDashboard.printReviewers();

        ParticipantDashboard participantDashboard = new ParticipantDashboard();
        participantDashboard.printParticipants(15);
    }

    protected void printUsernames(int eventId) throws IOException {
        // Get github issue to check homework
        GHIssue issue = getGhIssue(eventId);

        // Get usernames
        Set<String> usernames = getUsername(issue);

        // Print usernames
        usernames.forEach(System.out::println);
    }

    private Set<String> getUsername(GHIssue issue) throws IOException {

        Set<String> usernames = new HashSet<>();
        issue.getComments().forEach(c -> usernames.add(c.getUserName()));
        return usernames;
    }

    private GHIssue getGhIssue(int eventId) throws IOException {

        GitHub gitHub = new GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(eventId);
        return issue;
    }

}


public class ParticipantDashboard extends Dashboard {

    public void printParticipants(int eventId) throws IOException {

        super.printUsernames(eventId);
    }

}


public class ReviewerDashboard extends Dashboard {

    public void printReviewers() throws IOException {

        super.printUsernames(30);
    }

}
```

