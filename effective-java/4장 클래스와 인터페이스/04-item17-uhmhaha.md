
## [Item017] 변경 가능성을 최소화하라( immutable class )
> Immutable classes are less prone to error and are more secure
>   ex) String, BigInteger, BigDecimal

## **상세설명**

### 1. class immutable have five rules
- Don’t provide methods that modify the object’s state 
- Ensure that the class can’t be extended
- Make all fields final
- Make all fields private
- Ensure exclusive access to any mutable components

``` java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() {
        return re;
    }

    public double imaginaryPart() {
        return im;
    }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im, re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp, (im * c.re - re * c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        return Double.compare(c.re, re) == 0 && Double.compare(c.im, im) == 0;
    }

    @Override
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```
- 사칙연산 메서드들은 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어서 반환한다.(함수형 프로그래밍)
  - 함수형 프로그래밍이란? 피연산자에 함수를 적용해 그 겨과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴 
  - Tips : 함수이름도 동사 add, 대신 전치사 plus를 사용했다. 객체 값을 변경하지 않는다는 사실을 강조.
  - BigInteger, BigDecimal을 사람들이 잘못사용해 오류가 발생하는 경우가 자주 있다.
</br></br>

### 2. 불변 클래스의 장점

- 불변의 객체는 단순하다.
  - 생성 시점의 상태를 파괴되기 전까지 그대로 간직한다.
- 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다.
  - 불변 객체에 대해서는 그 어떤 스레드도 다른 스레드에 영향을 줄 수 없으니 안심하고 공유할 수 있다.
  - 따라서 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용하기를 권한다.
  - 가장 쉬운 재활용 방법은 자주 쓰이는 값들을 상수(public static final)로 제공하는 것이다.
예컨대 Complex 클래스는 다음 상수들을 제공할 수 있다.
    ``` java
    public static final Complex ZERO = new Complext(0, 0);
    public static final Complex ONE = new Complext(1, 0);
    public static final Complex I = new Complext(0, 1);
    ```

- 불변 클래스는 정적 팩토리를 제공할 수 있다.
  - 불변 클래스는 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩토리를 제공할 수 있다. 박싱된 기본 타입 클래스 전부와 BigInteger가 여기에 속한다.
- 정적 팩토리를 사용하면 여러 클라이언트가 인스턴스를 공유하여 메모리 사용량과 가비지 컬렉션 비용이 줄어든다.
  - 새로운 클래스를 설계할 때 public 생성자 대신 정적 팩토리를 만들어두면 클라이언트를 수정하지 않고도 필요에 따라 캐시 기능을 나중에 덧붙일 수 있다.
- 불변 객체는 방어적 복사가 필요하지 않다.
  - 불변 객체는 아무리 복사해봐야 원본과 똑같으니 복사 자체가 의미가 없다.
  - 그러니 불변 클래스는 clone 메소드나 복사 생성자를 제공하지 않는 게 좋다.
- 불변 객체끼리는 내부 데이터도 공유할 수 있다.
  - 예컨대 BigInteger 클래스는 내부에서 값의 부호와 크기를 따로 표현한다.
  - 부호에는 int 변수를 크기(절댓값)에는 int 배열을 사용한다.
  - 여기서 negate 메소드는 크기가 같고 부호만 반대인 새로운 BigInteger를 생성하는데, 이 때 배열은 비록 가변이지만 복사하지 않고 원본 인스턴스와 공유해도 된다. 그 결과 새로 만든 BigInteger 인스턴스도 원본 인스턴스가 가리키는 내부 배열을 그대로 가리킨다.
- 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.
  - 값이 바뀌지 않는 구성요소들로 이뤄진 객체라면 그 구조가 복잡하더라도 불변식을 유지하기 수월하기 때문이다.
  - 좋은 예로 불변 객체는 Map의 키와 Set의 원소로 쓰기에 적합하다. 맵이나 Set은 안에 담긴 값이 바뀌면 불변식이 허물어지는데, 불변 객체를 사용하면 그런 걱정은 하지 않아도 된다.
- 불변 객체는 그 자체로 실패 원자성을 제공한다.(실패 원자성)
  - 상태가 절대 변하지 않으니 잠시라도 불일치 상태에 빠질 가능성이 없다.


### 3. 불변 클래스의 단점

- 값이 다르면 반드시 독립된 객체로 만들어야 한다.
  - 불변 객체는 값의 가짓수가 많다면 이들을 모두 만드는 데 큰 비용을 치러야 한다는 단점이 있다.
  - 예를 들어 백만 비트짜리 BigInteger에서 비트 하나를 바꿔야 한다고 하면 원본과 단지 한 비트만 다른 백만 비트짜리 BigInteger 인스턴스를 생성한다. 이 연산은 BigInteger의 크기에 비례해 시간과 공간을 잡아먹는다.

    > 예시> immutable case bit : 하나를 변경하는데 자원낭비가 심함
    ``` java
    BigInteger moby = ...;
    moby = moby.flipBit(0);    // 새로운 BigInteger 인스턴스 생성 : 
    ```
    
    > 예시> non-immutable case : 가변
    ``` java
    BigSet moby = ...;
    moby.flip(0);    // BigSet 인스턴스
    ```

- 값이 다를 때마다 독립된 객체를 만드는 경우의 주의사항
원하는 객체를 완성하기까지의 단계가 많고, 그 중간 단계에서 만들어진 객체들이 모두 버려진다면 성능이 저하될 것이다. 이 문제에 대처하는 방법은 두 가지다.

