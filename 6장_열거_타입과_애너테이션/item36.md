# Item 36. 비트 필드 대신 EnumSet을 사용하라

Created: 2022년 10월 30일 오후 3:16
Last Edited Time: 2022년 11월 1일 오후 8:30
Status: 정리 완료

# 비트 필드 대신 EnumSet을 사용하라

열거한 값들이 주로 집합으로 사용되는 경우 각 상수에 서로 다른 2의 거듭제곱을 할당한 정수 열거패턴을 사용해왔다.

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0;
    public static final int STYLE_ITALIC = 1 << 1;
    public static final int STYLE_UNDERLINE = 1 << 2;
    public static final int STYLE_STRIKETHROUGH = 1 << 3;
}
```

`Text.applyStyles(STYLE_BOLD | STYLE_ITALIC);`

이런식으로 여러 상수를 하나의 집합으로 모을 수 있고  이렇게 만들어지는 집합을 비트필드라고 한다.

이 비트필드를 사용하면 비트별 연산을 사용하여 집합 연산을 효율적으로 수행할 수 있다.

## 문제점

- 비트 필드 값이 그대로 출력되면 정수 열거 상수를 출력할때보다 해석하기가 훨씬 어렵다.
- 비트필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다.
- 최대 몇 비트가 필요한지를 API작성 시 미리 예측하여 적절한 타입을 선택해야 한다.

## 해결책

`EnumSet` 클래스는 열거타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.

Set 인터페이스를 완벽하게 구현해놓았고, 타입 안전 + 다른 Set 구현체와 함께 사용이 가능하다.

EnumSet의 내부는 비트 벡터로 구현되었다.

총 원소개수가 64개 이하라면, 대부분의 경우에 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.

removeAll, retainAll 같은 대량의 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현하였다.

```java
public void applyStyles(Set<Style> styles) {
    //...
}
```

Set 인터페이스를 매개변수에서 받는 이유는 직접 EnumSet을 선언해준거보다 나은 이유는

좀 특이한 클라이언트들이 다른 Set 구현체를 여기에 주어도 처리할 수 있기 때문이다.