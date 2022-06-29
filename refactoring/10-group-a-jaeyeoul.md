# 10. 데이터 뭉치 (Data Clumps)

항상 뭉쳐 다니는 데이터는 한 곳으로 모아두는 것이 좋다.
여러 클래스에 존재하는 비슷한 필드 목록
여러 함수에 전달하는 매개변수 목록

## 관련 리팩토링 기술
“클래스 추출하기 (Extract Class)”를 사용해 여러 필드를 하나의 객체나 클래스로 모을 수 있다.
“매개변수 객체 만들기 (Introduce Parameter Object)” 또는 “객체 통째로 넘기기 (Preserve Whole Object)”를 사용해 메소드 매개변수를 개선할 수 있다


> 데이터 뭉치인지 판별하려면 값 하나를 삭제해보자. 그랬을 때 나머지 데이터만으로는 의미가 없다면 객체로 환생하길 갈망하는 데이터 뭉치라는 뜻이다. 
> - 리팩터링 2판 122p

### 문제
함께 뭉쳐서 다니는 데이터 뭉치로 인해서, 사용하는 곳에서의 코드가 지저분하다

### 예제

```java
public class Employee {

    private String name;

    private String personalAreaCode;

    private String personalNumber;
    
    // 데이터 뭉치 냄새가 나는 구간 증거 1
    public Employee(String name, String personalAreaCode, String personalNumber) {
        this.name = name;
        this.personalAreaCode = personalAreaCode;
        this.personalNumber = personalNumber;
    }
    
    // 데이터 뭉치 냄새가 나는 구간 증거 2
    public String personalPhoneNumber() {
        return personalAreaCode + "-" + personalNumber;
    }
	
    // etc ...
}
```

## 해결

- 클래스로 묶자!

```java
public class TelephoneNumber {

    private String areaCode; // 추상화하여 personal 접두사 제거

    private String number;

    @Override
    public String toString() {
        return this.areaCode + " - " + this.number; // 묶어서 표현하는 구간 해결
    }
    
    ..
}
```

```java
public class Employee {

    private String name;

    private TelephoneNumber personalNumber;

    public Employee(String name, TelephoneNumber telephoneNumber) {
        this.name = name;
        this.personalNumber = telephoneNumber;
    }
    
    // 타 클래스에서 아래 코드의 참조가 많다면, 아래와 같이 변경
    public setPersonalAreaCode(String personalAreaCode) {
        this.personalNumber.setArea(personalAreacode);
    }
    
}
```
