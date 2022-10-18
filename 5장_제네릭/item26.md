# 아이템26. 로 타입은 사용하지 말라

## 0. 제네릭 용어 정리

제네릭 챕터에서 사용할 대표적인 용어 먼저 정리했다.

```java
public class<T> Sample {
}
```

| 이름                           | 설명                                             | 예제             |
|------------------------------|------------------------------------------------|----------------|
| 제네릭 클래스(or 인터페이스)            | 클래스(인터페이스) 선언에 타입 매개변수(type parameter)가 사용된 경우 | Sample.class   |
| 제네릭 타입(Generic Type)         | 제네릭 클래스(인터페이스)를 의미                             | Sample<T>      |
| 매개변수화 타입(Parameterized Type) | 각각의 제네릭 타입은 parameterizedType을 선언              | Sample<String> |
| 타입 매개 변수(Type parameter)     | 제네릭 선언에 사용되는 매개변수                              | <T>            |
| formalType                   | 제네릭 표현                                         | Sample<E>      |
| actualType                   | 실제 타입                                          | Sample<String> |
| 로타입(raw type)                | 제네릭 타입에서 타입 매개변수를 사용하지 않는 경우                   | Sample         |

로 타입을 알아보기 전에 제네릭부터 살펴보자.

## 1. 제네릭의 장점

### 1-1. 타입 안정성

- 컴파일 시점에서 타입을 체크함으로써 **타입 안정성**을 제공함

![image](https://user-images.githubusercontent.com/42997924/196436583-617e9637-434e-41f9-b187-84e23007c6fd.png)

### 1-2. 컴파일 타임에 타입 선언

- **타입체크와 형변환을 생략**함으로써 코드가 간결해짐
- 컴파일러가 타입선언에 대해 인지하고 있기 때문에, 아무런 경고 없이 컴파일되면 의도대로 동작할 것임을 보장

## 2. 로 타입의 단점

### 2-1. 컴파일 타임에 타입 정보를 알 수 없음

런타임 에러(ex: `ClassCastException`)를 만들기 쉽다.

로 타입은 매개변수를 사용하지 않기 때문에 컴파일러가 자동으로 형변환을 해주면서 런타임에 `ClassCastException`을 던진다.

- 컴파일은 되지만 런타임에서 에러가 발생하여 좋지 않은 코드가 된다

**예제 코드**

![image](https://user-images.githubusercontent.com/42997924/196436674-45614fa7-5b7a-4e51-a080-ea505aa841a5.png)

![image](https://user-images.githubusercontent.com/42997924/196436698-e1effd27-b4cd-4e23-b692-8416947fee3e.png)

> 모든 타입을 받을 수 있도록 자유도를 주지만, 제네릭의 타입 안정성을 제공할 수 없다.
>

## 3. 로 타입은 왜 만들어졌을까

### 3-1. 호환성

기존 코드를 모두 유지하면서 제네릭을 사용하는 새로운 코드도 동작하게 하기 위해

- 호환성 : 로타입 지원 + 제네릭 구현 시 **소거(erasure)** 방식

## 4. 비한정 와일드카드 타입

- 타입 안전을 보장하고 유연한 타입
- 제네릭 타입을 쓰고 싶지만, 실제 타입 매개변수는 모두 받고 싶을 때 사용
- `Set<E>`의 비한정 와일드카드 타입은 `Set<?>`

```dart
 주의사항.
 List<Object>와 List<String>은 공변관계가 아니다.
 Object가 String의 상위 클래스라 해서 매개변수타입에서도 그런 규칙을 따르는 것은 아니다.
 이 관계에서는 서로 다른 타입이라고 생각하면 된다.

```

### 4-1. 로 타입과 차이점

> 와일드카드 타입은 안전하고 로타입은 타입 안전하지 않다.
>
- 로타입 : 아무 원소나 넣을 수 있어 타입 불변식을 훼손하지 쉽다.
- 와일드카드 타입 : Collection<?>에는 어떤 원소도 넣을 수 없음 (`null 제외`)
    - 컴파일 타임에 Error 확인 가능
    - 컬렉션의 타입 불변식을 훼손하지 못하게 막고, 컬렉션에서 꺼낼 수 있는 객체의 타입도 알수 없게 함

## 5. 로 타입 규칙 예외

### 5-1. 클래스 리터럴

자바 명세 자체가 class 리터럴에 매개변수화 타입을 사용하지 못하게 하였다.

- 허용 : `List.class`, `String[].class`, `int.class`
- 불가 : `List<String>.class`, `List<?>.class`

### 5-2. Instanceof 연산자

instanceof 연산자는 비한정 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없다.

**런타임**에는 **제네릭 타입** 정보가 지워진다.

그래서 instanceof에서 리스트 타입인지 알고싶다면, 로타입이든 비한정적 와일드 타입이든 둘중 하나를 사용해라.

> 로타입이든 비한정적 와일드카드 타입이든 instanceof는 똑같이 동작한다. : <?> 코드만 지저분해지므로 로타입 런타임에는 제네릭 타입정보가 지워진다.
>

## 6. 용어 정리

| 한글 용어         | 영문 용어                   | 예                                |
|---------------|-------------------------|----------------------------------|
| 매개변수화 타입      | parameterized type      | List<String>                     |
| 실제 타입 매개변수    | actual type parameter   | String                           |
| 제네릭 타입        | generic type            | List<E>                          |
| 정규 타입 매개변수    | formal type parameter   | E                                |
| 비한정적 와일드카드 타입 | unbounded wildcard type | List<?>                          |
| 로 타입          | raw type                | List                             |
| 한정적 타입 매개변수   | bounded type parameter  | <E extends Number>               |
| 재귀적 타입 한정     | recursive type bound    | <T extends Comparable<T>>        |
| 한정적 와일드카드 타입  | bounded wildcard type   | List<? extends Number>           |
| 제네릭 메서드       | generic method          | static <E> List<E> asList(E[] a) |
| 타입 토큰         | type token              | String.class                     |