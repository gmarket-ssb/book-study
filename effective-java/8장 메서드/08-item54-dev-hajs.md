## null 이 아닌, 빈 컬렉션이나 배열을 반환하라
> 컬렉션이나 배열 같은 컨테이너가 비었을 때 null 을 반환하는 메소드를 사용하게 되면 항시 방어 코드를 넣어줘야 한다.<br>
> 만일, 클라이언트에서 방어 코드를 빼먹으면 오류가 발생할 수 있다.

<br>

```java
/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 * 단, 재고가 하나도 없다면 null 을 반환한다.
 */
public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInstock);
}

// client
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON)) // cheeses != null <<
  System.out.println("좋았어, 바로 그거야.");
```

- 빈 컬렉션을 반환하기 위해서 하는 할당이 성능에 영향을 끼친다고 생각된다면 ‘불변’ 컬렉션을 반환하면 된다.

  - 이는 최적화가 필요하다고 판단되는 경우에만 쓰고, 쓰기 전후의 성능을 꼭 확인하자.

  - ex. Collections.emptyList
    ```java
    /**
     * The empty list (immutable).  This list is serializable.
     *
     * @see #emptyList()
     */
    @SuppressWarnings("rawtypes")
    public static final List EMPTY_LIST = new EmptyList<>();
    ```
<br>

### 정리
- null 을 반환하는 API 는 사용하기 어렵고, 오류 처리 코드도 늘어나고, 성능이 좋은 것도 아니다.
- null 이 아닌, 빈 배열이나 컬렉션을 반환하라.
