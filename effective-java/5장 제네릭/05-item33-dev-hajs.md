# 타입 안전 이종 컨테이너를 고려하라

### 타입 안전 이종 컨테이너 (type safe heterogeneous container pattern)
* 책에서는 "컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하는 설계 방식" 이라고 설명하고 있다.
* 설명이 너무 어렵다 🤯  그냥 "Map 처럼 인스턴스를 넣고 뺴고 하는데 타입이 안전한 설계 방식" 이라고 봐도 충분해 보인다.
<br><br>

### 코드 예시
```java
public class Item33 {
    
    public static void main(String[] args) {
        Favorites f = new Favorites();

        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 1000);
        f.putFavorite(Class.class, Favorites.class);

        String favoriteString = f.getFavorite(String.class);
        Integer favoriteInteger = f.getFavorite(Integer.class);
        Class<?> favoriteClass = f.getFavorite(Class.class);

        System.out.println("favoriteString = " + favoriteString);
        System.out.println("favoriteInteger = " + favoriteInteger);
        System.out.println("favoriteClass = " + favoriteClass.getName());
    }

    /**
     * Favorites Class
     */
    static class Favorites {
        private Map<Class<?>, Object> favorites = new HashMap<>();

        public <T> void putFavorite(Class<T> type, T instance) {
            favorites.put(Objects.requireNonNull(type), type.cast(instance));
        }
        public <T> T getFavorite(Class<T> type) {
            return type.cast(favorites.get(type));
        }
    }
}
```
위 코드는 세 가지를 염두해서 봐야 한다.
1. favorites 의 타입은 `Map<Class<?>, Object>` 이다.
    * 비한정적 와일드카드가 쓰여서 Map 안에 아무것도 넣을 수 없다고 생각할 수 있지만, **중첩** 되게 사용했으므로 실은 그 반대다.
    * Map 이 아니라 Map 의 Key 가 와일드카드 타입이므로, "모든 키가 서로 다른 매개변수화 타입일 수 있다" 는 뜻이 된다.
2. favorites 맵의 값 타입은 단순히 `Object` 이다.
    * Map 의 Key 타입과 Value 타입이 같음을 보장하지 않는다.
    * Key 와 Value 사이의 '타입 링크(type linkage)' 정보가 버려진다.
      * ✨ 이제 여기서 `.getFavorite()` 메소드를 유심히 봐야 한다.
      * ✨ 위 코드가 타입이 안전하다는 것은 해당 메소드의 역할 때문이다.
      * Map 에서 꺼낸 객체는 잘못된 컴파일타임 타입을 가지고 있으므로 Object 나 T 로 바꿔 반환해야 한다.
      * 즉, Map 안의 값은 해당 Key 의 타입과 항상 일치해진다.
3. favorites 에 값을 넣을 때 type casting 을 한번 더 체크한다.
    * 이 방법은 '로 타입' 으로 값을 넘겼을 때 타입 안전성을 깨지는 현상을 방지할 수 있다.
<br><br>

그리고 실체화 불가 타입에는 위 코드와 같은 방식을 사용할 수 없다.
* List<String> 용 Class 객체를 얻을 수 없기 때문이다.
* 만일 이를 허용한다면, List<String> 와 List<Integer> 는 같은 `List.class` 클래스를 공유하므로 Map 객체 내부는 아수라장이 될 것이다.
<br><br>
  
### 추가 개념 설명
* 클래스 리터럴
  * ex. String.class, Integer.class
* 타입 토큰 (type token)
  * 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 의미함
  * ex. `objectMapper.readValue(jsonString, Book.class)` -> 클래스 리터럴이 타입 토큰으로 쓰인 경우
* 슈퍼 타입 토큰
  * 상속과 리플렉션을 조합해 타입 토큰을 넘어서 `List<String>` 와 같은 형태도 타입 토큰으로 사용할 수 있도록 만든 토큰
  * 자세한 내용은 Neal Gafter 블로그 글을 참고.. http://gafter.blogspot.com/2006/12/super-type-tokens.html
<br><br>
