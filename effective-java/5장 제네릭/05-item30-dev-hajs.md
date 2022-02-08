# 이왕이면 제네릭 메서드로 만들라
| 메소드도 제네릭으로 만들 수 있다.
<br><br>

### 제네릭 메서드 사용
다음은 **일반적인** 두 집합의 합집합을 반환하는 메소드이다.
```java
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1);
  result.addAdd(s2);
  return result;
}
```

컴파일은 되지만, 경고가 발생한다.
![image](https://user-images.githubusercontent.com/57446639/153078814-e79a0765-26f3-4f32-909a-d855c2c10cda.png)

경고를 없애려면 이 메소드를 타입 안전하게 만들면 된다.
```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet<>(s1);
  result.addAll(s2);
  return result;
}
```
* 전의 코드보다 경고없이 타입 안전하고 쓰기도 쉬우나,<br>
와일드카드 타입을 사용하지 않으면 타입이 모두 같아야 한다는 점은 유의하자.
<br><br>
  
### 제네릭 싱글턴 팩토리(Generic Singleton Factory)
요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩토리
```java
private static class EmptyMap<K,V> extends AbstractMap<K,V> implements Serializable { 
  ...
  public Set<K> keySet()                     {return emptySet();}
  public Collection<V> values()              {return emptySet();}
  public Set<Map.Entry<K,V>> entrySet()      {return emptySet();}
  ..
}
```
<br><br>

### 재귀적 타입 한정(Recursive type bound)
자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용범위를 한정할 수 있는 방식
```java
public interface Comparable<T> {
  int compareTo(T p);
}
```
* 여기서 타입 매개변수 `T` 는 `Comparable<T>` 를 구현한 타입이 비교할 수 있는 원소의 타입을 정의함<br>
(실제로 거의 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있다.)
* 주로 타입의 자연적 순서를 정하는 `Comparable` 인터페이스와 함께 쓰인다.
<br><br>

### 핵심 정리
* 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다.
* 메서드도 형변환 없이 사용할 수 있는 편이 좋으며, 많은 경우 그렇게 하려면 제네릭 메서드가 되어야 한다.
* 형변환을 해줘야 하는 기존 메서드는 제네릭하게 만들자.
* 기존 클라이언트는 그대로 둔 채 새로운 사용자의 삶을 훨씬 편하게 만들어줄 것이다.
* **-> "가급적 메소드도 제네릭으로 만들자"**
