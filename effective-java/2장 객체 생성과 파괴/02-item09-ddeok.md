## try-finally보다는 try-with-resource를 사용하라

<br>

자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.   
- InputStream, OutputStream, java.sql.Connection   

이러한 resource를 닫는건 클라이언트가 놓치기 쉬워 예측할 수 없는 성능 문제로 이어지기도 한다.   
그래서 안전망으로 finalizer를 활용하고 있는데 그리 믿을만 하지 못하다. (GC에서 우선순위가 낮아 자원회수가 늦음 - 아이템8)   

아래 코드 처럼 우리는 전통적으로 닫힘을 보장하는 수단으로 try-finally를 사용했다.

```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    
    try {
        return br.readLine();    
    } finally {
        br.close();
    }
}
```
나쁘지 않지만 자원을 하나 더 사용한다면?   

```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```
자원이 둘 이상이면 try-finally 방식은 너무 지저분하다!

믿기 힘들겠지만 2007년 당시 자바 라이브러리에서 close 메서드를 제대로 구현한 비율은   
겨우 1/3 정도이다.   

finally 문을 제대로 사용한 앞의 두 코드 예제에도 미묘한 결점이 있다.   
예외는 try 블록과 finally 블록 모두에서 발생할 수 있는데, 위의 firstLineOfFile 메서드에서   
파일을 제대로 읽지 못한다면 readLine 메서드가 예외를 던지고 같은 이유로 close도 실패 하게 된다.   

이런 상황 일 경우 두번째 예외가 첫번 째 예외를 덮어 stacktrace 내역에 첫번째 readLine 예외에 대한   
정보는 남지 않아 디버깅을 어렵게 한다. (일반적으로 문제를 진단하려면 첫번 째 예외를 보고 싶어함)   

이러한 문제들은 자바7의 try-with-resource 덕에 해결 됐고, 이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현하면 된다.
(AutoCloseable은 단순히 void 를 반환하는 close 메서드 하나만 정의 된 인터페이스)   
많은 서드파티 라이브러리 들이 AutoCloseable를 구현 해뒀음.   

try-with-resource를 사용해 첫번째 예제를 재작성하면
```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();    
    }
}
```

try-with-resource를 사용해 두번째 예제를 재작성하면
```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0) {
            out.write(buf, 0, n);
        }
    }
}
```

짧고 읽기 수월할 뿐만 아니라 문제 진단에도 좋다.
firstLineOfFile 메서드에서 readLine과 코드에는 없는 close 양쪽에서 예외가 발생하면,
close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외만 기록 된다. (close는 suppressed: 숨겨졌다는 꼬리표를 달고 출력)   
필요시 suppressed exception은 getSuppresed() 메서드로 가져올 수 있음

보통의 try-finally처럼 try-with-resources에서도 catch절을 쓸 수 있다.   
catch 절 덕분에 try 문을 더 중첩하지 않고도 다수의 예외를 처리할 수 있다.

ex) firstLineOfFile 코드에서 파일을 열거나 데이터를 읽지 못했을 때 예외를 던지는 대신 기본값을 반환하는 코드

```java
static String firstLineOfFile(String path, String defaultVal) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();    
    } catch (IOException e) {
        return defaultVal;
    }
}
```

JAVA 9 이후로는 아래처럼 가능
```java
final Scanner scanner = new Scanner(new File("testRead.txt"));
PrintWriter writer = new PrintWriter(new File("testWrite.txt"))

        try (scanner;writer) {
    // omitted
}
```

### 핵심정리   
- 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고 try-with-resource를 사용하자   
- 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 유용하다