
## 기본 타입
![](https://velog.velcdn.com/images/jiyeong/post/78e105ee-7d91-49d2-bbb5-33f3ee51b824/image.png)
![](https://velog.velcdn.com/images/jiyeong/post/09aaa850-02ab-4375-bd8c-64218fb00fc1/image.png)

## 박싱된 기본 타입이란?

각각의 기본 타입에는 대응하는 참조 타입!

- int - Integer
- double - Double
- boolean - Boolean

![](https://velog.velcdn.com/images/jiyeong/post/87e36b25-6140-4378-a0aa-e4b574049420/image.png)
[출처](https://swiftymind.tistory.com/127) </br>
![](https://velog.velcdn.com/images/jiyeong/post/d2562032-eeb1-49d7-aa1c-b880dcb54fd2/image.png)
[출처](https://medium.com/geekculture/effective-java-prefer-primitive-types-to-boxed-types-ea02eb9b8c62)

### 기본 타입과 박싱된 기본 타입의 차이점!
[설명이 자세한 곳](https://it-mesung.tistory.com/66)

1. 기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 값에 더해 **식별성**(identity) 이란 속성을 갖는다.
따라서 박싱된 기본 타입의 두 인스턴스는 값이 같아도 서로 다르다고 식별될 수 있다.

다음과 같이 비교자를 선언한 경우, 같은 값이더라도 실제로는 0이 아니라 1을 출력한다.
< 연산에서는 값을 비교하지만, == 연산에서는 두 객체 참조의 식별성을 검사하기 때문이다.

```
Comparator<Integer> naturalOrder =
        (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```
2. 기본 타입의 값은 언제나 유효하나, 박싱된 기본 타입은 유효하지 않은 값인 null을 가질 수 있다.아래의 프로그램에서는 NullPointerException 이 발생한다.

```
static Integer i;

public static void main(String[] args) {
    if (i == 42)
        System.out.println("믿을 수 없군!");
}
```

기본 타입과 박싱된 기본 타입을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 자동으로 풀리면서, 오류가 발생하는 것이다. 이때 해법은 i를 int로 바꿔주는 것이다.

3. 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다. 박싱된 기본 타입을 사용할 경우에는 박싱과 언박싱이 반복해서 일어나 체감될 정도로 성능이 느리다.

```
public static void main(String[] args) {
	Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
    	sum += i;
    }
    System.out.println(sum);
}
```

### 박싱된 기본 타입을 사용해야 하는 경우

- 컬렉션의 원소, 키, 값으로 사용하는 경우
: 컬렉션은 기본 타입을 담을 수 없으므로 어쩔 수 없이 박싱된 기본 타입을 써야만 한다.

- 매개변수화 타입이나 매개변수화 메서드의 타입 매개변수로는 박싱된 기본 타입을 써야 하는 경우
: 자바 언어가 타입 매개변수로 기본 타입을 지원하지 않기 때문이다.

- 리플렉션(아이템 65)을 통해 메서드를 호출할 때도 박싱된 기본 타입을 사용해야 함!

> 기본 타입과 박싱된 기본 타입 중 하나를 선택해야 한다면 가능하면 **기본 타입**을 사용하라. 왜냐하면 기본 타입은 간단하고 빠르기 때문!
**박싱된 기본 타입을 써야 한다면 주의**를 기울이자. 오토박싱이 박싱된 기본 타입을 사용할 때의 번거로움을 줄여주지만, 그 위험 까지 없애주지는 않는다.두 박싱된 기본 타입을 == 연산자로 비교한다면 식별성 비교가 이뤄지는데, 이는 여러분이 원한 게 아닐 가능성이 크다.
같은 연산에서 기본 타입과 박싱된 기본 타입을 혼용하면 언박싱이 이뤄지며, 언박싱 과정에서 NullPointerException을 던질 수 있다.
마지막으로, 기본 타입을 박싱하는 작업은 필요없는 객체를 생성하는 부작용을 나을 수 있다.

