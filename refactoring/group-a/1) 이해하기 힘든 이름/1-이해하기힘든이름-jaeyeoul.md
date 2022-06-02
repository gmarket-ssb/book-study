# 1주차

## 냄새 1. 이해하기 힘든 이름 Mysterious Name

- 깔끔한 코드에서 가장 중요한 것 중 하나가 바로 “좋은 이름”이다.
- 함수, 변수, 클래스, 모듈의 이름 등 모두 어떤 역할을 하는지 어떻게 쓰이는지 직관적이어야 한다.

### 사용할 수 있는 리팩토링 기술

- 함수 선언 변경하기 (Change Function Declaration)
- 변수 이름 바꾸기 (Rename Variable)
- 필드 이름 바꾸기 (Rename Field)

이름 작성은 최초 개발시에는 항상 ‘좋게’ 만들기는 힘들다.

- 리팩토링 과정을 통해서 자연스럽게 더 나은 코드를 만들자.

----
코딩에 영향을 주는 인지과정

코드가 이해하기 어렵고 혼란스러우면 불편하고 꺼림칙하다. 이러한 혼란을 초래하는 원인은 다음과 같다.
1. 프로그래밍 언어나 알고리즘 혹은 업무 영역에 대한 지식이 없는 경우 -> (냄새 1 이해하기 힘든이름에 해당)
2. 코드를 이해하기 위해 필요한 정보를 충분히 가지고 있지 못한경우 (라이브러리, 모듈 등)
3. 코드가 너무 복잡해서 혼한이 생겨 두뇌의 처리 용량이 부족한 경우 -> (냄새 2 중복코드에 해당)

- 출처 : 프로그래머의 뇌

----

## 리팩토링 1. 함수 선언 변경하기

### Change Function Declaration

함수 이름 변경하기, 메소드 이름 변경하기, 매개변수 추가하기, 매개변수 제거하기, 시그니처 변경하기

좋은 이름을 가진 함수는 함수가 어떻게 구현되었는지 코드를 보지 않아도 이름만 보고도이해할 수 있다.

좋은 이름을 찾아내는 방법? 함수에 주석을 작성한 다음, 주석을 함수 이름으로 만들어 본다.

함수의 매개변수는 함수 내부의 문맥을 결정한다. (예, 전화번호 포매팅 함수)

- Person Object vs 전화번호 String : 함수 내부에서의 확장성이 달라지고, 내부 문맥의 흐름을 결정 할 수 있는 요인이다. 의존성을 결정한다. (예, Payment 만기일 계산 함수)
- 단순 숫자만 넘길지, 객체를 넘길지

### InteliJ를 이용한 리팩토링 방법

