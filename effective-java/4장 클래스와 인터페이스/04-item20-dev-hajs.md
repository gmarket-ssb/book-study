## 추상 클래스보다는 인터페이스를 우선하라
자바가 제공하는 다중 구현 메커니즘에는 '인터페이스(1.8+)' 와 '추상 클래스' 가 있다.<br>
그런데 왜 인터페이스를 우선해야할까?
<br>

### 인터페이스로 설계 시 얻을 수 있는 이점
* 기존 클래스에 새로운 추상 클래스를 끼워 넣는건 일반적으로 어렵지만, 새로운 인터페이스는 손쉽게 구현해넣을 수 있다.
  ```java
  public class ClassA extends AbstractClassA implements InterfaceA, InterfaceB {}
  ```
* 래퍼 클래스 관용구<sup>(아이템18)</sup> 와 함께 사용하면 인터페이스로 설계 시 기능을 향상시키는 안전하고 강력한 수단으로 만들 수 있다.
  * 반면, 추상 클래스로 타입을 정의해두면 그 타입에 기능을 추가하는 방법은 오직 상속뿐이다.
* 믹스인(mixin) 정의에 안성맞춤이다.
  * mixin : `Comparable` 처럼 자신을 구현한 클래스의 인스턴스들끼리는 순서를 정할 수 있다고 선언할 수 있는 것 처럼<br>
            mixin 을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.
### 그 외 인터페이스만의 강점
* 인터페이스는 인스턴스 필드를 가질 수 없고, public 이 아닌 정적 멤버도 가질 수 없다.
* 인터페이스로는 계층 구조가 없는 타입 프레임워크를 만들 수 있다.
* 구현 방법이 명백하다면 인터페이스의 디폴트 메서드(1.8+)로 제공할 수 있다.
  * 다만 많은 인터페이스가 정의한 `equals` 나 `hashCode` 같은 메서드들은 디폴트 메서드로 제공해서는 안 된다. (=할 수 없다)
   ![image](https://user-images.githubusercontent.com/57446639/150679631-f19449a4-4f53-45b7-9944-4531015d2663.png)
<br>

### 추상 골격 구현 클래스 (Skeletal implementation)
* 인터페이스와 추상 클래스의 장점을 모두 취할 수 있는 방식이다.
* 인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드 몇 개도 함께 제공한다.<br>
  그리고 골격 구현 클래스는 나머지 메서드들까지 구현한다.
* 이러면 단순히 골격 구현을 확장(extends) 하는 것만으로 이 인터페이스를 구현(implements) 하는데 필요한 일이 대부분 완료된다.
* 대표적으로 `AbstractCollection`, `AbstractSet`, `AbstractList`, `AbstractMap` 등등이 있다.

  ![image](https://user-images.githubusercontent.com/57446639/150678947-7bde7960-98f0-4f07-8278-d4142ffeafc2.png)

  ```java
  public abstract class AbstractCollection<E> implements Collection<E> {}
  public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {}
  public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {}
  public abstract class AbstractMap<K,V> implements Map<K,V> {}
  ```
* => 템플릿 메소드 패턴 ([wikipedia](https://en.wikipedia.org/wiki/Template_method_pattern))


### 정리
* 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다.
* 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 꼭 고려해보자.
* 골격 구현은 '가능한 한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다.<br>
  '가능한 한' 이라고 한 이유는, 인터페이스에 걸려 있는 구현상의 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하기 때문이다.
