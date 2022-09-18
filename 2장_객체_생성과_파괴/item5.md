## item5 자원을 직접 명시하지말고 의존 객체 주입을 사용하라


![](https://velog.velcdn.com/images/rodlsdyd/post/6ef0c9d8-7edf-4c41-bb66-a5cea81d5b17/image.png)

### 자원을 직접명시하는 방법의 문제점


```java
public class SpellChecker {
    private static final Dictionary dictionary = new ADictionary(); // 사전에 따라 변경될 수 있다

    private SpellChecker() {} // Noninstantiable

    public static boolean isValid(String word){...}
    public static List<String> suggestions(String typo) { ... }
}
```
- ADictionary 가 아닌 다른 자원을 사용하려면 코드를 계속 변경해야한다

- SpellChecker의 동작이 주입 자원에 따라 변할수 있으며 재사용이 힘들어진다 
(한국사전,독일사전 등에 따라 SpellChecker의 동작이 변경되어 KoreanSpellChecker, 등을 따로 만들어야한다)

- 테스트에 효율적이지 않다

```java
class SpellCheckerTest{
	
    @Test
    void isValid() {
    	assertTrue(SpellChecker.isValid("test"));
    } // ADictionary를 항상 만들게 된다
}
```

- SpellChcker를 만들때마다 ADictionary를 생성하게된다   
- ADictionary를 생성하는 비용이 큰경우엔 더욱 효율적이지 않다


위방식에서 final을 없애고 다른사전으로 교체하는 메서드 (setter메서드)생성? 
=> 오류내기쉬우며, 멀티스레드 환경에서 쓸수없다

오류 = setter주입으로 생길수있는 Null예외?



### 유연한 방식

```java
public class SpellChecker {
    private final Dictionary dictionary;

    public SpellChecker(Dictionary dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
	//function...
}
```

- Dictionary가 변경되어도 SpellChecker의 코드를 재사용할수있다. ( 단, Dictionary가 인터페이스라는 가정하에)

- 필요할때마다 여러 인스턴스를 사용할수있고 불변 

- 테스트용 주입도 가능해진다


```java
class SpellCheckerTest{
	
    @Test
    void isValid() {
    	SpellChecker spellChecker = new SpellChecker(new MockDictionary());
    	assertTrue(SpellChecker.isValid("test"));
    } // 테스트용 Mock객체로 변경이 가능해진다
}
```

#### 정적팩토리를 넘겨주는 방식
```java
public static SpellChecker staticFactory(Dictionary dictionary) {
    return new SpellChecker(dictionary); // 유연하게 주입
}
```

#### 빌더에 적용

```java
public static Builder builder(Dictionary dictionary) {
   return new Builder(dictionary); // 유연하게 주입
}

public static class Builder{
   private Dictionary dictionary;

   public Builder(Dictionary dictionary) {
      this.dictionary = dictionary;
   }
        //        빌더...
}
```

#### 생성자에 자원팩토리를 넘겨주는 방식

```java
public class SpellChecker {

    private final Dictionary dictionary;

	public SpellChecker(Supplier<Dictionary> dictionarySupplier) {
        this.dictionary = dictionarySupplier.get();
    }
}

	// Main 
     SpellChecker spellCheckerBySupplier = new SpellChecker(() -> new ADictionary());
```




- `Supplier<T>`를 입력받는 메서드는 한정적 와일드카드타입을 사용해 팩터리의 타입매개변수를 제한해야한다

=> 타입에 구체적인 Supllier가 온다고 가정했나? 타입한정자를 사용해서 범위를 제한해라  
ex) `Supplier<MockDictionary>` 대신 `Supplier<Dictionary>`



### 이야기 해볼점 

- 생성자 주입방식의 단점은 하나도 없는지?

- (스프링) 세터주입이나 다른주입 방식이 생성자 주입보다 나은경우가 있는지?
