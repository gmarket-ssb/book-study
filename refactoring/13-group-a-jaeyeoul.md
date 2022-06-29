# 13. 반복문(Loops)

프로그래밍 언어 초기부터 있었던 반복문은 처음엔 별다른 대안이 없어서 간과했지만 
최근 Java와 같은 언어에서 함수형 프로그래밍을 지원하면서 반복문에 비해 더 나은 대안책이 생겼다.

- __반복문을 파이프라인으로 바꾸는 (Replace Loop with Pipeline)__ 리팩토링을 적용하면 필터나 맵핑과 같은 파이프라인 기능을 사용해서 각 코드에서 원소가 어떻게 처리되는지를 쉽게 파악 할 수 있다.


## 해결
### Replace Loop with Pipeline

콜렉션 파이프라인 (자바의 Stream, C#의 LINQ - Language Integrated Query)
고전적인 반복문을 파이프라인 오퍼레이션을 사용해 표현하면 코드를 더 명확하게 만들 수있다.

- 필터 (filter): 전달받은 조건의 true에 해당하는 데이터만 다음 오퍼레이션으로 전달.
- 맵 (map): 전달받은 함수를 사용해 입력값을 원하는 출력값으로 변환하여 다음 오퍼레이션으로 전달.

• https://martinfowler.com/articles/refactoring-pipelines.html


### 리팩토링 전
```java
public class Author {

    private String company;

    private String twitterHandle;

    public Author(String company, String twitterHandle) {
        this.company = company;
        this.twitterHandle = twitterHandle;
    }

    static public List<String> TwitterHandles(List<Author> authors, String company) {
        var result = new ArrayList<String> ();
        for (Author a : authors) { // 반복
            if (a.company.equals(company)) { // 필터 부분
                var handle = a.twitterHandle;
                if (handle != null)
                    result.add(handle); // 수집 부분
            }
        }
        return result;
    }

}
```

### 리팩토링 후 
- map, filter, collect 사용해서 위 코드에서의 주석이 필요없고 명확해진다.
- 더 긴 코드였을 경우 더 빠르게 읽고 수정하기 좋다.

```java
public class Author {

    private String company;

    private String twitterHandle;

    public Author(String company, String twitterHandle) {
        this.company = company;
        this.twitterHandle = twitterHandle;
    }

    public static List<String> TwitterHandles(List<Author> authors, String company) {

        return authors.stream().filter(author -> author.equals(company))
                .map(author -> author.twitterHandle)
                .filter(t -> t != null)
                .collect(Collectors.toList());
    }

}

```
