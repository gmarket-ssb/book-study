## equals는 일반 규약을 지켜 재정의하라

***꼭 필요한 경우가 아니라면 `equals`를 재정의하지 말자.***

**[equals를 재정의할 필요가 없는 경우]**

- 각 인스턴스가 고유할 경우
- 논리적 동치성을 검사할 일이 없을 경우.
- 상위 클래스에서 재정의한 `equals`로도 충분할 경우.
- 클래스가 `private` 혹은 `package-private`이고, `equals`가 호출되지 않았음을 확신할 경우.
  - 만약 실수로 호출될 것 같다면 `equals`를 호출 즉시 예외를 던지도록 재정의 하면 된다.

논리적으로 값이 같은 인스턴스가 2개 이상 생성되지 않는 경우(`Enum` 등)  
논리적 동치성과 객체 식별성의 의미가 같기 때문에 `Object`의 `equals` 만으로도 정상적으로 동작한다.

<br>

**[equals 재정의 규약]**  
(단, `x`,`y`,`z`는 `null`이 아니다)

- 반사성(reflexivity): `x.equals(x) == true`
- 대칭성(symmetry): `x.equals(y) == true` → `y.equals(x) == true`
- 추이성(transitivity): `x.equals(y) == true` → `y.equals(z) == true` → `z.equals(x) == true`
- 일관성(consistency): `x.equals(y)`를 반복해서 호출해도 항상 같은 결과가 나온다.
- null-아님: `x.equals(null)`은 항상 `false`이다.

<br>

### 위배 사례

대소문자를 구분하지 않는 값을 가진 `CaseInsensitiveString`이라는 클래스가 있다.  
이 클래스의 `equals`를 아래와 같이 구현한다면 <ins>**대칭성**</ins>을 위배하게 된다.

```java
@Override // CaseInsensitiveString의 equals 재정의
public boolean equals(Object o) {
    if (o instanceof CaseInsensitiveString)
        return s.equalsIgnoreCase(((CaseInsensitiveString) o).str);
    if (o instanceof String) // 대칭성 위배 요소
        return str.equalsIgnoreCase((String) o);
    return false;
}

CaseInsensitiveString caseInsensitive = new CaseInsensitiveString("Polish");

>>>>> caseInsensitive.equals("polish") != "polish".equals(caseInsensitive);
```

혹은 구체 클래스를 확장하면서 `equals`를 구현하려다가 일반규약을 위배하기도 한다.  
`Point` 클래스를 상속받는 `ColorPoint`라는 클래스가 있다고 할 때, 아래 예시는 <ins>**대칭성**</ins>을 위배하였다.

```java
@Override // ColorPoint의 equals 재정의
public boolean equals(Object o) {
    if (!(o instanceof ColorPoint)) // 대칭성 위배 요소
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}

Point point = new Point(1, 2);
ColorPoint colorPoint = new ColorPoint(1, 2, Color.RED);

>>>>> colorPoint.equals(point) != point.equals(colorPoint);
```

그리고 아래 예시는 <ins>**추이성**</ins>을 위배한다.

```java
@Override // ColorPoint의 equals 재정의
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return o.equals(this);
    if (!(o instanceof ColorPoint)) // 추이성 위배 요소
        return o.equals(this);
    return super.equals(o) && ((ColorPoint) o).color == color;
}

ColorPoint redPoint = new ColorPoint(1, 2, Color.RED);
ColorPoint bluePoint = new ColorPoint(1, 2, Color.BLUE);
Point bridgePoint = new Point(1, 2);

>>>>> redPoint.equals(point) == bluePoint.equals(point) == true;
>>>>> redPoint.equals(bluePoint) == false
```

또한 다음 예시는 `getClass()`를 활용해서 규약은 지켰으나 <ins>**리스코프 치환 원칙**</ins>(*상위 타입의 객체를 하위 타입으로 치환해도 정상적으로 동작해야 한다*)을 위배하였기 때문에 기능이 올바르게 동작하지 않는다.

```java
@Override // Point의 equals 재정의
public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass())
        return false;
    Point p = (Point) o;
    return p.x == x && p.y == y;
}

class UnitCircle {
    private static final Set<Point> unitCircle = Set.of(
            new Point( 1, 0), new Point(0, 1),
            new Point(-1, 0), new Point(0,-1));

    public static boolean onUnitCircle(Point p) {
        return unitCircle.contains(p);
    }
}

public class ColorPoint extends Point {
    private Color color;
    ...
}

>>>>> UnitCircle.onUnitCircle(new ColorPoint(1, 0, Color.RED)) == false;
```

 <br>

결론적으로 **구체 클래스를 확장해 새로운 값을 추가하면서 `equals` 규약을 만족시킬 방법은 존재하지 않는다.**  
다만 상속이 아닌 컴포지션(확장하려는 클래스를 멤버변수로 선언하는 것)을 활용한다면 가능하다.

<br>

### 결론

- 꼭 필요한 경우가 아니라면 `equals`를 재정의 하지 말자.
- `equals`를 구현해야 한다면 다음 방식을 따르면 좋다.
  - `==` 연산자를 이용해 입력이 자기 자신의 참조인지 확인한다.
  - `instanceof` 연산자로 입력이 올바른 타입인지 확인한다.
  - 입력을 올바른 타입으로 형 변환한다.
  - 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 검사한다.
- `equals`를 구현했다면 **대칭성, 추이성, 일관성**을 점검하자.
- `equals`를 재정의할 땐 `hashCode`도 반드시 재정의하자.

<br>

### 기타

- 다음은 실제 자바 라이브러리에서 일반규약을 위배한 사례이다.
  - `java.sql.TimeStamp.equals(...)`: 대칭성 위반
  - `java.net.URL.equals(...)`: 일관성 위반
- AutoValue나 Lombok과 같은 어노테이션 프로세서를 사용해서 equals & hashCode를 구현할 수 있다.  
  <ins>하지만 [Pitfall(함정)](https://kwonnam.pe.kr/wiki/java/lombok/pitfall)을 항상 조심하자</ins>.