#### (1) 흔히 쓰일 다단계 연산(multistep operation)들을 예측하여 기본 기능으로 제공 ####

다단계 연산을 기본으로 제공한다면 더 이상 각 단계마다 객체를 생성하지 않아도 된다.
예컨대 BigInteger는 모듈러 지수 같은 다단계 연산 속도를 높여주는 가변 동반 클래스(companion class)를 package-private으로 두고 있다. 
![image](https://user-images.githubusercontent.com/5934737/150108068-2e81a00c-4582-4b7a-9977-4f1446f75048.png)

클라이언트들이 원하는 복잡한 연산들을 정확히 예측할 수 있다면 package-private의 가변 동반 클래스만으로 충분하다.
> 예 : 소수인 이 BigInteger보다 큰 첫 번째 정수를 반환
![image](https://user-images.githubusercontent.com/5934737/150108003-689b741b-8851-4067-9b5a-f04cb6370ebb.png)

#### (2) 다단계 연산을 제공하는 가변 동반 클래스를 public으로 제공 #### 

클라이언트들이 원하는 복잡한 연산들을 예측할 수 없다면 이 클래스를 public으로 제공하는 것이 최선이다.
대표적인 예는 String 클래스로 String은 가변 동반 클래스인 StringBuilder/StringBuffer 를 제공하고 있다.

### 4. 불변 클래스를 만드는 또 다른 설계 방법
앞에서 말했듯 불변 클래스를 만드는 가장 쉬운 방법은 final로 선언하는 방법이지만 더 유연한 방법이 있다.
    
    > 생성자를 private 혹은 package-private으로 만들고 public 정적 팩토리를 제공하는 방법
    ``` java
    public class Complex {
        private final double re;
        private final double im;

        private Complex(double re, double im) {
            this.re = re;
            this.im = im;
        }

        public static Complex valueOf(double re, double im) {
            return new Complex(re, im);
        }
    }
    ```
    
바깥(client로 칭함)에서는 사실상 final로 public이나 protected 생성자가 없으니 다른 패키지에서는 이 클래스를 확장하는 게 불가능하다.
정적 팩토리 방식은 다수의 구현 클래스를 활용한 유연성을 제공하고, 다음 릴리즈에서 객체 캐싱 기능을 추가해 성능을 끌어올릴 수도 있다.

불변 객체가 아닌 객체를 인수로 받는 상황에서 이 값들이 불변이어야 클래스의 보안을 지킬 수 있다면 방어적으로 복사해 사용해야 한다.

    ``` java
    /**
     * BigInteger, BigDecimal은 불변 클래스로 사용되지만 설계 당시 잘못된 설계로
     * 모든 메소드가 재정의될 수 있게 설계되었다. 
     * 따라서 인수로 받은 BigInteger가 이를 상속한 하위 클래스의 인스턴스인지
     * 확신할 수 없으니 이를 가변이라 가정하고 방어적으로 복사해 사용해야 한다.
     */
    public static BigInteger safeInstance(BigInteger val) {
        return val.getClass() == BigInteger.class ? val : new BigInteger(val.toByteArray());
    }
    ```
성능을 위해 불변 클래스의 규칙을 완화할 수 있다.
"모든 필드가 final이고 어떤 메소드도 그 객체를 수정할 수 없어야 한다"는 원칙은 과한 감이 있어서 성능을 위해 다음처럼 완화할 수 있다. → "어떤 메소드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없다."
어떤 불변 클래스는 계산 비용이 큰 값을 나중에 계산하여 final이 아닌 필드에 캐시해놓기도 한다. 똑같은 값을 다시 요청하면 캐시해둔 값을 반환하여 계산 비용을 절감하는 것이다.
private int hashCode;

// 처음 불렸을 때 해시 값을 계산해 캐시한다.
@Override
public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
직렬화할 때 추가로 주의할 점
Serializable을 구현하는 불변 클래스의 내부에 가변 객체를 참조하는 필드가 있다면 readObject나 readResolve 메소드를 반드시 제공하거나, ObjectOutputStream.writeUnshared와 ObjectInputStream.readUnshared 메소드를 사용해야 한다.
플랫폼이 제공하는 기본 직렬화 방법이면 충분하더라도 말이다. 그렇지 않으면 공격자가 이 클래스로부터 가변 인스턴스를 만들어낼 수 있다.

> 정리
  > 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
  > 불변 클래스는 장점이 많으며, 단점이라곤 특정 상황에서의 잠재적 성능 저하뿐이다.
  > 단순한 값 객체는 항상 불변으로 만들자.
  > String과 BigInteger처럼 무거운 값 객체도 불변으로 만들 수 있는지 고심하라.
  > 성능 때문에 어쩔 수 없다면 불변 클래스와 쌍을 이루는 가변 동반 클래스를 public 클래스로 제공하라.
  > 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
  > 객체가 가질 수 있는 상태의 수를 줄이면 그 객체를 예측하기 쉬워지고 오류가 생길 가능성이 줄어든다.
  > 그러니 꼭 변경해야 할 필드를 뺀 나머지 모두를 final로 선언하자.
  > 즉, 다른 합당한 이유가 없다면 모든 필드는 private final 이어야 한다.
  > 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.
  > 확실한 이유가 없다면 생성자와 정적 팩토리 외에는 그 어떤 초기화 메소드도 public으로 제공해서는 안된다.
  > 객체를 재활용할 목적으로 상태를 다시 초기화하는 메소드도 안된다. 복잡성만 커지고 성능 이점은 거의 없다.
