# [Item-42] 익명 클래스보다는 람다를 사용하라

배정: 익명
상태: Effective Java 3E

- 예전에는 자바에서 함수 타입을 표현할 때 추상메서드를 하나만 담은 인터페이스(드물게는 추상클래스)를 사용했었다
    - 이런 인터페이스의 인스턴스를 “함수객체"라고 하여, 특정 함수나 동작을 나타내는 데 사용했다
    - 1997년 JDK1.1이 등장하면서 함수 객체를 만드는 주요 수단은 익명클래스(아이템 24)가 되었다

```java
// 예시 코드로 문자열을 길이순으로 정렬하는데, 정렬을 위한 비교함수로 익명클래스를 사용한다
Collections.sort(words, new Comparator<String>() {
  public int compare(String s1, String s2) {
    return Integer.compare(s1.length(), s2.length());
  }
});
```

- 전략패턴처럼, 함수 객체를 사용하는 과거 객체 지향 디자인 패턴에는 익명 클래스면 충분했다
    - 위의 코드에서 Comparator 인터페이스가 정렬을 담당하는 추상 전략을 뜻하며 문자열을 정렬하는 구체적인 전략을 익명 클래스로 구현했다
- **하지만 익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않았다**

## 람다식의 등장

- 자바 8에 와서 추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 **특별한 대우**를 받게 되었다
- 지금은 함수형 인터페이스라 부르는 이 인터페이스들의 인스턴스를 람다식(lambda exression)을 사용해 만들 수 있게 된것이다
    - 람다는 함수나 익명 클래스와 개념은 비슷하지만, 코드는 훨씬 간결하다

```java
// 다음은 익명 클래스를 사용한 앞의 코드를 람다 방식으로 바꾼 모습이다 
// 자질구레한 코드들이 사라지고 어떤 동작을 하는지가 명확하게 드러난다
Collections.sort(word, (s1,s2) -> 	Integer.compare(s1.length(), s2.length()));
```

- 여기서 람다, 매개변수 (s1, s2) 반환값의 타입은 코드에서는 언급이 없어도 **컴파일러가 문맥을 살펴 타입을 추론해준다**
    - 컴파일러가 타입을 결정하지 못할 수도 있는데, 그럴 때는 프로그래머가 직접 명시해야 한다
    - 이런 경우는 제네릭의 로 타입 사용에서 발생할 수 있다
        - 로 타입 제네릭 메서드와 람다식을 같이 활용한다면 컴파일러가 타입추론하기 힘들어진다
        - 그 이유는 컴파일러가 타임을 추론하는데 필요한 정보를 제네릭에서 얻기 때문이다
    
    ![스크린샷 2022-11-08 00.08.47.png](%5BItem-42%5D%20%E1%84%8B%E1%85%B5%E1%86%A8%E1%84%86%E1%85%A7%E1%86%BC%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%E1%84%87%E1%85%A9%E1%84%83%E1%85%A1%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%85%E1%85%A1%20b2cc9d0c685e4777b1b6e07baed11517/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2022-11-08_00.08.47.png)
    
    ![스크린샷 2022-11-08 21.16.24.png](%5BItem-42%5D%20%E1%84%8B%E1%85%B5%E1%86%A8%E1%84%86%E1%85%A7%E1%86%BC%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%E1%84%87%E1%85%A9%E1%84%83%E1%85%A1%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%85%E1%85%A1%20b2cc9d0c685e4777b1b6e07baed11517/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2022-11-08_21.16.24.png)
    
- **타입을 명시해야 람다 코드가 더 명확해지는 경우를 제외하고는 모든 매개변수 타입은 생략하도록 하자**

```java
// 람다의 사용법을 알게 되었다면 비교자 생성 메서드를 통해 더 코드를 깔끔하게 작성할 수 있다.
Collections.sort(words,comparingInt(String::length));
```

```java
// 자바 8때 List 인터페이스에 추가된 sort 메서드를 이용하면 더욱 짧아진다.
words.sort(comparingInt(String::length));
```

### [Item 34](https://github.com/back-end-study/effective-java/blob/main/6%EC%9E%A5_%EC%97%B4%EA%B1%B0_%ED%83%80%EC%9E%85%EA%B3%BC_%EC%95%A0%EB%84%88%ED%85%8C%EC%9D%B4%EC%85%98/Item34.md#%EC%97%B4%EA%B1%B0-%ED%83%80%EC%9E%85%EC%9D%98-%EC%83%81%EC%88%98%EB%A7%88%EB%8B%A4-%EB%8B%A4%EB%A5%B8-%EB%8F%99%EC%9E%91%EC%9D%84-%ED%95%B4%EC%95%BC%ED%95%98%EB%8A%94-%EA%B2%BD%EC%9A%B0) 예제 변형시키기

