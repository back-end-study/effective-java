# item43 람다보다는 메서드 참조를 사용하라

[메서드 참조(method reference)](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)


## 람다 사용

### 람다 사용 예제
```java
map.merge(key, 1, (count, incr) -> count + incr);
```
이 코드는 임의의 키와 Integer 값의 매핑을 관리하는 프로그램의 일부다. 값이 키의 인스턴스 개수로 해석된다면, 이 프로그램은 [멀티셋(multiset)](https://www.baeldung.com/guava-multiset) 을 구현한 게 된다.
> 멀티셋 : 포함된 각 요소의 수를 추적하여 동일한 요소를 허용

키가 맵 안에 없다면 키와 숫자 1을 매핑하고, 이미 있다면 기존 매핑 값을 증가시킨다.

### Map.merge
merge는 자바 8에서 Map에 추가된 메서드로 키, 값, 함수를 인수로 받는다.
- 주어진 키가 맵 안에 아직 없다면 주어진 {키, 값} 쌍을 그대로 저장
- 키가 이미 있다면 함수를 현재 값과 주어진 값에 적용한 다음, 그 결과로 현재 값을 덮어 쓴다. 즉, 맵에 {키, 함수의 결과} 쌍을 저장한다.
```java
default V merge(K key, V value,  
        BiFunction<? super V, ? super V, ? extends V> remappingFunction) {  
    Objects.requireNonNull(remappingFunction);  
    Objects.requireNonNull(value);  
    V oldValue = get(key);  
    V newValue = (oldValue == null) ? value :  
               remappingFunction.apply(oldValue, value);  
    if (newValue == null) {  
        remove(key);  
    } else {  
        put(key, newValue);  
    }  
    return newValue;  
}
```

## 메서드 참조 사용

### Integer.sum
자바 8이 되면서 Integer 클래스(와 모든 기본타입의 박싱 타입)는 이 람다와 기능이 같은 정적 메서드 sum을 제공하기 시작했다.
```java
public static int sum(int a, int b) {  
    return a + b;  
}
```

### 메서드 참조 사용 예제
람다 대신 이 메서드의 참조를 전달하면 똑같은 결과를 더 보기 좋게 얻을 수 있다.
```java
map.merge(key, 1, Integer::sum);
```


### 메서드 참조 이점
매개변수 수가 늘어날수록 메서드 참조로 제거할 수 있는 코드양도 늘어난다.
람다로 구현했을 때 너무 길거나 복잡하다면 메서드 참조가 좋은 대안이 되어 준다.
람다로 작성할 코드를 새로운 메서드에 담은 다음, 람다 대신 그 메서드 참조를 사용할 수 있다.
메서드 참조에는 기능을 잘 드러내는 이름을 지어줄 수 있고 친절한 설명을 문서로 남길 수도 있다.

### 람다가 더 나은 경우
어떤 람다에서는 매개변수의 이름 자체가 프로그래머에게 좋은 가이드가 되기도 한다. 이런 람다는 길이는 더 길지만 메서드 참조보다 읽기 쉽고 유지보수도 쉬울 수 있다.
때론 람다가 메서드 참조보다 간결할 때가 있다. 주로 메서드와 람다가 같은 클래스에 있을 때 그렇다.
GoshThisClassNameIsHumongous 클래스 안에 다음 코드가 있다고 해보자.
```java
service.execute(GoshThisClassNameIsHumongous::action);
```
이를 람다로 대체하면 다음 처럼 된다.
```java
service.execute(() -> action());
```
메서드 참조 쪽은 더 짧지도, 더 명확하지도 않다. 따라서 람다 쪽이 낫다.
java.util.function 패키지가 제공하는 제네릭 정적 팩터리 메서드인 `Function.identity()`를 사용하기보다는 똑같은 기능의 람다(x -> x)를 직접 사용하는 편이 코드도 짧고 명확하다.

### 메서드 참조의 유형
|메서드 참조 유형|예|같은 기능을 하는 람다|
|---|---|---|
|정적|`Integer::parseInt`|`str -> Integer.parseInt(str)`|
|한정적(인스턴스)|`Instant.now()::isAfter`|`Instant then = Instant.now;  t -> then.isAfter(t)`|
|비한정적(인스턴스)|`String::toLowerCase`|`str -> str.toLowerCase()`|
|클래스 생성자|`TreeMap<K,V>::new`|`() -> new TreeMap<K,V>()`|
|배열 생성자|`int[]::new`|`len -> new int[len]`|

#### (수신 객체를 특정하는) 한정적 인스턴스 메서드 참조
근본적으로 정적 참조와 비슷하다. 즉, 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 똑같다.
```java
createBicyclesList().stream()
	.sorted((a, b) -> bikeFrameSizeComparator.compare(a, b));
createBicyclesList().stream() 
	.sorted(bikeFrameSizeComparator::compare);
```
#### 비한정적 인스턴스 메서드 참조
함수 객체를 적용하는 시점에 수신 객체를 알려준다. 이를 위해 **수신 객체 전달용 매개변수가 매개변수 목록의 첫 번째로 추가**되며, 그 뒤로는 참조되는 메서드 선언에 정의된 매개변수들이 뒤따른다. 비한정적 참조는 주로 스트림 파이프라인에서의 매핑과 필터 함수에 쓰인다.(item45)
```java
numbers.stream()
	.sorted((a, b) -> a.compareTo(b)); 
numbers.stream()
	.sorted(Integer::compareTo);
```

### 제네릭 함수 타입 구현에서 메서드 참조 사용
람다로는 불가능하나 메서드 참조로는 가능한 유일한 예는 바로 제네릭 함수 타입(generic function type) 구현이다. [자바 명세 예제 9.9-2](https://docs.oracle.com/javase/specs/jls/se17/html/jls-9.html#jls-9.9)
> 함수형 인터페이스의 추상 메서드가 제네릭일 수 있듯이 함수 타입도 제네릭일 수 있다. 다음 인터페이스 계층구조를 생각해보자.
```java
interface G1 {
    <E extends Exception> Object m() throws E;
}
interface G2 {
    <F extends Exception> String m() throws Exception;
}
interface G extends G1, G2 {}
```
> 이때 함수형 인터페이스 G를 함수 타입으로 표현하면 다음과 같다.
```java
<F extends Exception> () -> String throws F
```
> 이처럼 함수형 인터페이스를 위한 제네릭 함수 타입은 메서드 참조 표현식으로는 구현할 수 있지만, 람다식으로는 불가능하다. 제네릭 람다식이라는 문법이 존재하지 않기 때문이다.

```java
public class Test {  
    public static void main(String[] args) {  
        G lambda = () -> "aa";
    }
}
// java: incompatible types: invalid functional descriptor for lambda expression
//   method <F>()java.lang.String in interface G is generic
```
```java
public class Test {  
    public static void main(String[] args) {   
        G methodReference = Test::m;
    }  
  
    private static String m() {  
        return "aa";  
    }  
}
```


[Baeldung - method references](https://www.baeldung.com/java-method-references)

[Java8 람다 관련 스펙 정리](https://homoefficio.github.io/2017/02/19/Java8-%EB%9E%8C%EB%8B%A4-%EA%B4%80%EB%A0%A8-%EC%8A%A4%ED%8E%99-%EC%A0%95%EB%A6%AC/)  
[Function Types, JLS 9.9](https://docs.oracle.com/javase/specs/jls/se8/html/jls-9.html#jls-9.9)  
[Type of a Lambda Expression, JLS 15.27.3](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.27.3)  
[Type of a Method Reference, JLS 15.13.2](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.13.2)  
[Generalized Target-Type Inference – Java 8](https://www.baeldung.com/java-generalized-target-type-inference#generalized)  