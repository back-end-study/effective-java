## 아이템 38 확장할 수 있는 열거타입이 필요하면 인터페이스를 사용하라

열거타입은 거의 모든 상황에서 타입 안전 열거 패턴보다 우수하다
그러나 타입안전열거 패턴은 **확장** 할 수 있으나 열거 타입은 그럴 수 없다.

### 타입 안전 열거패턴의 형태

```java
class Suit {
    private final String name;

    public static final Suit CLUBS = new Suit("clubs");
    public static final Suit DIAMONDS = new Suit("diamonds");
    public static final Suit HEARTS = new Suit("hearts");
    public static final Suit SPADES = new Suit("spades");

    private Suit(String name) {
        this.name = name;
    }
}
```

### 확장 열거 패턴의 예시

```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

열거타입을 확장한다는것은 대부분의 상황에서 좋지 않은 생각이다.

1. 확장한 타입의 원소는 기반타입의 원소로 취급하지만 그반대는 성립하지 않는다(?)
2. 기반 타입과 확장된 타입들의 원소 모두를 순회할 방법도 마땅치 않다.
    1. 그냥 enum일 경우 values()를 이용하면 모든 원소를 순회 할 수 있음 그러나 아래 의 확장열거 방식을 사용하면 불가능
3. 확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 복잡하다.
    1. 당연 확장성을 생각하면 고려할게 많아지니 복잡함

그러나 확장한 열거타입이 어울리는 쓰임이 단 하나 있으니 그것은 바로 **연산 코드**이다.

위와같이 설계를 할 경우 BasicOperation 은 확장 할 수 없지만, Operation 은 확장 할 수 있으니 이 인터페이스를 이용하여 연산의 타입으로 사용하면 된다.

```java
/**
 * 새롭게 추가한 연산은 기존 연산을 쓰던 곳이면 어디든 쓸 수 있다.
 * */

public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

개별 인스턴스 수준에서 뿐 아니라 타입수준에서도, 기본 열거 타입대신 확장 열거 타입을 넘겨 확장된 열거타입의 원소 모두를 사용할게 할 수 있다.

```java
// Class 객체를 넘기는 방식 (x = 1, y = 2 대입)

public static<T extends Enum<T> & Operation> void test(Class<T> opEnumType,double x,double y){
        for(Operation op:opEnumType.getEnumConstants()){
        System.out.printf("%f %s %f = %f%n",x,op,y,op.apply(x,y));
        }
        }


/** RESULT
 * 1.000000 ^ 2.000000 = 1.000000
 * 1.000000 % 2.000000 = 1.000000
 */
```

```java
// Collection 을 넘기는 방식 (x = 1, y = 2 대입)

private static void test(Collection<?extends Operation> opSet,double x,double y){
        for(Operation op:opSet){
            System.out.printf("%f %s %f = %f%n",x,op,y,op.apply(x,y));
        }
}

/** RESULT
 * 1.000000 ^ 2.000000 = 1.000000
 * 1.000000 % 2.000000 = 1.000000
 */
```

### 그러나 특정연산에서는 enumMap 과 enumSet 을 사용하지 못한다.

( 영준님 제공 예제! )

![image](https://user-images.githubusercontent.com/43979984/200595015-493e8b23-5080-4d6d-8cb2-155014fe2fac.png)

BasicOperation 과 ExtendedOperation 둘다 Operation 을 상속받았지만 EnumSet 과 EnumMap 에 담지는 못한다.



### 하지만 인터페이스를 이용한 확장 가능한 열거 타입을 흉내 내는 방식에도 한가지 문제가 있다.

1. 열거타입끼리 구현을 상속할 수 없다는 점이다.
2. 연산 기호를 찾고 저장하는 로직이 중복으로 들어간다
 