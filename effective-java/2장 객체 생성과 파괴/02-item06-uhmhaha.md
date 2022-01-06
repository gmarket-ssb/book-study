## [Item06] 불피요한 객체를 생성을 피하라
1. 객체를 재사용하는 것을 고려해라
2. 성능이 중요한 상황에서 불변하는 객체는 직접 생성하여 캐싱하자.
3. 객체 풀은 DB connection등을 제외하고는 직접만들어사용하지말자( 요즘 JVM 성능은 좋다. ) 
4. Unboxing vs AutoBoxing 고려하자 ( primitive type 과 Wrapper class, feat. Static method in Wrapper class )
</br></br>

## **Examples **
### 1.객체를 재사용하는 것을 고려해라

- **객체 재사용의 안좋은 예**
``` java
String two = new String("abc"); //  on the stack 'two' variables
```
이 방식을 이용하면 똑같은 문자열을 생성하더라도 항상 새로운 객체를 생성하므로 낭비가 됩니다.
- **리터럴 방식**
``` java
String str = "hello";   //  on the heap 'str' variables
```
`리터럴` 방식을 사용하면 JVM에서 동일한 문자열이 존재한다면 그 리터럴을 재사용합니다. 
</br></br>

- **어댑터**
어댑터는 인터페이스를 통해서 다른 객체의 연결해주는 객체라 여러개 만들 필요가 없습니다.</br>
대표적으로 `Map` 인터페이스에 있는 `keySet` 메서드는 호출 될 때마다 새로운 객체가 나오는 게 아니라 같은 객체를 반환하기 때문에 리턴받은 `Set` 타입의 객체를 변경하면, 결국 그 뒤에 있는 `Map` 인터페이스도 변경되는 것입니다.
</br></br>

### 2.성능이 중요한 상황에서 불변하는 객체는 직접 생성하여 캐싱
생성 비용이 비싼 객체들 같은 경우 캐싱해서 재사용할 수 있는 방법을 고려해보는 게 좋습니다.
- **before**
``` java
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
String.matches`는 내부에서 만드는 `Pattern`객체를 만들어서 사용하는데, 이 메서드에서는 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상
</br>
- **after**
``` java
public class RomanNumber {

    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }

}
```
</br>
성능을 개선한 코드 수정입니다.`Pattern`인스턴스 클래스 초기화 과정에서 직접 생성하고 캐싱해두었다가 해당 isRomanNumeral가 호출될 때마다 인스턴스를 재사용
</br></br>

### 4. Unboxing vs AutoBoxing 고려하자 ( primitive type 과 Wrapper class )
- 자동 박싱(AutoBoxing)과 자동 언박싱(AutoUnBoxing)
``` java
public class Wrapper_Ex {
    public static void main(String[] args)  {
        Integer num = 17; // 자동 박싱
        int n = num; //자동 언박싱
        System.out.println(n);
    }
}
```

### 참고 ####
- Wrapper Class
  - 자바의 자료형은 크게 기본 타입(primitive type)과 참조 타입(reference type)
  - 기본 자료타입(primitive type)을 객체로 다루기 위해서 사용하는 클래스들을 래퍼 클래스(wrapper class)
  - 종류
   - 기본타입(primitive type)	래퍼클래스(wrapper class)

```
byte	Byte
char	Character
int	Integer
float	Float
double	Double
boolean	Boolean
long	Long
short	Short 
```
  - Wrapper class 구조
![image](https://user-images.githubusercontent.com/5934737/148334769-4cfffb46-4de0-4d6a-b726-7327c34ad640.png)

  - Boxing & UnBoxing
![image](https://user-images.githubusercontent.com/5934737/148334686-fb5e3b79-c8e0-4534-ab4b-4b43aff017d1.png)

    - UnBoxing
``` java
public class Wrapper_Ex {
    public static void main(String[] args)  {
        Integer num = new Integer(17); // 박싱
        int n = num.intValue(); //언박싱
        System.out.println(n);
    }
}
``` 
   - 자동 박싱(AutoBoxing)과 자동 언박싱(AutoUnBoxing)
``` java
public class Wrapper_Ex {
    public static void main(String[] args)  {
        Integer num = 17; // 자동 박싱
        int n = num; //자동 언박싱
        System.out.println(n);
    }
}
```
   - 값비교

``` java
public class Wrapper_Ex {
    public static void main(String[] args)  {
        Integer num = new Integer(10); //래퍼 클래스1
        Integer num2 = new Integer(10); //래퍼 클래스2
        int i = 10; //기본타입
		 
        System.out.println("래퍼클래스 == 기본타입 : "+(num == i)); //true
        System.out.println("래퍼클래스.equals(기본타입) : "+num.equals(i)); //true
        System.out.println("래퍼클래스 == 래퍼클래스 : "+(num == num2)); //false
        System.out.println("래퍼클래스.equals(래퍼클래스) : "+num.equals(num2)); //true
    }
}
```       
        
- String pool
  - String 클래스의 immutable(불변)
  - new를 통해 String을 생성하면 Heap영역에 존재
  - 리터럴을 이용할 경우 String constant pool
  - 
``` java
String a = "Hello";
String b = "Hello";
String c = new String("Hello");
System.out.println(a==b);  //true
System.out.println(a==c);  //false
``` 
위에 예제의 결과에서 왜 3가지 객체가 다 같은 값을 가지고 있음에도 불구하고 비교 결과가 다른지 의문을 가진 분들이 있을것이다.

우선 == 연산은 값을 비교하는것이 아니라 같은 메모리를 참조하는지 비교하는 것이다.

a의 경우 Hello 라는 문자열을 String pool 에 넣게 된다.
b의 경우는 이미 같은 문자열이 String pool 에 존재하기에 같은 값을 참조하게 된다.
c의 경우에는 new 연산자를 사용하여 새로운 객체를 명시적으로 생성하도록 했기 때문에 String pool이 아닌 다른 주소값을 참조하게된다.
이러한 이유로 위와 같은 결과를 얻을 수 있다.

String pool 의 위치
String pool 은 java 6 버전까지 Perm 영역이었다.

하지만 Perm 영역은 고정된 사이즈이며 Runtime 에 사이즈가 확장되지 않는다

그래서 intern 되는 String 값이 커지면 OutOfMemoryException을 발생시킬 수 있었고 그에따라 java 7 버전에서 heap 영역으로 String pool 의 위치를 변경하였다.
