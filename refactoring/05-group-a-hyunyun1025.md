# 냄새 5. 전역 데이터(Global Data)

- 전역 데이터 (ex, 자바의 public static 변수)
- 전역 데이터는 아무곳에서나 변경될 수 있다는 문제가 있다.
- 따라서, 어디서 값이 바뀌었는지 파악하기 어렵다.
- **변수 캡슐화**를 적용해서 접근을 제어하거나 어디서 사용하는지 파악하기 쉽게 만들 수 있다.

## 전역 변수에 대한 마음가짐
- 전역 데이터가 필요하여 사용될 수 있으나, **많이 사용하는 것은 지양**해야한다.
- 피라켈수스의 격언 "약과 독의 차이를 결정하는 것은 사용량일 뿐이다."

## 변수 캡슐화하기
1. 데이터를 직접 변경하면 모든 영역에 영향이 가므로, 가급적이면 메서드를 통해 변경하도록 해야한다. 데이터 변경 메서드를 추가하며 점진적으로 변경방법을 늘려 유연하게 대처할 수 있다.
2. 메서드화 하면 값 변경에 대한 유효성 검증이 쉬워진다.

> #### before

```java
// 온도 조절기
public class Thermostats {
    public static Integer targetTemperature = 70;
    public static Boolean heating = true;
    public static Boolean cooling = false;
    public static Boolean fahrenheit = true; // 화씨, 섭씨
}

public class Home {
    public static void main(String[] args) {
        System.out.println(Thermostats.targetTemperature);

        // targetTemperature를 말도 안되는 온도로 설정하면??
        Thermostats.targetTemperature = -1111600;
        Thermostats.fahrenheit = false;
    }
}
```

> #### after

```java
@Getter
@Setter
public class Thermostats {
    private static Integer targetTemperature = 70;
    private static Boolean heating = true;
    private static Boolean cooling = false;
    private static Boolean fahrenheit = true; // 화씨, 섭씨

    @Override
    public static void setTargetTemperature(Integer targetTemperature) {
        // TODO validation
        Thermostats.targetTemperature = targetTemperature;
        // TODO notify
    }
}

public class Home {
    public static void main(String[] args) {
        System.out.println(Thermostats.getTargetTemperature());

        Thermostats.setTargetTemperature(-1111600);
        Thermostats.setFahrenheit(false);
    }
}
```

> ### 점진적으로 수정 메서드를 늘려나가던 실제 경험
> 1. JPA 엔티티 setter 추가
> - 이전 회사에서, 엔티티에 전체 적용되는 @Setter 사용을 금지하였으며, 필요한 경우 직접 Setter 메서드를 추가하여야 했음.
> - Setter가 일부만 선언되어서, 추적이 쉬워진다는 장점을 직접 느낄 수 있음
> - 여러 프로퍼티를 일괄적으로 set해야하는 경우에 실수를 줄일 수 있었음

> #### before

```java
public void updateSomething() {
    Something s = somethingRepository.findById(1L);
    s.setA("Something");
    s.setB("Something");
    s.setC("Something"); // 이러면 어딘가에선 set 하나를 빼먹는 등의 실수가 일어날 수 있다.
    somethingRepository.save(s);
}
```

> #### after

```java
public void updateSomething() {
    Something s = somethingRepository.findById(1L);
    s.setABC("Something");
    somethingRepository.save(s);
}
```

# 번외. FrontEnd에서는 왜 Store를 선호할까??

## Store란?
- 다른 컴포넌트들이 같은 데이터를 접근하기 위해 **전역변수처럼** 사용하는 것
- 상태관리 기법을 Vue에서는 Vuex, React에서는 Redux로 부른다. 

### 개인적인 의문점
- FrontEnd에서 Store는 전역변수를 미화하여 아무렇지 않게 사용하고 있다. 왜 그럴까??

![이미지](https://i.imgur.com/grw7odu.png)

### 답변
- 전역변수 개념과는 다르게, Redux에서 상태를 관리해준다. 따라서 비교적 loose coupling하다.
- 상태 관리를 캡슐화한 상태로 유지하면서 UI 구현을 쉽게 바꿀수 있게 도와준다.
- 컴포넌트A에서 컴포넌트B의 전역변수를 변경하면, 컴포넌트B는 이를 감지할수 없다. Redux는 상태관리를 통해 참조된 변수에 대해 컴포넌트에 변경을 전파하는 것이 가능하다.

### 결론
- Redux는 **전역변수**와는 정확하게 다른 의미이며, **전역 상태 컨테이너**이다.
- 캡슐화되어, ReadOnly 데이터와 수정 메서드가 있는 단일 전역 인스턴스라 생각하면된다.
- 즉, Redux는 전역변수와는 다르게 **항상 최신 상태**이며, read/write에 대한 **지침과 절차**가 있으며, **상태를 추적**하고 관리한다.

### 참조링크
https://stackoverflow.com/questions/33424157/isnt-redux-just-glorified-global-state
https://www.reddit.com/r/reactjs/comments/gpzg3i/is_redux_just_reacts_way_of_using_global_variables/
https://www.reddit.com/r/reactjs/comments/b4p87h/redux_is_just_money_laundering_for_global/