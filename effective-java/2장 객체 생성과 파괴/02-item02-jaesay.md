## 생성자에 매개변수가 많다면 빌더를 고려하라
객체 생성방식 중 빌더 패턴을 알아봅니다.
### 점층적 생성자 패턴(telescoping constructor pattern)
필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, ... 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식이다.
```java
public class NutritionFacts {
    private final int servingSize; // (ml, 1회 제공량) 필수
    private final int servings; // (회, 총 n회 제공량) 필수
    private final int calories; // (1회 제공량당) 선택
    private final int fat; // (g/1회 제공량) 선택
    private final int sodium; // (mg/ 1회 제공량) 선택
    private final int carbohydrate; // (g/1회 제공량) 선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}

// 객체 생성 시
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```
#### 장점
- 호출코드가 간결하다
#### 단점
- 읽고 쓰기 어렵다.  
- (특히 같은 타입일 경우) 실수로 버그를 유발하기 쉽다.

### 자바빈즈 패턴(JavaBeans pattern)
Setter를 통해 매개변수 값을 설정하는 방식이다.  
```java
public class NutritionFacts {
    // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
    private int servingSize = -1; // 필수; 기본값 없음
    private int servings = -1; // 필수; 기본값 없음
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {}
    // 불변식(invariant) 추가
    public void setServingSize(int servingSize) {this.servingSize = servingSize;}
    public void setServings(int servings) {this.servings = servings;}
    public void setCalories(int calories) {this.calories = calories;}
    public void setFat(int fat) {this.fat = fat;}
    public void setSodium(int sodium) {this.sodium = sodium;}
    public void setCarbohydrate(int carbohydrate) {this.carbohydrate = carbohydrate;}
}

// 객체 생성 시
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```
#### 장점
- 코드가 조금 길어지긴 했지만 `set변수명`으로 변수에 값을 할당하기 때문에 읽기 쉽다. 
#### 단점
- 객체 하나를 만들려면 메서드를 여러개 호출해야 하고 객체가 완전히 생성되기 전까지는 모두 호출되기 전까지는 일관성이 무너진 상태에 놓이게 된다.  
클래스를 불변으로 만들 수 없다.

### 빌더 패턴(Builder pattern)
점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 겸비한 빌더 패턴이다. 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적팩터리)를 호출해 빌더 객체를 얻는다. 그런 다음 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다. 마지막으로 매개변수가 없는 build 메서드를 호출해 필요한 (보통은 불변) 객체를 얻는다.
```java
public class NutritionFacts {
    private final int servingSize; // (ml, 1회 제공량) 필수
    private final int servings; // (회, 총 n회 제공량) 필수
    private final int calories; // (1회 제공량당) 선택
    private final int fat; // (g/1회 제공량) 선택
    private final int sodium; // (mg/ 1회 제공량) 선택
    private final int carbohydrate; // (g/1회 제공량) 선택

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        // 불변식 추가
        public Builder calories(int val) {calories = val;return this;}
        public Builder fat(int val) {this.fat = val;return this;}
        public Builder sodium(int val) {this.sodium = val;return this;}
        public Builder carbohydrate(int val) {this.carbohydrate = val;return this;}
        public NutritionFacts build() {return new NutritionFacts(this);}
    }

    private NutritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.calories = builder.calories;
        this.fat = builder.fat;
        this.sodium = builder.sodium;
        this.carbohydrate = builder.carbohydrate;
    }
}

// 객체 생성 시
NutritionFacts cocaCola = new Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();

// Lombok 사용 시
@Data
@Builder(builderMethodName = "hiddenBuilder")
public class NutritionFacts {
    private final int servingSize; // (ml, 1회 제공량) 필수
    private final int servings; // (회, 총 n회 제공량) 필수
    @Builder.Default private final int calories = 0; // (1회 제공량당) 선택
    @Builder.Default private final int fat = 0; // (g/1회 제공량) 선택
    @Builder.Default private final int sodium = 0; // (mg/ 1회 제공량) 선택
    @Builder.Default private final int carbohydrate = 0; // (g/1회 제공량) 선택

    public static NutritionFactsBuilder builder(int servingSize, int servings) {
        // 불변식 추가
        return hiddenBuilder().servingSize(servingSize).servings(servings);
    }
}

// 객체 생성 시
NutritionFacts cocaCola = NutritionFacts.builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();

// toBuilder = true 하면 near copy된 객체를 만들 수 있다. 
NutritionFacts cocaColaZero = cocaCola.toBuilder().calories(0).build();
```
#### 장점
- 변수명(값) 메소드 사용 및 메소드 체이닝을 통해 더 읽고 쓰기 쉽다.

#### 단점
- Builder부터 만들어야 하기 때문에 생성비용이 든다.
- 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다.

### 핵심정리
생정자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다. 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드가 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.