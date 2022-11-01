# [Item-34] int 상수 대신 열거 타입을 사용하라

배정: 김세용
상태: Effective Java 3E

Java에서 Enum을 지원하기 전에는 아래와 같이 정수 상수를 한 곳에 선언해서 사용하곤 했다

```java
public static final int APPLE_FUJI          = 0;
public static final int APPLE_PIPPIN        = 1;
public static final int APPLE_GRANNY_SMITH  = 2;

public static final int ORANGE_NAVEL        = 0;
public static final int ORANGE_TEMPLE       = 1;
public static final int ORANGE_BLOOD        = 2;
```

이러한 방식을 정수 열거 패턴(int enum pattern)이라고 하는데, 여기에는 단점이 많다

## 정수 열거 패턴(int enum pattern)의 단점

- 타입 안전을 보장할 방법이 없다
    - APPLE 관련 변수만 받도록 제한하고 싶은 경우
- 표현력이 좋지 않다
    - 별도의 이름공간을 지원하지 않기 때문에 접두어를 써서 이름 충돌을 방지하는데 이 경우 휴먼 에러가 발생할 확률이 높다
- 정수 상수는 문자열로 출력하기 다소 까다롭다
    - 값을 출력하거나 디버거로 살펴봐도 단지 숫자로만 보여서 썩 도움이 되지 않는다
- 같은 정수 열거 그룹에 속한 모든 상수를 한 바퀴 순회하는 방법도 마땅치 않다

문자열 열거 패턴(string enum pattern)은 더욱 좋지 않은 방법이다
상수의 의미를 출력할 수 있다는 장점만 있을 뿐, 하드코딩이 불가피하며 오타가 있어도 컴파일러가 확인할 방법이 없다.
또한, 문자열 비교에 따른 성능 저하 역시 발생한다

## 열거 타입(Enum Type)?  [JLS 8.9](https://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.9)

```java
public enum Applie { FUJI, PIPPIN, GRANNY_SMITH }

public enum ORANGE { NAVEL, TEMPLE, BLOOD }
```

C, C++, C# 같은 다른 언어의 열거 타입과 비슷하지만, Java의 Enum은 각 인스턴스가 속성과 로직을 구현할 수 있는 **완전한 클래스**라는 점에서 다른 언어의 열거형 구조보다 유연하다

### 열거 타입(Enum Type) 특징

- 열거 타입 자체는 클래스이다
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개한다
- 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 `final` 이며, 딱 하나만 존재한다(싱글톤 보장)
- 컴파일타임 타입 안전성을 제공한다
- namespace가 있어서 이름이 같은 상수도 공존할 수 있다
- 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다
- 열거 타입은 자신 안의 정의된 상수들의 값을 배열에 담아 반환하는 정적메서드(values)를 제공한다

### 열거 타입은 언제 필요할까?

**각 상수와 연관된 데이터를 해당 상수 자체에 내제시키고 싶은 경우**

태양계의 여덟 행성을 열거 타입으로 작성한 예시이다

각 행성에는 질량과 반지름이 있고, 이 속성을 이용해 표면 중력을 계산할 수도 있다

```java
public enum Planet {
    MERCURY (3.302e+23, 2.439e6),
    VENUS   (4.869e+24, 6.052e6),
    EARTH   (5.975e+24, 6.378e6),
    MARS    (6.419e+23, 3.393e6),
    JUPITER (1.899e+27, 7.149e7),
    SATURN  (5.685e+26, 6.027e7),
    URANUS  (8.683e+25, 2.556e7),
    NEPTUNE (1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면 중력(단위: m / s^2)

    // 중력 상수(단위: in m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // Constructor
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```

→ 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다
→ 열거 타입은 기본적으로 불변이기에 모든 필드는  final이어야 한다
→ 필드는 public으로 해도 되지만 private로 두고 별도의 public접근자 메서드를 두는게 낫다

<aside>
❓ **열거 타입에서 상수를 하나 제거하면 어떻게 될까?**

</aside>

→ 제거한 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다

→ 참조하고 있는 클라이언트가 있다 하더라도 컴파일 단계에서 알 수 있다

## 열거 타입을 잘 쓰기위한 여러 사용 방법

### **열거 타입의 상수마다 다른 동작을 해야하는 경우**

```java
public enum Operation {
	PLUS, MINUS, TIMES, DIVIDE;

	// 상수가 뜻하는 연산을 수행한다
	public double apply(double x, double y) {
		switch(this) {
			case PLUS:    return x + y;
			case MINUS:   return x - y;
			case TIMES:   return x * y;
			case DIVIDE:  return x / y;
		}
		throw new AssertionError("알 수 없는 연산: " + this);
	}
}
```

동작은 하지만 그리 예쁘지 않고 깨지기 쉬운 코드이다

새로운 상수를 추가하면 case문도 추가해야 하고, 혹시라도 깜빡한다면 컴파일은 되지만 새로 추가한 연산을 수행하려 할 때 에러가 발생할 것이다

다음은 **상수별 메서드 구현**을 활용한 열거 타입 예시이다

