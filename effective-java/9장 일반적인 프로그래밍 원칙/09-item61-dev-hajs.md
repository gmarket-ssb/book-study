## 박싱된 기본 타입보다는 기본 타입을 사용하라
> 자바의 데이터 타입은 기본 타입과 참조 타입으로 나눌 수 있으며, 기본 타입도 대응되는 참조 타입을 하나씩 가지고 있다.<br>
아이템 6<sup>불필요한 객체 생성을 피하라</sup> 에서 언급했듯이 오토박싱과 오토언박싱 덕분에 둘 사이의 경계가 옅어지긴 했으나, 차이점은 분명하니 사용할 때 주의해서 사용하자.
<br>

### 박싱된 기본 타입과 기본 타입의 차이점
1. 박싱된 기본 타입은 `식별성` 을 갖는다.
    * -> 박싱된 기본 타입에 `==` 연산자를 사용하면 오류가 발생한다. (발생할 수 있다.)
    * -> 이때는 `Comparator.naturalOrder()` 를 사용하면 된다.
3. 박싱된 기본 타입은 `null` 을 가질 수 있다.
4. 박싱된 기본 타입은 기본 타입보다 시간과 메모리 사용면에서 비효율적이다.
<br>

### 차이점을 모른채 사용하면 발생하는 이슈
```java
public class Unbelievable {
  static Integer i;
  
  public static void main(String[] args) {
    if (i == 42)
      System.out.println("믿을 수 없군!");
  }
}
```
* 이 프로그램은 `NullPointerException` 을 던지며 끝난다.
* 그 이유는, 박싱된 기본 타입과 기본 타입을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 자동으로 (거의 대부분) 풀리기 때문이다.
* 이 과정에서 null 참조를 언박싱하게 되어 `NullPointerException` 을 마주한다.
<br>

### 그럼에도 박싱된 기본 타입을 사용하는 경우
* 컬렉션은 기본 타입을 담을 수 없기에 **컬렉션의 원소, 키, 값** 으로 사용한다.
  * `Map<String, String> map = new HashMap<>()` ...
* 자바 언어가 타입 매개변수로 기본 타입을 지원하지 않기에 **매개변수화 타입이나 매개변수화 메서드의 타입매개변수** 로 사용한다.
  * `ThreadLocal<Integer>` ...
* **리플렉션을 통해 메서드를 호출할 때** 도 박싱된 기본타입을 사용해야 한다.
   * `Method.invoke(Object object)` -> [docs](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html#invoke-java.lang.Object-java.lang.Object...-)
<br>

### 정리
* 기본 타입과 박싱된 기본 타입 중 하나를 선택해야 한다면 가능하면 기본 타입을 사용하라.
* 박싱된 기본 타입을 써야 한다면 주의를 기울이자.
