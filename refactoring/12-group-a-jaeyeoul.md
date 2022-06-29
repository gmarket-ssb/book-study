# 12. 반복되는 switch 문 (Repeated Swtiches)

- 예전에는 switch 문이 한번만 등장해도 코드 냄새로 생각하고 __다형성 적용을 권장__ 했다.
- 하지만 최근에는 다형성이 꽤 널리 사용되고 있으며, 여러 프로그래밍 언어에서 보다 세련된 형태의 switch 문을 지원하고 있다.
- 따라서 오늘날은 __반복해서 등장하는 동일한 switch 문__ 을 냄새로 여기고 있다.
- 반복해서 동일한 switch 문이 존재할 경우, 새로운 조건을 추가하거나 기존의 조건을 변경 할 때 모든 switch 문을 찾아서 코드를 고쳐야 할지도 모른다

## Switch Expression
자바 12와 13,14를 거쳐서 나옴 
- https://docs.oracle.com/en/java/javase/13/language/switch-expressions.html

### Switch statement
```java
public enum Day { SUNDAY, MONDAY, TUESDAY,
    WEDNESDAY, THURSDAY, FRIDAY, SATURDAY; }

// ...

    int numLetters = 0;
    Day day = Day.WEDNESDAY;
    switch (day) {
        case MONDAY: // 불편한 구간
        case FRIDAY: // 불편한 구간
        case SUNDAY: // 불편한 구간
            numLetters = 6;
            break; // 불편한 구간
        case TUESDAY:
            numLetters = 7;
            break;
        case THURSDAY:
        case SATURDAY:
            numLetters = 8;
            break;
        case WEDNESDAY:
            numLetters = 9;
            break;
        default:
            throw new IllegalStateException("Invalid day: " + day);
    }
    System.out.println(numLetters);
```

### Switch Expression

더이상 `break` 를 신경쓰지 않아도 된다. 불필요한 case 도 추가하지 않아도 된다.
단지 콤마 , 만 추가하면 된다.

또 변수에 담아두지 않고 immutable 하게 사용가능하다.

```java
    Day day = Day.WEDNESDAY;    
    System.out.println(
        switch (day) {
            case MONDAY, FRIDAY, SUNDAY -> 6;
            case TUESDAY                -> 7;
            case THURSDAY, SATURDAY     -> 8;
            case WEDNESDAY              -> 9;
            default -> throw new IllegalStateException("Invalid day: " + day);
        }
    );    
```
