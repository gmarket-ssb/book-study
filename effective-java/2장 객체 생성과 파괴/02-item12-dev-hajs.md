## toString을 항상 재정의하라

Object 의 기본 toString 메소드는 단순히 `"클래스 이름@16진수로 표시한 해시코드"` 를 반환한다.<br>
이는 일반적으로 우리가 흔히 원하는 객체의 정보가 아니기 때문에 toString 메소드는 재정의하여 표현해야 바람직하다.
<br><br>

Object 의 toString 메소드 주석을 보면 "모든 하위 클래스에서 이 메서드를 재정의하라" 라는 문구가 존재한다.
> *\* It is recommended that all subclasses override this method.*<br>
<br>

### toString 을 재정의 했을 때 vs toString 을 재정의하지 않았을 때의 출력 값 비교
```java
public class Item11 {
    public static void main(String[] args) {
        Item11ToString item11ToString = new Item11ToString();
        item11ToString.setPhone1("010");
        item11ToString.setPhone2("1234");
        item11ToString.setPhone3("5678");

        /**
         * toString 을 재정의하지 않았을 때
         * > item11ToString = chap3.Item11ToString@3f0ee7cb
         *
         * toString 을 재정의했을 때 - basic format
         * > item11ToString = Item11ToString{phone1='010', phone2='1234', phone3='5678'}
         *
         * toString 을 재정의했을 때 - custom format
         * > item11ToString = 010-1234-5678
         */
        System.out.println("item11ToString = " + item11ToString);
    }
}

@Setter
public class Item11ToString {
    private String phone1;
    private String phone2;
    private String phone3;

    @Override
    public String toString() {
        return "Item11ToString{" +
                "phone1='" + phone1 + '\'' +
                ", phone2='" + phone2 + '\'' +
                ", phone3='" + phone3 + '\'' +
                '}';
//        return phone1 + "-" + phone2 + "-" + phone3;
    }
}

```
<br>

### 정리해서..
toString 메소드를 재정의하지 않으면 발생하는 대표적인 문제점은<br>
객체를 로깅할 때 쓸모없는<i>(필요없는)</i> 메세지만 로그에 남게 되어 결국은 **로깅의 어려움**이 발생한다는 점이다.
<br><br>

그렇다고 모든 클래스가 toString 메소드를 재정의해야 하는건 아니다.
* **정적 유틸리티 클래스**는 (객체를 표현할 필요가 없기 때문에) toString 을 제공할 이유가 없다.
* '대부분의' **열거 타입**은 자바가 이미 완벽한 toString 을 제공한다.
  * -> Enum 도 `return name;` 이라는 이미 정의한 toString 이 있다.
* '대다수의' **컬렉션 구현체**는 추상 컬렉션 클래스들의 toString 메소드를 상속해 사용한다.
  * -> 통상적으로 많이 사용하는 컬렉션 구현체들은 모두 상속받은 각자의 추상화컬렉션에 이미 toString 가 있다. 