```java
public enum Operation {
    PLUS {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    public abstract double apply(double x, double y);
}
```

apply가 상수 선언 바로 옆에 붙어 있으니 새로운 상수를 추가할 때 apply의 사실을 깜빡하기 힘들 것이다..

또한 apply가 추상 메서드이므로 재정의하지 않는다면 컴파일 오류로 알려준다

```java
public enum Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };
    public final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    public abstract double apply(double x, double y);

    @Override
    public String toString() {
        return symbol;
    }

		public static void main(String[] args) {
    double x = 2D;
    double y = 4D;

    for (Operation op : Operation.values()) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x,y));
    }
		/** 실행 결과
		2.000000 + 4.000000 = 6.000000
		2.000000 - 4.000000 = -2.000000
		2.000000 * 4.000000 = 8.000000
		2.000000 / 4.000000 = 0.500000
		**/
	}
}
```

각 상수에 연결된 데이터로 상수별 심볼(symbol)을 선언해줬고 toString을 재정의해 해당 연산을 의미하는 심볼을 반환하도록 했다. 이를 사용하면 이 사칙연산 열거타입을 편하게 사용할 수 있다.

<aside>
❗ **열거 타입의 toString 메서드를 재정의하려거든, 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공하는 걸 고려해보자**

</aside>

→ 모든 상수의 문자열 표현이 고유하고, 타입 이름을 적절히 바꿀 수 있다면 아래와 같은 메서드를 제공 해보는것을 고려해보자

```java
private static final Map<String, Operation> stringToEnum = Stream.of(values())
            .collect(toMap(Object::toString, e -> e));

// 지정한 문자열에 해당하는 Operation이 존재한다면 해당 상수를 반환
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
} 
```

### **전략 패턴(Strategy Pattern)을 사용하자**

위에서 설명한 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다

급여명세서에서 쓸 요일을 표현하는 열거 타입을 예로 생각해보자

- 열거타입 명: PayrollDay
- 상수: 일주일 각각의 날짜 (MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY)
- 업무시간(분): 8 * 60 (1일 8시간 근무 기준)
- 추가조건
    - 주중 오버타임은 잔업시간이 주어진다.
    - 주말에는 무조건 잔업수당이 주어진다.

```java
public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        
        int overtimePay;
        switch (this) {
						// 주말
            case SATURDAY: case SUNDAY:
                overtimePay = basePay / 2;
                break;
						// 주중
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT 
                        ? 0
                        : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        
        return basePay + overtimePay;
    }
}
```

분명 간결하지만, 여기에는 치명적인 문제가 있다

주말과 주중을 구분하고 주중에 오버타임에 대해서는 구분을 해놨지만, 그 외의 오버타임에 대해 고려가 되지 않았고 휴가나 공휴일 등과 같은 새로운 열거 타입이 추가되면 매번 case 를 추가하거나 조건들을 고쳐주어야 할 것이다

이를 해결 하기위한 가장 깔끔한 방법은 새로운 상수를 추가할 때 잔업수당 **전략**을 선택하도록 하는 것이다

```java
enum PayrollDay {
    MONDAY(WEEKDAY), 
    TUESDAY(WEEKDAY), 
    WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), 
    FRIDAY(WEEKDAY),

    SATURDAY(WEEKEND), 
    SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```

코드는 조금 복잡해지지만 switch문 없이도 상수별 메서드 구현이 가능하면서도 중복도 최소화하여 동일한 로직이 필요한경우도 유연하게 대처 가능하다

<aside>
❓ **switch문이 없으면 무조건 좋은걸까?**

</aside>

→ 꼭 그렇지만은 않다

기존 열거 타입에 상수별 동작을 혼합해 넣을 경우 switch문이 좋은 선택이 될 수 있다

예를 들어 각 사칙연산의 반대 연산을 반환하는 inverse라는 메서드를 구현해야 한다면 switch문이 적절한 선택이 될 수 있다

```java
public static Operation inverse(Operation op) {
    switch(op) {
        case PLUS: return Operation.MINUS;
        case MINUS: return Operation.PLUS;
        case TIMES: return Operation.DIVIDE;
        case DIVIDE: return Operation.TIMES;
        default: throw new AssertionError("알 수 없는 연산" + op);
    }
}
```

## 그래서 열거 타입을 과연 언제 쓰란 말인가??

**필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자**

→ 태양계 행성, 한 주의 요일, 체스 말처럼 본질적으로 열거 타입인 케이스는 당연하고 메뉴 아이템, 연산 코드, 명확한 객체 타입, 검색 조건 등등 까지도..

**열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다**

→ 열거 타입은 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되었다

## 핵심 정리

- 열거 타입은 정수 상수보다 가독성과 안전성 부분에서 뛰어나다
- 각 상수를 각각 다른 데이터와 동작과 연결시킬 수 있다
- 하나의 메서드가 상수별로 다른 동작을 해야 할 경우 switch가 아니라 상수별 메서드 구현을 사용하자
- 상수 몇몇이 같은 동작을 공유할 경우 전략 열거 타입 패턴을 사용하자