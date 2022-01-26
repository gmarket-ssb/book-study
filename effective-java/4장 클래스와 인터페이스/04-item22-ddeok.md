## [item 22] 인터페이스는 타입을 정의하는 용도로만 사용하라

<br>

클래스가 어떤 인터페이스를 구현 한다는 것  
= 자신의 인스턴스로 무엇을 할 수 있는지를 사용자에게 얘기해 주는 것

```java
public interface Eatable {
    void eat();
}

public class People implements Eatable {
    
    @Override
    public void eat() {
        ...
    }
}
```


### 용도에 맞지 않는 안티패턴

이렇게 사용되지 않는 대표적인 예: 상수 인터페이스
- 메서드 없이, 상수를 뜻하는 static final 필드로만 가득찬 인터페이스  

```java
// 22-1 상수 인터페이스 안티패턴 - 사용금지
public interface PhysicalConstants {
    // 아보가드로 수 (1몰)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    
    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    // 전자 질량 (kg)
    static final double ELECTRON_MASS = 9.109_383_56e-31;

    // JAVA 7부터 추가된 밑줄은 숫자에 아무런 영향을 주지 않고 가독성이 좋아짐
    // 고정소수점 수든 부동소수점 수든 5자리 이상이라면 밑줄을 사용하는걸 고려해보자. 
    // 십진수 리터럴도 (정수든 부동소수점 수든) 밑줄을 사용해 세자릿씩 묶어주는 것이 좋다.
}
```

클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다.  
따라서 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위다.  
사용자에게 혼란을 주기도 하며, 더 심하게는 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속되게 한다.  

상수를 공개할 목적이라면 더 합당한 선택지가 몇가지 있다.
모든 숫자 기본 타입의 박싱 클래스가 대표적으로, Ingeger와 Double에 선언된 MIN_VALUE와 MAX_VALUE 상수가 이런 예다.  

```java
class Integer {
    public static final int MIN_VALUE = 0x80000000; // -2147483648
    public static final int MAX_VALUE = 0x7fffffff; // 2147483647
}
```

열거 타입으로 나타내기 적합한 상수라면 열거 타입으로 만들어 공개하면 된다(아이템 34 - ENUM 사용). 그것도 아니라면, 인스턴스화할 수 없는 유틸리티 클래스(아이템 4)에 담아 공개하자.  
다음 코드는 앞서 보여준 PhysicalConstants의 유틸리티 클래스 버전이다.


```java
// 22-2 상수 유틸리티 클래스 (item04)
public class PhysicalConstants {
    private PhysicalConstants() { } // 인스턴스화 방지

    // 아보가드로 수 (1몰)
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```


```java
// 22-3 정적 임포트를 사용해 상수 이름만으로 사용하기
import static ...PhysicalConstants.*;

public class Test {
    double atoms(double mols) {
        return AVOGADROS_NUMBER * mols;
    }
    ...
    // PhysicalConstants를 빈번히 사용한다면 정적 임포트가 값어치를 한다.
}
```

### 핵심정리
★인터페이스는 타입을 정의하는 용도로만 사용★  
★상수 공개용 수단으로 사용하지 말자★
★사용 하려면 상수 유틸리티 클래스를 사용하자★