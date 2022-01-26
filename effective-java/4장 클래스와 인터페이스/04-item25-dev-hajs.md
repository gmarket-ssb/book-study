## 톱레벨 클래스는 한 파일에 하나만 담으라
> A top level class is a class taht is not a nested class. - 공식 문서

소스 파일 하나에 톱레벨 클래스를 여러 개 선언해도 컴파일러는 오류를 뱉지 않는다.<br>
다만, 아무런 득도 없고 파일간 컴파일 되는 순서에 따라 결과 값(동작)이 달라질 수 있는 심각한 위험을 감수해야 하는 행위다.
<br><br>

```java
// A.java
class A {
  static final String NAME = "A";
}
class B {
  static final String NAME = "B";
}

// B.java
class B {
  static final String NAME = "B";
}
class A {
  static final String NAME = "A";
}
```
![image](https://user-images.githubusercontent.com/57446639/151125166-518d974d-79bc-40f5-951e-9f3d5ed4b499.png)
<br><br>

### 정리
* 교훈은 명확하다. **"소스 파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)** 를 하나만 담자.
* 이 규칙만 따른다면 컴파일러가 한 클래스에 대한 정의를 여러 개 만들어내는 일은 사라진다.
* 소스 파일을 어떤 순서로 컴파일하든 바이너리 파일이나 프로그램의 동작이 달라지는 일은 결코 일어나지 않는다.
