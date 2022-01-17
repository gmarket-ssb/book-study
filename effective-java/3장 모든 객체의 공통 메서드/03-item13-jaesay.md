## Clone 재정의는 주의해서 진행하라
모든 객체의 공통 메소드 중 clone 메소드를 알아봅니다.
### clone 메소드를 오버라이드 하는 방법

1. clone 메소드를 재정의하기 위해서는 Cloneable 인터페이스를 구현해야 한다. 하지만 Cloneable 인터페이스를 상속받았기 때문에 절대로 발생할 수 없는 예외를 try~catch로 해결해줘야한다. (throws로 던져줘도 되지만 검사 예외를 던지지 않아야 그 메소드를 사용하기 편하기 때문에 이런식으로 해결해주자(item 71))
    ```java
    @Override
    public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); // cloneable을 상속받았기 때문에 절대로 오류가 발생하지 않는다
        }
    }
    ```
2. clone 메소드는 protected 메소드이기 때문에 Cloneable을 구현하는 클래스는 clone을 재정의해야 한다. 이때 접근 제한자는 pulbic으로, 반환 타입은 클래스 자기 자신으로 변경한다.
    ```java
    protected Object clone() throws CloneNotSupportedException
    {
        return J9VMInternals.primitiveClone(this);
    }
    ```
3. Shallow copy가 아닌 deep copy를 해주기 위해서는 가장 먼저 super.clone을 호출한 후 필요한 필드를 전부 적절히 수정해야 한다. 단, 기본 타입 또는 불변객체를 참조한다면 수정할 필요없다. 또한, 가변객체를 참조하는 필드는 final로 선언하라는 일반 용법과도 충돌한다.
    ```java
    public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            // final이면 안됨
            result.elements = elements.clone(); // element 배열의 clone을 재귀적으로 호출
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
    
    public Class HashTable implements Cloneable {
        //...
        // 이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
        // 리스트의 원소 수만큼 스택 프레임 소비하여 오버플로우가 걱정된다면 반복문으로
        public static class Entry {
            Entry deepCopy() {
                return new Entry(key, value, next == null ? null : next.deepCopy());
            }
        }
        @Override
        public HashTable clone() {
            try {
                HashTable result = (HashTable) super.clone();
                result.buckets = new Entry[buckets.length];
                for (int i = 0; i < buckets.length; i++) {
                    if (buckets[i] != null) result.buckets[i] = buckets[i].deepCopy();
                }
            } catch (CloneNotSupportedException e) {
                throw new AssertionError();
            }
        }
    }
    ```
4. 상속용 클래스는 Cloneable을 구현해서는 안 된다. (보충 설명 추가, 84p 및 아이템 19)
5. 쓰레드 세이프하게 클래스를 작성할 때는 동기화해줘야 한다. (구현 코드 추가)

### clone 재정의 문제
1. 허술하게 기술된 일반 규약
2. 언어 모순적이고 위험천만한 객체 생성 메커니즘
    1. 생성자를 쓰지 않음 ⇒ 네이티브 메소드이기 때문?
    2. shallow copy 및 불변식 문제?
3. 정상적인 final 필드 용법과 충돌
4. 불필요한 예외 검사 및 형변환

### 해결 방법
Conversion constructor / Conversion factory 로 더 나은 객체 복사 방식을 제공할 수 있다.
```java
// Conversion constructor 
public Yum(Yum yum) { ... };

// Conversion factory
public static Yum newInstance(Yum yum) { ... }; 
```
Conversion constructor / Conversion factory 란 단순히 자기 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자/스태틱 팩토리 메소드를 말한다. 위 clone 메소드 재정의 문제들도 피할 수 있으며 인터페이스 타입의 인스턴스를 인수로 받을 수도 있다. 원본의 구현 타입에 얽매이지 않고 본제본의 타입을 직접 선택할 수 있다. 관례상 모든 범용 컬렉션 구현체는 Collection이나 Map 타입을 받는 생성자를 제공한다.