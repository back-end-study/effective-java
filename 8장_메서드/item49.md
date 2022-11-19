# 아이템49. 매개변수가 유효한지 검사하라

## 1. 메서드 매개변수의 제약

- 입력 매개변수의 값이 특정 조건을 만족해야 함
    - ex) index의 값은 음수이면 안 됨, 객체 참조는 null이 아니어야 함
- 제약은 반드시 문서화해야 함
- 메서드 body가 시작되기 전에 `검사`해야 함

### 1-1. 매개변수 유효성 검사

> 오류는 가능한 한 빨리 (발생한 곳에서) 잡아야 한다.

- 잘못된 값이 매개변수로 넘어왔을 때 유효성 검사를 통해 예외를 던지자

**유효성 검사가 제대로 안 되면 발생하는 문제**

1. 메서드가 수행되는 중간에 모호한 예외를 던지며 Error 발생
2. 메서드가 잘 수행되지만 잘못된 return 값 반환
3. 메서드 return 까지 잘 됐지만, 다른 객체의 상태를 바꿔 미래의 알 수 없는 시점에 이 메서드와 관련 없는 Error 발생

> 매개변수 유효성 검사에 실패하면 실패 원자성(failure atomicity, 아이템 76)을 어기는 결과를 낳을 수 있다.

### 1-2. 매개변수 값 오류에 대한 예외는 문서화하자

public과 protected 메서드의 매개변수 검사에 대한 Exception은 `@throws` javadoc 태그(아이템 74)를 활용해서 문서화하자.

- 매개변수의 제약을 문서화한다면, 그 제약을 어겼을 때 발생하는 Exception도 적어야 한다.

![image](https://user-images.githubusercontent.com/42997924/202854401-40b4984d-fd05-4503-baa9-17f64094f3b7.png)

(↑ 매개변수 제약과 예외를 문서화한 예제 코드)

- 이 메서드는 m이 null 이면 m.signum을 호출 할 때 NullPointerException을 던진다.
- NPE에 대해 문서화하지 않은 이유는 `BigInteger` 클래스에서 적었기 때문이다.

![image](https://user-images.githubusercontent.com/42997924/202854801-a18e2b1d-6a47-4531-873e-e9c4fa7d6a53.png)

(↑ BigInteger 클래스 설명에 NPE 예외 문서화)

- 클래스 레벨의 주석은 해당 클래스의 모든 public 메서드에 적용된다.

✅ **[참고] 많이 발생하는 표준 예외 (아이템 72)**

- IllegalArgumentException
- IndexOutOfBoundsException
- NullPointerException

## 2. 유효성 검증 방법

먼저 null 검사를 알아보자.

### 2-1. Objects.requireNonNull

null 검사를 위해 자바 7에서 `java.util.Objects.requireNonNull` 메서드가 추가되었다.

```java
// Objects Class method
public static <T> T requireNonNull(T obj, String message) {
    if (obj == null)
        throw new NullPointerException(message);
    return obj;
}
```

아래와 같이 필요한 곳에서 편하게 null 검사를 할 수 있다.

```java
strategy = Objects.requireNonNull(param, "매개변수가 null 입니다.");
```

### 2-2. List, Array 범위 검사

자바 9에서 Objects에 범위 검사 기능이 추가됐다.

```java
// Objects Class method
@ForceInline
public static int checkIndex(int index, int length) {
    return Preconditions.checkIndex(index, length, null);
}

public static int checkFromToIndex(int fromIndex, int toIndex, int length) {
    return Preconditions.checkFromToIndex(fromIndex, toIndex, length, null);
}

public static int checkFromIndexSize(int fromIndex, int size, int length) {
    return Preconditions.checkFromIndexSize(fromIndex, size, length, null);
}
```

- 메서드 매개변수를 보면 알겠지만, 예외 메시지를 지정할 수 없고 List와 Array 전용으로 설계되었다.
- [checkIndex 메서드 호출 샘플 링크](https://www.educative.io/answers/what-is-objectscheckindex-in-java)

### 2-3. assert 검증

공개되지 않은 메서드라면 메서드를 작성한 개발자가 유효한 값만 매개변수로 받을 것을 보증할 때 assert 검증을 활용할 수 있다.

```java
private static void sort(long a[], int offset, int length) {
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    // 생략 (계산 수행)
}
```

**특징**

1. 이 assert(단언문)들은 조건이 무조건 참이 되어야 한다.
2. 실패하면 AssertionError 던진다.
3. 런타임에 아무런 성능 저하가 없다.

다만, 실행 시 VM 옵션에 런타임에 `-ea` 또는 `--enableassertions` 를 넘겨주면 assert를 수행하여 성능에 영향을 준다.

### 2-4. 주의 사항

> 나중에 쓰기 위해 저장하는 매개변수는 더 신경 써서 검사하자.

더 자세한건 예제 코드로 살펴보자.

```java
// Item20에서 다뤘던 정적 팩터리 메서드
static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a); // null 검사

    return new AbstractList<>() {
        // 생략
    };
}
```

위 코드는 아이템20에서 다뤘던 입력받은 int 배열의 List 뷰를 반환하는 정적 팩터리 메서드이다.

- 만약 `Objects.requireNonNull(a)` 검사를 생략했다면 비어있는 배열이 넘어와도 List 인스턴스가 그대로 생성된다.
- 클라이언트가 이 List를 사용하려고 할 때 NPE가 발생하여 디버깅이 어려워진다.

> 추가로, 생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않게 하는 데 꼭 필요하다.

## 3. 유효성 검사가 필요 없는 경우

- 유효성을 검사 비용이 지나치게 큰 경우
- 계산 과정에서 암묵적으로 유효성 검사가 진행되는 경우
    - ex) `Collections.sort(List)`처럼 리스트를 정렬할 때는 정렬 과정에서 모든 객체가 상호 비교된다. 만일 비교할 수 없는 타입의 객체가 있으면 `ClassCastException`이 발생할 것이기 때문에 비교하기에 앞서 모든 원소를 검증하는 것은 불필요한 과정이 된다.

## 4. 정리

***매개변수의 제약을 고민하여 문서화하고, 유효성 검사는 메서드 코드 시작 부분에 추가하자.***

**Reference**

- [이펙티브자바 서적](http://www.yes24.com/Product/Goods/65551284)