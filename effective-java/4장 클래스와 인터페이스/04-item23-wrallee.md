## 태그 달린 클래스보다는 클래스 계층구조를 활용하라

아래와 같이 **특정 필드에 따라 유형이 분류되는 클래스**를 **<ins>Tagged class</ins>**라고 한다.  
아래 클래스는 Shape라는 필드를 기준으로 사각형이냐 원이냐로 나뉜다.

```java
// Tagged class - vastly inferior to a class hierarchy! (Page 109)
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // Tag field - the shape of this figure
    final Shape shape;

    // These fields are used only if shape is RECTANGLE
    double length;
    double width;

    // This field is used only if shape is CIRCLE
    double radius;

    // Constructor for circle
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // Constructor for rectangle
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

만약 위와 같은 구조에서 삼각형이라도 구현하려 한다면 더 복잡해진다.  
삼각형의 세 변을 새로운 변수로 추가하고, 삼각형의 넓이 계산법을 switch문 안에 추가해야된다.

이처럼 TaggedClass는 열거타입 선언, 태그 필드, 스위치문 등 쓸데없는 코드가 많다.  
또한 새로운 유형을 추가할 경우 기존 소스를 분석하고 이해해서 수정해야 한다. 잘못 이해하고 수정할 경우 기능이 제대로 동작하지 않을수도 있다.

<br>

> TaggedClass는 장황하고, 오류를 내기 쉽고, 비효율적이다.

<br>

따라서 TaggedClass를 사용해야 한다면 이를 계층구조로 구성할수는 없는지 고민해보자.  
아래 소스는 위 Figure를 계층 구조로 리팩터링 한 코드이다.

```java
// Class hierarchy replacement for a tagged class  (Page 110-11)
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}

class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```

<br>

추가로 JPA의 **@Inheritance**, **@DiscriminatorColumn** 어노테이션을 활용한 상속관계도 비슷한 사례로 생각할 수 있다. 상속 전략으로 <ins>SINGLE_TABLE</ins> 방식을 쓰게되면 테이블은 마치 TaggedClass처럼 구성된다. 하지만 엔티티의 관계는 상속으로 구현된다는 점을 생각해보자.

만약 테이블을 보고 엔티티를 구현해야한다고 할 때, 같은 테이블을 보고 누군가는 TaggedClass로 구현할것이고 누군가는 상속관계로 구현할 것이다. 생각하는 방식에 따라 구현방법이 달라질 수 있다.

JPA참고: https://www.baeldung.com/hibernate-inheritance

<br>

#### 결론

- TaggedClass를 써야 하는 상황은 거의 없다.
- 설계 시 TaggedClass를 만들게 되었다면, 이를 계층구조로 구현할수는 없는지 생각해보자.
- 기존에 TaggedClass가 존재했다면, 이를 계층구조로 리팩터링 하는걸 고민해보자.