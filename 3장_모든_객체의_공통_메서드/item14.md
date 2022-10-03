# item 14. Comparable을 고려하라

> **알파벳, 숫자, 연대, 번호 등 순서가 있는 값 클래스를 작성하면 반드시 Comparable 를 구현하자.**

- Comparable 인터페이스는 compareTo() 라는 하나의 메서드를 정의한다.
- 이 메서드는 Object 메서드가 아니다.
- 이 메서드의 성격은 Object의 equals와 유사하지만 아래와 같은 차이가 있음
    - 동치성 비교 뿐 아니라, 순서까지 비교가 가능
    - 제네릭하다

> Comparable을 구현했다는 것은 순서가 있음을 뜻함

- 자바에서 제공하는 모든 값 클래스와 열거 타입이 Comparable을 구현했다.
- 수많은 제네릭 알고리즘과 컬렉션의 힘을 누릴 수 있음.

## 1️⃣ compareTo 규약

compareTo 메서드의 일반 규약은 equals과 비슷하다.

Comparable 을 구현한 객체는 다음 규약들을 지켜야 한다.

**특징**

> 해당 객체와 주어진(매개변수) 객체의 순서를 비교한다.

- 해당 객체가 더 크다면 **양수**를 반환
- 해당 객체가 더 작다면 **음수**를 반환
- 해당객체와 주어진 객체가 같을경우 **0**을 반환
- 해당 객체와 비교할 수 없는 타입의 객체가 전달되면 `ClassCastException`이 발생

### 1-1. 대칭성 보장

- 모든 x, y클래스에 대해서 `sgn(x.compareTo(y) == -sgn(y.compareTo(x))` 이다.
- `x.compareTo(y)`는 `y.compareTo(x)`가 예외를 던질 때에 한해서 예외가 발생해야 한다.

```java
BigDecimal n1 = BigDecimal.valueOf(23134134);
BigDecimal n2 = BigDecimal.valueOf(11231230);

System.out.println(n1.compareTo(n2));
System.out.println(n2.compareTo(n1));
```

### 1-2. 추이성 보장

객체 `x`, `y`, `z`가 있다고 할 때

- x.compareTo(y)가 양수이고 y.compareTo(z)도 양수라면, x.compareTo(z)도 양수여야한다.
- (x > y && y > z 이면 x > z여야 한다.)
- x.compareTo(y) == 0 일 때 sgn(x.compareTo(z)) == sgn(y.compareTo(z))이어야 한다.

### 1-3. 반사성 보장 (equlas 규약과 동일)

- null이 아닌 모든 참조 값(n1)에 대해 아래 n1.compareTo(n1)은 true

```java
BigDecimal n1 = BigDecimal.valueOf(23134134);

System.out.println(n1.compareTo(n1));
```

### 1-4. 동치성 결과가 equals와 같아야 한다.

- x.compareTo(y) == 0 일 때 x.equals(y)어야 한다.

```java
BigDecimal oneZero = new BigDecimal("1.0");
BigDecimal oneZeroZero = new BigDecimal("1.00");
System.out.println(oneZero.compareTo(oneZeroZero)); // Tree, TreeMap
System.out.println(oneZero.equals(oneZeroZero)); // 순서가 없는 콜렉션
```

```bash
0
false
```

같지 않다면 문서화하는 것을 권장한다.

