
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

### 2. 불변의 객체

- 지역변수를 선언할 때, 해당 scope에서만 선언하도록 만들어라.
- for문 내부에, 사용하는 변수는 그 외부에 선언을 해놓으면 GC가 자동으로 정리해주지 못한다.
</br></br>

### 3. 캐시를 사용를 사용하여 객체 참조하는 경우
> GC vs non-GC
- GC 하지 않는 경우 : GC의 기준 **'힙 영역의 객체들 중에서 스택에서 도달 불가능한(Unreachable)한 객체'**
  - Strong References( non-GC )
      ``` java
      Integer prime = 1;
      ```
  - Soft References( in GC time )
      ``` java
      Integer prime = 1;  
      SoftReference<Integer> soft = new SoftReference<Integer>(prime); 
      prime = null;
      ```
- GC 하는 경우
  - Weak References( Immediately )
      ``` java
      Integer prime = 1;  
      WeakReference<Integer> soft = new WeakReference<Integer>(prime); 
      prime = null;
      ```
> WeakHashMap 활용 : 캐시의 키에 대한 레퍼런스가 캐시 밖에서 필요 없어지면 해당 엔트리를 캐시에서 자동으로 비워줌
- 예시코드1
    ``` java
	    WeakHashMap<UniqueImageName, BigImage> map = new WeakHashMap<>();
	    BigImage bigImage = new BigImage("image_id");
	    UniqueImageName imageName = new UniqueImageName("name_of_big_image");

	    map.put(imageName, bigImage);
	    assertTrue(map.containsKey(imageName));

	    mageName = null;
	    System.gc();

	    await().atMost(10, TimeUnit.SECONDS).until(map::isEmpty);
    ``` 
 
	- key 레퍼런스가 쓸모 없어졌다면, (key - value) 엔트리를 GC의 대상이 되도록해 캐시에서 자동으로 비워준다
	  ``` java  
		Object key = new Object();
		Object value = new Object();

		Map<Object, List> cache = new WeakHashMap<>();
		cache.put(key, value);
	  ```
- 참고 : WeakReference
  -  WeakReference : Strong 레퍼런스를 Weak 레퍼런스로 감싸면 Weak 레퍼런스가 된다.:
	``` java
		WeakReference weakWidget = new WeakReference(widget);
	``` 

## References

- [Guide to WeakHashMap in Java]( https://www.baeldung.com/java-weakhashmap )
- [java variable scope](https://www.geeksforgeeks.org/variable-scope-in-java/)
