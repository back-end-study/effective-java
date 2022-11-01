# item36 비트 필드 대신 EnumSet을 사용하라

열거한 값들이 주로 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해 왔다.

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int styles) {
        // ...
    }
}
```

비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 비트 필드라 한다.

`text.applyStyles(STYLE_BOLD | STYLE_ITALIC)`

### 비트 필드 장점
- 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다.

### 비트 필드 단점
- 정수 열거 상수 단점을 그대로 지닌다.
- 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기 훨씬 어렵다.
- 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 까다롭다.
- 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입을 선택해야 한다. API를 수정하지 않고는 비트 수를 더 늘릴 수 없다.

상수 집합을 주고 받아야 할 때는 여전히 비트 필드를 사용하기도 한다.

### 비트 필드 대안 EnumSet 클래스
열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.  
Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다.  
EnumSet의 내부는 비트 벡터로 구현되었다.  
대부분의 경우 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.  
removeAll과 retainAll 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다.

```java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyle(Set<Style> styles) {
        // ...
    }
}
```

`text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));`

모든 클라이언트가 EnumSet을 건네리라 짐작되는 상황이라도 이왕이면 인터페이스로 받는게 일반적으로 좋은 습관이다.(item64)

## 핵심 정리
열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다.  
EnumSet 클래스가 비트 필드 수준의 명료함과 성능을 제공하고 아이템 34에서 설명한 열거 타입 장점까지 선사하기 때문이다.  
EnumSet의 유일한 단점이라면 불변 EnumSet을 만들 수 없다는 것이다.  
(명확성과 성능이 조금 희생되지만) Collections.unmodifiableSet으로 EnumSet을 감싸 사용할 수 있다.  
`Set<Style> test = Collections.unmodifiableSet(EnumSet.of(Style.BOLD));`  

[[openjdk source] EnumSet](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/EnumSet.java)