![image](https://user-images.githubusercontent.com/42997924/192532863-d816b84c-8ba9-4cdd-9d29-ea63d3a9e9c3.png)


### 1-5. equals 규약과의 차이점 정리

- `대칭성`, `추이성`, `반사성` 규약은 equals와 규약들이 비슷하다.
- 하지만, 모든 객체에 대해 전역 동치관계를 부여하는 equals와 다르게 compareTo는 타입이 다른 객체를 신경쓰지 않아도 된다.
- 다를 경우 `ClassCastException`을 던지면 된다.

## 2️⃣ 작성 요령

- equals와 비슷하지만, Comparable은 타입을 인수로 받는 `제네릭 인터페이스`라서 compareTo 메서드의 인수 타입은 `컴파일 타임`에 정해진다.
- 입력 인수의 타입을 확인 & 형변환 할 필요가 없다는 의미이다.

- compareTo 메서드는 각 필드의 동치관계를 보는게 아니라 **그 순서를 비교**한다.
- 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출한다.
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 할 경우 Comparator를 쓰면 된다.
- Sample Code

    ```java
    @Getter
    public final class PhoneNumber implements Comparable<PhoneNumber> {
        private short areaCode, prefix, lineNum;
    
    	// 기본 타입 필드가 여럿일 때의 비교자 (91page)
        @Override
        public int compareTo(PhoneNumber pn) {
            int result = Short.compare(areaCode, pn.areaCode);
            if (result == 0)  {
                result = Short.compare(prefix, pn.prefix);
                if (result == 0)
                    result = Short.compare(lineNum, pn.lineNum);
            }
            return result;
        }
    }
    ```

    - 중요도에 따라 우선순위로 비교 가능

### 2-1. 비교자 생성 메서드(comparator construction method)

- 자바 8 부터는 Comparator 인터페이스가 비교자 생성 메서드를 이용해 `메서드 연쇄 방식`으로 비교자를 생성할 수 있게 되었다.
- 방식은 간결하지만 성능은 떨어진다.

```java
public final class PhoneNumber implements Comparable<PhoneNumber> {
    private short areaCode, prefix, lineNum;

    // 비교자 생성 메서드를 활용한 비교자 (92page)
    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.getPrefix())
                    .thenComparingInt(pn -> pn.lineNum);
//
//    @Override
//    public int compareTo(PhoneNumber pn) {
//        return COMPARATOR.compare(this, pn);
//    }
}
```

- 자바에서 타입추론을 제대로 하지 못하기 때문에 명시해 줄 필요가 있다.

### 2-2. 정적 메서드 혹은 비교자 생성 메서드를 활용하자.

객체간 순서를 정한다고 해시코드를 기준으로 정렬하기도하는데 단순히 첫 번째 값이 크면 양수, 같으면 0, 첫 번째 값이 작으면 음수를 반환한다는 것만 생각해서 다음과 같이 작성을해선 안된다.

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object 01, Object 02) {
		return ol.hashCode() - o2.hashCode();
	}
}
```

이런 방식은 얼핏보면 문제 없을 것 같지만 정수 오버플로 혹은 IEEE754 부동소수점 계산 방식에 따른 오류를 낼 수 있다. 게다가 속도가 엄청 빠르지도 않다.

대신, 다음처럼 정적 compare메서드 혹은 비교자 생성 메서드를 활용해보자.

```java
static Comparator<Object hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object 02){
		return Integer.compare(01.hashCode(), o2.hashCode());
	}
}
```

```java
static Comparator<Object> hashCodeOrder = 
		Comparator.comparingInt(Object::hashCode);
```

## 3️⃣ Comparable 구현 클래스를 확장할 때 주의점

> 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지킬 방법이 없다.

객체 지향적 추상화의 이점을 포기해서 `뷰 메서드`를 활용해 우회한 코드는 아래와 같다.

`Point 클래스`

```java
class Point implements Comparable<Point> {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public int compareTo(Point point) {
        int result = Integer.compare(x, point.x);
        if (result == 0) {
            return Integer.compare(y, point.y);
        }
        return result;
    }
}
```

`NamedPoint 클래스`

```java
class NamedPoint implements Comparable<NamedPoint> {
    // 원래 클래스의 인스턴스를 가르키는 필드
    private Point point;

    private String name;

    public NamedPoint(Point point, String name) {
        this.point = point;
        this.name = name;
    }
    // 내부 인스턴스를 반환하는 '뷰' 메서드
    public Point viewPoint() {
        return point;
    }

    @Override
    public int compareTo(NamedPoint namePoint) {
        int result = point.compareTo(namePoint.point);
        if (result == 0) {
            return name.compareTo(namePoint.name);
        }
        return result;
    }
}
```

이렇게 하면 NamedPoint 클래스에 원하는 compareTo 메서드를 새롭게 구현할 수 있게 되어 구체 클래스 Point에서 compareTo 일반 규약을 지킬 수 있게 된다.

Test 코드로 검증하면 아래와 같다.

```java
class PointTest {

    @DisplayName("compareTo의 일반 규약을 지킬 수 있다")
    @Test
    void test() {
        Point point = new Point(1, 3);
        NamedPoint namedPoint = new NamedPoint(new Point(1, 2), "food");

        assertThat(point.compareTo(namedPoint.viewPoint())).isEqualTo(1);
        assertThat(namedPoint.viewPoint().compareTo(point)).isEqualTo(-1);
    }

}
```

## 4️⃣ 정리

- 순서를 고려해야하는 값 클래스는 Comparable인터페이스를 꼭 구현하면 좋다.
- compareTo 메서드에서는 `<` , `>`같은 연산자는 쓰지 않아야 한다.
- 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 활용하자.