- [rename-refactorings](https://www.jetbrains.com/help/idea/rename-refactorings.html)
- [change-signature](https://www.jetbrains.com/help/idea/change-signature.html)

### 예시 - Before

> __문제__ : 깃허브 레포지토리 이슈중 30번째 작성자와 리뷰를 출력한다.

```java
package me.whiteship.refactoring._01_smell_mysterious_name._01_before;

import org.kohsuke.github.*;
import org.springframework.beans.factory.annotation.Value;

import java.io.IOException;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class StudyDashboard {

    @Value("token")
    private static String token;

    private Set<String> usernames = new HashSet<>();

    private Set<String> reviews = new HashSet<>();

    // TODO: 메서드 명 변경 필요, Mysterious Name
    // TODO: 매개변수는 받지 않아도 됨, 읽을 대상의 리뷰는 이미 사전 정의 되어 있음
    private void studyReviews(GHIssue issue) throws IOException {

        List<GHIssueComment> comments = issue.getComments();
        for (GHIssueComment comment : comments) {
            usernames.add(comment.getUserName());
            reviews.add(comment.getBody());
        }
    }

    public Set<String> getUsernames() {

        return usernames;
    }

    public Set<String> getReviews() {

        return reviews;
    }

    public static void main(String[] args) throws IOException {

        GitHub gitHub = new GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(30);

        StudyDashboard studyDashboard = new StudyDashboard();
        studyDashboard.studyReviews(issue);
        studyDashboard.getUsernames().forEach(System.out::println);
        studyDashboard.getReviews().forEach(System.out::println);
    }
}

```

1. `studyReviews` -> `loadReviews` 로 변경
2. Param `GHIssue issue ` 제거

### After -1

```java
package me.whiteship.refactoring._01_smell_mysterious_name._01_before;

import org.kohsuke.github.*;
import org.springframework.beans.factory.annotation.Value;

import java.io.IOException;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class StudyDashboard {

    @Value("token")
    private static String token;

    private Set<String> usernames = new HashSet<>();

    private Set<String> reviews = new HashSet<>();

    private void loadReviews() throws IOException {

        GitHub gitHub = new.GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(30);
        // TODO: 조금 더 구체적인 이름으로 변경 필요와 메서드명과의 맥락이 맞지 않는 단어 선정 개선 필요
        List<GHIssueComment> comments = issue.getComments();
        for (GHIssueComment comment : comments) {
            usernames.add(comment.getUserName());
            reviews.add(comment.getBody());
        }
    }

    public Set<String> getUsernames() {

        return usernames;
    }

    public Set<String> getReviews() {

        return reviews;
    }

    public static void main(String[] args) throws IOException {

        StudyDashboard studyDashboard = new StudyDashboard();
        studyDashboard.loadReviews();

        // JAVA 문법에서만 통용되는 람다표현식 사용은 좋은가?
        studyDashboard.getUsernames().forEach(System.out::println);
        studyDashboard.getReviews().forEach(System.out::println);
    }
}

```

----

## 리팩토링 2. 변수 이름 바꾸기 (Rename Variable)

더 많이 사용되는 변수일수록 그 이름이 더 중요하다.

- 람다식에서 사용하는 변수 vs 함수의 매개변수 다이나믹 타입을 지원하는 언어에서는 타입을 이름에 넣기도 한다.
- 타입을 쓰는 사람의 추론을 위해서 사용하기도 함, ex) IntAge 여러 함수에 걸쳐 쓰이는 필드 이름에는 더 많이 고민하고 이름을 짓는다

### After - 2

D `After-1` 의 TODO를 개선했다.

- 메서드명의 맥락에 맞는 변수명으로 변경 `Comment` -> `Review`
- 그런데, 객체 타입명은 Comments ????

```java

public class StudyDashboard {

    @Value("token")
    private static String token; //TODO: 필드 명 변경 고민 필요 //TODO: 타입 고민 필요 -> 더 정확한 타입으로 개선, user와 review와의 관계는 알 수 없다. private
    Set<String> usernames = new HashSet<>();

    private Set<String> reviews = new HashSet<>();

    private void loadReviews() throws IOException {

        GitHub gitHub = new.GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(30);

        List<GHIssueComment> reviews = issue.getComments();
        for (GHIssueComment review : reviews) {
            usernames.add(review.getUserName());
            reviews.add(review.getBody());
        }
    }

    public Set<String> getUsernames() {

        return usernames;
    }

    public Set<String> getReviews() {

        return reviews;
    }

    public static void main(String[] args) throws IOException {

        StudyDashboard studyDashboard = new StudyDashboard();
        studyDashboard.loadReviews();
        studyDashboard.getUsernames().forEach(System.out::println);
        studyDashboard.getReviews().forEach(System.out::println);
    }
}


```

----

## 리팩토링 3. 필드 이름 바꾸기 (Rename Field)

Record 자료 구조의 필드 이름은 프로그램 전반에 걸쳐 참조될 수 있기 때문에 매우 중요 하다.

- Record 자료 구조: 특정 데이터와 관련있는 필드를 묶어놓은 자료 구조
- 파이썬의 Dictionay, 또는 줄여서 dicts.
- C#의 Record.
- 자바 14 버전부터 지원. (record 키워드)
- 자바에서는 Getter와 Setter 메소드 이름도 필드의 이름과 비슷하게 간주할 수 있다.

### Record

Immutable 한 개체생성 지원, private final 멤버변수 선언 및 Constructor, Getter/Setter, equals, hashCode, toString 지원 

- [java-record-keyword](https://www.baeldung.com/java-record-keyword)
- [옛날 레코드](https://ko.wikipedia.org/wiki/%EB%A0%88%EC%BD%94%EB%93%9C\_(%EC%BB%B4%ED%93%A8%ED%84%B0\_%EA%B3%BC%ED%95%99))

아래의 기능을 통해 우리는 Immutable한 객체를 다룬다.

```java
public class Person {

    private final String name;
    private final String address;

    public Person(String name, String address) {

        this.name = name;
        this.address = address;
    }

    @Override
    public int hashCode() {

        return Objects.hash(name, address);
    }

    @Override
    public boolean equals(Object obj) {

        if (this == obj) {
            return true;
        } else if (!(obj instanceof Person)) {
            return false;
        } else {
            Person other = (Person) obj;
            return Objects.equals(name, other.name)
                    && Objects.equals(address, other.address);
        }
    }

    @Override
    public String toString() {

        return "Person [name=" + name + ", address=" + address + "]";
    }

    // standard getters

}
```

Java 14이후 Record를 통해 지원 가능하다.

```java
public record Person(String name, String address) {

}
```

### After3

더 정확하게 필드의 이름을 `userName` 과 `reviews` 에서 `studyReviews` 단일화 후 변경했다.

```java
public class StudyDashboard {

    @Value("token")
    private static String token;

    private Set<StudyReview> studyReviews = new HashSet<>();

    private void loadReviews() throws IOException {

        GitHub gitHub = new GitHubBuilder().withOAuthToken(token).build();
        GHRepository repository = gitHub.getRepository("whiteship/live-study");
        GHIssue issue = repository.getIssue(30);

        List<GHIssueComment> reviews = issue.getComments();
        for (GHIssueComment review : reviews) {
            studyReviews.add(new StudyReview(review.getUserName(), review.getBody()));
        }
    }

    public Set<StudyReview> getStudyReviews() {

        return studyReviews;
    }

    public static void main(String[] args) throws IOException {

        StudyDashboard studyDashboard = new StudyDashboard();
        studyDashboard.loadReviews();
        studyDashboard.getStudyReviews().forEach(System.out::println);
    }

}
```

StudyReview 레코드 추가

```java
public record StudyReview(String reviewer, String review) {

}
```
