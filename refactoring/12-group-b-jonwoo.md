# 냄새12. 반복되는 swtich 문

반복해서 등장하는 동일한 switch 문

-   동일한 swtich 문이 존재할 경우 새로운 조건을 추가하거나 기존 조건을 변경할 때 모든 swtich 문을 찾아서 코드를 고쳐야할지도 모른다


---

Java14 Switch Expression

```java
static void  test(Day day){
        switch (day) {
            case MONDAY:
            case FRIDAY:
            case SUNDAY:
                System.out.println(6);
                break;
            case TUESDAY:
                System.out.println(7);
                break;
            case THURSDAY:
            case SATURDAY:
                System.out.println(8);
                break;
            case WEDNESDAY:
                System.out.println(9);
                break;
    }
}
```

```java
static void  test(Day day){
        switch (day) {
             case MONDAY, FRIDAY, SUNDAY -> System.out.println(6);
             case TUESDAY                -> System.out.println(7);
             case THURSDAY, SATURDAY     -> System.out.println(8);
             case WEDNESDAY              -> System.out.println(9);
    }
}
```
