## 한정적 와일드카드를 사용하여 API 유연성을 높혀라

매개 변수화 타입은 불공변이다. 즉 서로 다른 타입 TypeA 와 TypeB가 있을 때 List<TypeA> 는 List<TypeB>의 하위타입도 상위타입도 아니다.

하지만 이러한 불공변 방식보다, 유연한 타입체크가 필요할 수 있다.

아래 예시를 보자

```java
public void pushAll(Iterable<E> src){
        for(E e:src){
            push(e);
        }
}

public static void main(String[]args){
        Stack<Number> stack = new Stack<>();
        Iterable<Integer> integers = new...;
        stack.pushAll(integers); // 컴파일 오류 발생
}
```

Number와 Integer는 상속관계에 있지만, 불공변이기 때문에  컴파일 오류가 발생하게 된다.

### 상한 경계 와일드 카드

```
상한경계 와일드 카드란?

<? extend T>
들어오는 타입의 경우 T의 자식 객체 타입이여 한다.
```

상한경계 와일드 카드를 이용하게 되면, 컴파일이 안되는 문제를 해결 할 수 있다.


```java
//  성공
public void pushAll(Iterable<? Extends E> src) {
        for(E e:src){
            push(e);
        }
}

```

### 하한경계 와일드 카드

그럼 하한경계 와일드 카드를 사용하는 예시도 봐보자.

```
하한경계 와일드 카드란?

<? super T>
들어오는 타입의 경우 T or 부모 객체 타입이여 한다.
```



```java
public void popAll(Collection<E> dst){
        while(!isEmpty()){
            dst.add(pop()); // compile error
        }
}

public static void main(String[]args) {
        Stack<Number> stack = new Stack<>();
        Collection<Object> objects = new...;
        stack.popAll(objects); // 컴파일 오류발생
}

```


```java
//  성공
public void popAll(Collection<? super T> dst){
        while(!isEmpty()){
            dst.add(pop());
        }
}
```


위와 같이 하한경계 와일드 카드를 이용하면 컴파일 오류로부터 벗어나게 된다.

## PECS (producer - extends, consumer-super)

책에서는 extends와 super를 어떤 상횡에서 쓰면 좋읗지 PECS라는 말로 설명해주고 있다.

### PECS란?

매개변수 타입이
T가 생산자(producer)인 경우 상한 경계 와일드 카드(extends) 를 사용하고,

T가 소비자(consumer)이면 하한 경계 와일드 카드(super)를 사용해라

```java
public void pushAll(Iterable<E> src){
        for(E e:src) {
            push(e);
        }
}
```

위 예제에서 src는 push를 하고 있으므로 생산자의 역할을 하고 있다.

```java
public void popAll(Collection<? super E>dst){
        while(!isEmpty()){
            dst.add(pop());
        }
}
```

위 예제에서 E 인스턴스를 소비하고 있기때문에 super를 사용하고 있다.


### 복잡한 불공변의 문제

아래 메서드를 와일드 카드 타입으로 다듬게 되면
```java
public static <E extends Comparable<E>> E max(List<E> list)
```

아래의 형대가 된다.
```java
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```
이 메서드는 E 타입을 List 인자로 받은 후, 받은 List 중 에서 가장 큰 값을 리턴하는 메서드이다.
그리고 return 값은 comparable 이다.

과연이렇게 복잡한 와일드카드도 쓰일곳이 있을까?

=> 쓰일곳이 있다.


당 메서드에 아래의 값을 넣어보게 되면 

```java
List<ScheduledFuture<?>> scheduledFutures = new ArrayList<>();
```

수정전 메서드는 위의 값을 처리하지 못한다.

그 이유는 매개변수 타입의 불공변 때문에 처리하지 못하게 된다.

### 수정 전

![image](https://user-images.githubusercontent.com/43979984/197767795-e85bc100-ae02-41ab-bb7c-1cf02e06471e.png)


### 수정 후

![스크린샷 2022-10-25 오후 8 37 16](https://user-images.githubusercontent.com/43979984/197767536-9ac21deb-396e-439b-87f3-25df62c1a0fc.png)

그래서 비록 수정 후의 방법이 복잡하더라도 의미가 있는 방법이다.

```java
// ScheduledFutuer의 상속형태

interface Comparable<E>
interface Delayed extends Comparable<Delayed>
interface ScheduledFuture<V> extends Delayed, Future<V>
```



### 타입 매개변수 or 와일드 카드

타입매개변수와 와일드 카드는 공통되는 부분이 있어, 둘중 어느것을 사용해도 괜찮을 때가 있다.

아래 예시  메서드에 대해서 간단하게 설명하자면 리스트에 명시한 두인덱스와 아이템들을 swap하는 메서드이다.

```java
public static<E> void swap(List<E> list,int i,int j);

public static void swap(List<?> list,int i,int j);
```

1번의 경우 비한정적 타입매개변수이고,

2번의 경우 비한정적 와일드카드방식이다.


만약 publicAPI 라고 하면 비한정적 와일드 카드타입이 낫다.

그 이유는 신경써야 할 타입 매게변수가 없기 때문이다.


기본규칙으로는 타입 매개변수가 한번만 나오면 와일드 카드로 대체해라

1. 비한정적타입 매개변수라면 비한정적 와일드카드로,
2. 한정적타입매개변수 라면 한정적 와일드카드로
   바꾸면 된다.

하지만 직관적으로 구현한 2번(swap)은 컴파일 되지 않는다.

```java
public static void swap(List<?> list,int i,int j){
        list.set(i,list.set(j,list.get(i)));
}
```

방금 꺼낸 원소를 다시 리스트에 넣을 수 없는 상황이다

이유는 리스트 타입이 List<?> 이기 때문이다, List<?> 에는 null 외에는 어떤 값도 add 할 수 수 없기 때문이다.


이런 경우에는 ***와일드 카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드*** 로 따로 작성하는것이다.

```java
    public static void swap(List<?> list, int i, int j) {
        swapHelper(list, i, j);
    }

    private static <E> void swapHelper(List<E> list, int i, int j) {
        list.set(i, list.set(j, list.get(i)));
    }

```

swapHelper 메서드는 리스트가 List<E>임을 알고 있다. 

즉 이 리스트에서 꺼낸 값은 항상 E이고, E 타입의 값이라면 이 리스트에 넣어도 안전함을 알고 있다.
