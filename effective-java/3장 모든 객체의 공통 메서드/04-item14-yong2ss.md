## [Item 14] Comparable을 구현할지 고려하라

Comparable을 구현한다는 것은 그 클래스의 인스턴스끼리는 순서를 비교 할 수 있다고 볼 수 있습니다.
<br>예를 들어, Arrays와 같이 Comparable은 사실상 자바 플랫폼 라이브러리의 대부분의 모든 값 클래스와 열거형을 구현했습니다.
<br>또한 알파벳, 숫자처럼 순서가 명확한 값 클래스는 대부분 구현하는 것이 좋습니다.

---

### Comparable 인터페이스의 유일한 메서드인 compareTo과 그 규약

> * 자신의 인스턴스가 매개변수보다 작으면 음수, 같으면 0, 크면 양수를 반환한다. <br>(객체 타입이 비교가 불가하면 ClassCastException)

> (sgn은 부호 함수이며, -1, 0, 1을 사용한다.)
> * [대칭성] 모든 x,y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다.
> * [추이성] x.compareTo(y) > 0 && y.compareTo(z) > 0 이면 x.compareTo(z) > 0이다.
> * [반사성] x.compareTo(y) == 0 이면 모든 z에 대해 sgn(x.compareTo(z)) == sgn(y.compareTo(z))
> * 필수는 아니지만, (x.compareTo(y) == 0) == x.equals(y)이다.
   <br>이를 지키지 않는 클래스에는 "주의 : 이 클래스의 순서는 equals 메서드와 일관되지 않다"를 명시하는게 좋다.

* equals != compareTo 예시
```
   HashSet<BigDecimal> set1 = new HashSet<>();
   TreeSet<BigDecimal> set2 = new TreeSet<>(); //비교(compare)를 사용하는 정렬된 컬렉션

   var a = new BigDecimal("1.0");
   var b = new BigDecimal("1.00");
   
   // BigDecimal은 equals와 compareTo가 일관되지 않는다.
   // a와 b를 equals == false, compareTo == 0
   // set1.size() == 2, set2.size() == 1
```

### Comparable 사용시 유의사항

- 기본 타입 필드를 비교 할 땐
기존의 compareTo 메서드에서는 관계연산자 < 와 > 를 사용하였지만 지금은 추천하지 않는 문법입니다.
<br>java7 부터는 기본 박싱 클래스에 정적 메서드가 추가되었기 때문에 BoxingClass.compare(p1, p2)처럼 사용하면 됩니다.
<br>[관련 overflow 문제](https://monicagranbois.com/blog/java/comparing-integers-using-integercompare-vs-subtraction/)
```java
Integer va = Integer.valueOf(5);
Integer vb = Integer.valueOf(5);
System.out.println(Integer.compare(va , vb));        
```
- 클래스에 핵심 필드가 여러개라면 어느 것을 먼저 비교할지가 중요하다.
- 가장 핵심적인 필드부터 비교를 하면 된다. 똑같으면 그 다음으로 중요한 필드를 비교하는 식으로 진행하면 된다.

<br>

#### Comparator
Comparable을 구현하지 않았을 때는 비교자(Comparator)를 대신 사용

```java
recipeManualTabSubs.stream().sorted(Comparator.comparing(RecipeManualTabSubCache::getPriority)).collect(Collectors.toList());
```

Java 8에서는 Comparator 인터페이스에서 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었습니다.
<br>이 비교자들을 Comparable 인터페이스가 요구하는 compareTo() 메서드를 구현하는데 활용할 수 있습니다.
<br>이 방식은 간결함이라는 장점이 있지만, 약간의 성능 저하가 뒤따릅니다.
```java
private static final Comparator<Member> COMPARATOR = 
    Comparator.comparingInt((Member member) -> member.age)
                .thenComparingInt(member -> member.name)
                .thenComparingInt(member -> member.height);

public int compareTo(Member member) {
    return COMPARATOR.compare(this, member);
}
```

---
### [ 정리 ]
* 순서를 고려해야 하는 값 클래스를 작성할 경우 되도록 Comparable 인터페이스를 구현합니다.
<br>그 인스턴스들은 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러질 수 있고
<br>단순 동치병 비교에 순서까지 비교 가능하며, 더 쉽게 정렬 할 수 있는 등의 많은 부가적인 혜택을 누릴 수 있습니다.
* compareTo()에서 필드의 값을 비교할 때 '<'와 '>' 연산자는 되도록 쓰지 않도록 합니다.
1) 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나
2) Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용합니다.
