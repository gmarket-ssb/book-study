## 냄새2. 중복 코드

-   변경이 필요할 때 같은 여러 곳에 변경이 일어나야 하므로 놓쳐서 버그가 발생하기 쉬움

### 함수 추출하기

-   의도와 구현을 분리한다
-   코드(들)을 함수로 분리하고 이름으로 '의도'를 표현한다. => 한 줄 짜리 코드도 괜찮다
-   하나의 함수 안에 주석이 있으면 해당 주석이 설명하는 코드(들)은 함수로 추출할 수 있는지 고민해 볼 수 있다.

```
// before
code 1-1
code 1-2
code 1-3
code 2

// after
doWorksForOne()   <-- 1-1, 1-2, 1-3 코드 모드 읽어보지 않아도 '1'의 일을 한다 로 '의도'를 표현
code2

func doWorksForOne() {   <-- 구현을 함수로 분리
  code 1-1
  code 1-2
  code 1-3
}
```

### 코드 분리하기

-   관련있는 코드들은 같은 구간에 둔다.
    -   '함수 추출하기'의 힌트가 될 수 있다.
    
```
code for work 1-1
code for work 1-2
code for work 1-3
        
code for work 2-1  
code for work 2-2
```

-   변수 선언은 사용하는 곳 바로 위에

```
int a;  
code for a 1-1  
code for a 1-2

int b;  
code for b 2-1  
code for b 2-2
```

> 변수 선언을 상단에 하면 해당 함수가 어떤 변수들을 사용하는 지 직관적으로 알 수 있는 장점이 있다.

### 메소드 올리기

-   여러 중복되는 코드를 상위 클래스로 이동시키고 이를 호출하여 사용한다.
-   일부 값만 다를 경우 함수 매개변수화하기 리팩토링을 적용하여 이 방법을 사용할 수 있다.
-   메소드 들이 비슷한 절차를 따르고 있다면 '템플릿 메소드 패턴'을 고려해볼수 있다.

```

// before  
class ChildA {  
void do() { ~~, workA() }  
}  

class ChildB {  
  void do() { ~~, workB() }
}

// after 1  
class Mother {  
  void do() {
    ~~~
  } // 동일 메소드를 상위 클래스로 올렸다.  
}

class ChildA extends Mother { }  
class ChildB extends Mother { }

// after 2  
class Mother {  
  void do() { ~~, work() }  
  abstract void work();  
}  
class ChildA extends Mother {  
  void work() { ~~ }  
}  
class ChildB extends Mother {  
  void work() { ~~~ }  
}

```
