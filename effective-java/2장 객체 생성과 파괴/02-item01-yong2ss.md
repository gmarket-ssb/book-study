## [Item1] 생성자 대신 정적 팩터리 메서드를 고려하라

기본적으로 클래스는 public 생성자를 통해서 인스턴스를 만든다.

```java
public class Company {
    String name;

    public Company() {
    }

    public Company (String name) {
        this.name = name;
    }
}
```

**BUT** 이와 별도로 클래스는 생성자 이외에 그 클래스의 인스턴스를 반환하는 정적 팩토리 메서드를 만들어서 객체를 생성할 수 있다.

```java
public static Company getCompany() {
    return new Company();
}
```

### ▶ **장점 5가지**
#### **(1) 이름을 가질 수 있다.**

```java
public Company (String name) {
    this.name = name;
}

public static Company withName(String name) {
    Company company = new Company();
    company.name = name;
    return company;
}

public static void main() {
    Company a = new Company("gmarket");
    Company b = Company.withName("gmarket");
    //a와는 다르게 네이밍으로 어느정도 반환 될 객체가 무엇인지 유추가 가능하다
}
```

같은 매개변수 시그니처를 갖는 생성자를 가질 수 있다.

```java
public Company (String name) {
    this.name = name;
}

 //불가!!!! (같은 매개변수를 갖는 생성자)
public Company (String address) { 
    this.address = address; 
}
```

하지만 정적 팩토리 메서드를 사용하면 어느정도 커버 가능하다.

```java
public static Company withName(String name) {
    Company company = new Company();
    company.name = name;
    return company;
}

public static Company withAddress(String address) {  
    Company company = new Company();
    company.address = address;
    return company;
}
```

#### **(2) 호출될 때마다 새로운 인스턴스를 새로 생성하지 않아도 된다.**<br> 
**(반드시 새로운 객체를 만들필요없다!)**


불변 클래스이거나 매번 새로운 객체를 만들 필요가 없을 때, 미리 만들어놓았던 인스턴스나 캐시해둔 인스턴스를 반환해서 사용 할 수 있다.

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}

private static final Company GMARKET = new Company();

public static Company getCompany() {
    return GMARKET;
}

public static void main() {
   Company a = new Company(); // 매번 새로운 인스턴스 생성
   Company b = Company.getCompany(); //매번 동일한 GMARKET 인스턴스 반환
}
```

#### **(3) 반환타입의 하위 타입 객체를 반환 할 수 있다.**
반환 할 객체의 클래스를 자유롭게 선택할 수 있는 유연함.

리턴타입의 하위타입도 반환 가능하다.

```java
public static companyInterface withName() {
    return new companyInterface(); 
    //리턴은 인터페이스로 해도 되며, 실제 구현체는 무엇인지 모른다.
}
```
ex) java.util.Collections같은 유틸클래스(?)는 45개에 달하는 인터페이스의 구현체를 갖는다.

반환되는 타입이 List여도 그 종류가 다를 수 있다.

==> api를 사용할 때 알아야 할 개념의 개수와 난이도가 줄어든다.

```java
public interface CompanyInterface() {
    //java8 ~
    public static Company getCompany() {
        return new Company();
    }
    //Collections처럼 사용하지 않더라도, 인터페이스 자체에다가 public static으로 구현체를 추가해도된다.

    //java9 ~ 
    //private static 지원
    private static Company getCompany2(){
        return new Company();
    }
}
```

#### **(4) 입력 매개변수에 따라 매번 다른 클래스의 객체를 리턴 할 수 있다.**
반환타입이 하위타입이기만 하면 어떤 클래스의 객체를 반환해도 상관없다는 장점 3번의 연장선으로, 

예를들어 EnumSet같은 경우에는 원소에 개수에(매개변수) 따라 RegularEnumSet or JumboEnumSet으로 반환한다.

```java
public static Company getCompany(int [] array) {
    return array.size() > 100 ? new Company() : new UniconCompany(); 
}

class UniconCompany extends Company {
}
```

#### **(5) 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.**

```java
public static Company getCompany() {
    Company a = new Company();

    //Company를 상속받은 특정 Company구현체의 FQCN(Fully qualified class name)을 읽어온다. (풀패키지경로를 텍스트로 받는다)
    // 해당하는 인스턴스를 생성
    // a가 해당 인스턴스를 가리키도록 수정
    //ex) Connection connection = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/jdbc", "root", "1234");

    return a;
} 
  ```

ex) JDBC의 경우 
* DriverManager.registerDriver가 프로바이더 등록 api (구현시 등록)
* DriverManager.getConnection()이 서비스 액세스 api (클라이언트가 서비스의 인스턴스를 얻을 때)
* Driver가 서비스 프로바이더 인터페이스의 역할
* Connection이 서비스 인터페이스의 역할 (구현체 동작 정의)

자바 6부터는 java.util.ServiceLoader라는 서비스 프로바이더 제공
<br><br>

### ▶ **단점 2가지**
#### **(1) 상속을 하려면 public이나 protected 생성자가 필요하기 때문에 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.**
하지만 예를들어 Collections같은 경우, 굳이 상속을 할 필요가 없는 클래스라 단점이라고 말하기 애매하다.

#### **(2) 정적 팩토리 메서드는 프로그래머가 찾기 힘들다**
생성자는 JAVADOC 상단에 Summary로 모아서 보여주지만 정적 팩토리 메소드는 API문서에서 다뤄주지 않는다.

ex)SpringApplication.run()

* 클래스 상단에 별도의 주석을 달아서 사용성을 높이자

▼ 아래는 정적 팩토리 메서드에서 주로 사용하는 명명법이다.

|메서드명|내용|예시
|-------|----|----
|from|매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드|Date d = Date.from(instant);
|of|여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드|Set faceCards - EnumSet.of(JACK, QUEEN, KING);
|valueOf|from과 of의 더 자세한 버전|BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
|instance, getInstance|(매개변수를 받는다면)명시한 인스턴스를 반환, 같은 인스턴스임을 보장하지는 않는다.|StackWalker luke = StackWalker.getInstance(options);
|create, newInstance|instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장|Object newArray = Array.newInstance(classObject, arrayLen);
|getType|getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. Type은 팩터리 메서드가 반환할 객체의 타입이다.|FileStore fs = Files.getFileStore(path);
|newType|newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. Type은 팩터리 메서드가 반환할 객체의 타입이다.|BufferedReader br = Files.newBufferedReader(path);
|type|getType과 newType 축소버전|List litany = Collections.list(legacyLitany);