- **람다를 사용하면 열거 타입의 인스턴스 필드를 이용하는 방식으로 상수별로 다르게 동작하는 코드를 쉽게 구현할 수 있다.**
- 단순히 각 열거 타입 상수의 동작을 람다로 구현해 생성자에 넘기고, 생성자는 이 람다를 인스턴스 필드로 저장해둔다, 그 다음 apply 메서드에서 필드에 저장된 람다를 호출하기만 하면 된다

```java
public enum Operation {
	PLUS("+", (x,y) -> x + y),
	MINUS("-", (x,y) -> x - y),
	TIMES("*", (x,y) -> x * y),
	DIVIDE("/", (x,y) -> x / y);

	private final String symbol;
	private final DoubleBinaryOperator op;

	Operation(String symbol, DoubleBinaryOperator op) {
		this.symbol = symbol;
		this.op = op;
	}

	@Override
	public String toString() {
		return symbol;
	}

	public double apply(double x, double y) {
		return op.applyAsdouble(x,y)
	}
}
```

<aside>
💡 이 코드에서 열거 타입 상수의 동작을 표현한 람다를 `DoubleBinaryOperator` 인터페이스 변수에 할당했다. 
`DoubleBinaryOperator`는 `java.util.function` 패키지가 제공하는 다양한 함수 인터페이스 중 하나로 `double` 타입 인수 2개를 받아 `double` 타입 결과를 돌려준다.

</aside>

## 람다식이 무조건 좋은 것일까?

- 람다 기반 Operation 열거 타입을 보면 추상메서드가 더이상 사용할 이유가 없다고 느낄지 모르지만, 꼭 그렇지는 않다
- 메서드나 클래스와 달리 **람다는 이름이 없고 문서화도 못한다** → 코드 자체로 **동작이 명확하게 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다**
- **람다는 한 줄 일때 가장 좋고** **길어야 세줄 안에 끝내는게 좋다**
    - 람다가 길거나 읽기 어렵다면 더 간단히 줄여보거나 람다를 쓰지 않는쪽으로 리팩토링을 해야한다

### **람다는 함수형 인터페이스에서만 쓰인다**

- 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니, 익명클래스를 써야 한다.
- 비슷하게 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때도 익명 클래스를 쓸 수 있다

### 람다는 자신을 참조할 수 없다

- 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다. 반면 익명 클래스에서의 this는 익명 클래스의 인스턴스 자신을 가리킨다.
- 함수 객체가 자신을 참조해야하는 경우, 익명클래스를 사용해야 한다.

```java
public interface LambdaTest {
	int value(int i);
}

public class Test {
    private final int value = 100;

    public LambdaTest anonymous = new LambdaTest() {
        final int value = 200;

        @Override
        public String getValue() {
            return "익명 클래스의 value는? " + this.value; // 익명 클래스는 자신을 가리킴
        }
    };

    public LambdaTest lambda = () -> "람다의 value는? " + this.value; // 람다는 바깥 인스턴스를 가리킴

    public static void main(String[] args) {
        Test test = new Test();
        test.anonymous.getValue();
        System.out.println(test.anonymous.getValue());
        System.out.println(test.lambda.getValue());
    }
    /*
    익명 클래스의 value는? 200
    람다의 value는? 100
    */
}
```

### 람다는 직렬화를 해서는 안된다

- 람다도 익명 클래스처럼 직렬화 형태가 구현별로 다를 수 있다. 그렇기 때문에 직렬화하는 일을 극히 삼가해야 한다.(익명 클래스도 마찬가지)
- 직렬화해야만 하는 함수 객체가 있다면 private static 중첩 클래스의 인스턴스를 사용하도록 하자.([Item 24](https://github.com/back-end-study/effective-java/blob/main/4%EC%9E%A5_%ED%81%B4%EB%9E%98%EC%8A%A4%EC%99%80_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4/item24.md))

## 결론

- 자바 8이 등장하면서 작은 함수 객체를 구현하는데 적합한 람다가 도입 되었다.
- 익명 클래스는 (함수형 인터페이스가 아닌) 타입의 인스턴스를 만들 때만 사용하라.
- 람다는 한 줄이 제일 좋으며 최대 세줄이다. 더 길어진다면 람다를 쓰지 않는 쪽으로 리팩토링하자 !