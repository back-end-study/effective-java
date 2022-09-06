# 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라

## 2-1. 점층적 생성자 패턴(Telescoping Constructor Pattern)

- `생성자 체이닝` 방식이라고도 불림
- 기존에 선언한 생성자를 재사용 

```java
public class NutritionFacts { 
    private final int servingSize;  // 필수
    private final int servings;     // 필수 
    private final int calories;     // 선택
    private final int fat;          // 선택
    private final int sodium;       // 선택
    private final int carbohydrate; // 선택

    public NutritionFact (int servingSize, int servings) {
        this (servingSize, servings, 0);
    }
    
    public NutritionFacts (int servingSize, int servings, int calories) {
        this (servingSize, servings, calories, 0);
    }
    
    ...

}
```

**장점**

- 중복코드를 줄일 수 있다.

**단점**

- 확장이 어렵다.
- IDE에 도움이 없다면, 파라미터를 파악하기 어렵다.
  - IntelliJ : `command + p` 활용

    
## 2-2. 자바빈즈 패턴(JavaBeans Pattern)

1. 기본 생성자로 객체 생성
2. setter 메서드로 필드 값 할당

```java
public class NutritionFacts { 
    // 기본값이 있다면, 매개변수들을 기본값으로 초기화된다.
    private final int servingSize  = -1;  // 필수, 기본값 없음
    private final int servings     = -1;  // 필수, 기본값 없음
    private final int calories     = 0;
    private final int fat          = 0;
    private final int sodium       = 0;
    private final int carbohydrate = 0;
    
    public NutritionFact () {}
    
    // 세터 메서드들
    public void setServingSize(int val) { servingSize = val;}
    public void setServings(int val) { servings = val;}
    public void setCalories(int val) { calories = val;}
    public void setFat(int val) { fat = val;}
    public void setSodium(int val) { sodium = val;}
    public void setCarbohydrate(int val) { carbohydrate = val;}
    
}
```

```java
NutritionFacts cocaCola = new NutritionFacts();
// 원하는 매개변수의 값만 설정
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

**장점**

- 객체 생성이 간단해진다.

**단점**

- 일관성이 깨짐(문서 의존)
- 불변객체로 만들기 어렵다.

> ✅ **불변객체로 만드는 이유?**

1. 쓰레드에 안전하여 멀티 쓰레드 환경에서 동기화를 고려하지 않아도 된다.
2. 불변객체를 필드로 사용할 때 방어적 복사가 필요없다.
3. 불변객체는 내부상태가 변경되지 않으므로, Map Key 와 Set 요소로 사용하기에 적합하다.
4. 불변객체를 한번 메모리에 할당하게 되면 같은 객체를 계속 호출하여도, 새롭게 할당하지 않아도 되므로 GC       의 성능을 높힐 수 있다.

## 2-3. 빌더 패턴 (Builder Pattern)

> 점층적 생성자 패턴과 자바빈즈 패턴의 대안!

- 직접 객체를 만들지 않고, 정적 팩터리(또는 생성자)를 호출해 빌더 객체를 얻는다.
- 빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어 둠
- 메서드 체이닝(플루언트(fluent) API) 방식

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
  
    public static class Builder { 
        // 필수 매개변수
        private final int servingSize;
        private final int servings;
    
        // 선택 매개변수
        private final int calories;
        private final int fat;
        private final int sodium;
        private final int carbohydrate;
        
        // 필수 매개변수만 사용
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        
        // 선택 매개변수 메서드
        public Builder calories(int val) {
            calories = val;
            return this;
        }
        public Builder fat(int val) {
            fat = val;
            return this;
        }
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
    
    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

### 🍯 Lombok을 활용하자

<img width="582" alt="image" src="https://user-images.githubusercontent.com/42997924/188634894-dd71548e-fc6b-492a-9ed4-78a4c1ab6938.png">

`AllArgsConsturctor`를 활용하여 외부(클라이언트)에서 빌더만 사용하도록 할 수 있다.

<img width="577" alt="image" src="https://user-images.githubusercontent.com/42997924/188635434-32e3ed77-1db5-421b-adc9-086cc1e03c7b.png">

### 계층형 빌더

#### 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.

> self()를 사용하면 형변환을 줄일 수 있다!

추상 클래스는 추상 빌더를, 구체 클래스는 구체 빌더를 갖게 한다.

계층적으로 설계된 클래스와 잘 어울리는 빌더 패턴

```java
public abstract class Pizza {
  public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
  final Set<Topping> toppings;
  abstract static class Builder<T extends Builder<T>> {
    EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
    
    // 피자 생성 시 "Topping"은 필수로 들어가야 한다.
    public T addTopping(Topping topping) {
      toppings.add(Objects.requireNonNull(topping));
      return self();
    }
    
    abstract Pizza build();
    
    // 하위 클래스는 이 메서드를 재정의(overriding)하여
    // "this"를 반환하도록 해야 한다.
    // self 타입이 없는 자바를 위한 우회 방법이며,
    // 시뮬레이트한 셀프 타입 (simulated self-type) 관용구라 한다.
    protected abstract T self();
  }
  Pizza(Builder<?> builder) {
    toppings = builder.toppings.clone(); // 아이템 50 참조
  }
}
```

하위 클래스의 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다.

이렇게 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌,
그 하위 타입을 반환하는 기능을 **`공변환 타이핑`** 이라 한다.

```java
public class NyPizza extends Pizza { 
  public enum Size { SMALL, MEDIUM, LARGE }
  private final Size size;
  
  public static class Builder extends Pizza.Builder<Builder> {
    private final Size size;
    // 피자 크기는 필수 매개변수
    public Builder(Size size) {
      this.size = Objects.requireNonNull(size);
    }
    @Override public NyPizza build() {
      return new NyPizza(this);
    }
    @Override protected Builder self() { return this; }
  }
  private NyPizza(Builder builder) {
    super(builder);
    size = builder.size;
  }
  @Override public String toString() {
    return toppings + "로 토핑한 뉴욕 피자";
  }
}
```

```java
public class PizzaTest {
  public static void main(String[] args) {
    NyPizza pizza = new NyPizza.Builder(SMALL)
        .addTopping(SAUSAGE)
        .addTopping(ONION).build();

    Calzone calzone = new Calzone.Builder()
        .addTopping(HAM).sauceInside().build();
    
    System.out.println(pizza);
    System.out.println(calzone);
  }
}
```

**장점**

- 객체 생성에 유연하다.
  - 클라이언트에서 필요한 필드의 모든 경우를 선택적으로 생성 가능
- 확장성이 뛰어나다.
  - API는 시간이 지날수록 파라미터가 많아지기 때문에 확장성이 중요하다.

**단점**

- 필수 값을 지정할 수 없음


생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다. 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다.
빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.

---

질문1) checked exception과 unchecked exception의 차이?

질문2) 간혹 메소드 선언부에 unchecked exception을 선언하는 이유는?

질문3) checked exception은 왜 사용할까?