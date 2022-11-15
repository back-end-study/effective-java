## 아이템 46 스트림에서는 부작용 없는 함수를 사용하라

스트림 패러다임의 핵심은 계산을 일련의 변환(transformation)으로 재구성 하는 부분이다.

이때 각 변환 단계는 가능한 이전 단계의 결과를 받아 처리하는 순수함수여야한다.

그러므로 스트림 연산에 건네는 함수 객체는 모두 부작용이 없어야한다.

### BAD 예시
```java
   Map<String, Long> freq = new HashMap<>();
   
    try(Stream<String> words = new Scanner(file).tokens())
           words.forEach(word -> {
               freq.merge(word.toLowerCase(), 1L, Long::sum);
           });
   }
```
위 예시는 스트림 코드를 가장한 반복적 코드이다.

이 코드의 모든 작업이 종단 연산인 forEach 에서 일어나는데, 이때 외부 상태를 수정하는 람다를 실행하면서 문제가 생긴다.

forEach 연산은 스트림 계산 결과를 보고할때만 사용해야하는데, 그 이상의 일(연산)을 하고 있다.

### GOOD 예시
```java
	Map<String, Long> freq;

	try (Stream<String> words = new Scanner(file).tokens()) {
		freq = words.collect(groupingBy(String::toLowerCase, counting()));
    }
```

+ 수집기(Collector)를 사용
+ forEach 연산은 종단 연산중 기능이 가장적고 대놓고 반복적이라 병렬화할 수 없다.
+ forEach 연산은 스트림 계산결과를 보고할때만 사용하고, 계산할때는 사용하지 말자
  + 가끔은 스트림 계산 결과를 기존 컬랙션에 추가하는 용도로는 쓸 수 있다.

## Collector(수집기)
+ `java.util.stream.Collectors` 클래스는 스트림의 원소들을 객체 하나에 취합하는 여러 메서드를 제공해준다.
+ 이를 이용하면 스트림 원소를 손쉽게 컬랜션으로 모을 수 있다 (toList(), toMap(), toCollection())


```java
// Collector 를 이용한 예시

List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10) 
    .collect(toList());
```

### 1. toMap(keyMapper, valueMapper)
가장 간단한 맵 수집기 이고, 스트림 원소를 키에 맵핑하는 함수와, 값에 맵핑하는 인수 두개를 받는다.

```java
private static final Map<String, Operation> stringToEnum = 
    Stream.of(values()).collect(
        toMap(Obejct::toString, e->e));
```
+ toMap 의 경우 스트림의 각 원소가 고유한 키에 맵핑 되어 있을때 적합하다
+ 만약 스트림 원소가 다수의 같은 키를 사용하면 파이프 라인은 `illegalStateException` 을 던진다.
+ 다수의 키를 사용하여 충돌이 발생하는 경우 toMap 은 추가 기능을 제공한다
  + toMap 에 키맵퍼, 값맵퍼, 병합 함수까지 제공한다 `BinaryOperator` 를 이용하여 충돌 처리 방법을 지정 할 수 있다.


toMap 은 네번째 인수로 mapFactory 를 받는다. 이 인수를 통해서 enumMap, treeMap 과 같이 특정 맵 구현체를 직접 지정할 수 있다.

```java
toMap(keyMapper, valueMapper, (oldVal, newVaD -> newVal, () -> new TreeMap<>())
```


### 2. groupingBy
groupingBy 는 카테고리 별로 모아 그륩화 하고 Map에 저장을 한다.

```java
// groupingBy 예시
words.collect(groupingBy(word -> alphabetsize(word)))

// 다운 스트림(SET)을 명시 
Map<String, Set<String>> collect = stringList.stream()
        .collect(groupingBy(String::toLowerCase, toSet()));

// 다운 스트림(Collection)을 명시 
Map<String, Set<String>> collect = stringList.stream()
        .collect(groupingBy(String::toLowerCase, toCollection(LinkedHashSet::new)));

// 다운 스트림(Counting)을 명시
Map<String, Long> collect = stringList.stream()
        .collect(groupingBy(String::toLowerCase, counting()));
```

+ 만약 리스트가 아닌 다른 값을 갖도록 하고 싶으면, 분류함수와 함께 다운 스트림 수집기도 명시해야한다.
+ groupingBy 는 다운스트림 수집기에 더해 맵 팩터리도 지정 할 수 있게 해준다. 
  + 참고로 이 메서드는 점층적 인수목록 패턴에 어긋난다.


![image](https://user-images.githubusercontent.com/43979984/201910117-31c42d7b-422a-4241-94fa-07f032b77194.png)

![image](https://user-images.githubusercontent.com/43979984/201910253-d41689c4-b402-4dec-a399-67c838055fa1.png)

(mapFactory가 downStream 매개 변수보다 앞에 놓인다)

```java
public static <T, K, D, A, M extends Map<K, D>>
    Collector<T, ?, M> groupingBy(Function<? super T, ? extends K> classifier,
                                  Supplier<M> mapFactory,
                                  Collector<? super T, A, D> downstream)
```

```java
TreeMap<String, Long> collect = stringList.stream()
        .collect(groupingBy(String::toLowerCase, TreeMap::new, counting()));
```

### 3. partitioningBy
+ grouponBy와 비슷한느낌을 주는 함수
+ 분류 함수 자리에 (predicate)를 받고 키가 Boolean 인 맵을 반환한다
+ predicate 에 더해 다운 스트림 수집기 까지 입력 받는 버전도 다중 정의 되어 있다


### 4. joining
+ joining 메서드는 charSequence 인스턴스의 스트림에만 적용 할 수 있다.
+ 인수가 없는 joining은 단순히 원소들을 연결 하는 수집기를 반환한다.
+ 한편 인수 하나짜리 joining은 CharSequence 타입의 구분문자(delimiter)를 매개변수로 받는다.

## 핵심정리
1. 스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다.
   1. 스트림 뿐만 아니라, 스트림 객체에 건네지는 모든 함수 객체가 문제가 있으면 안된다.
2. 종단 연산중 forEach는 스트림이 수행한 계산결과를 보고할 때만 사용한다
   1. 계산자체에는 이용하지 않는다
3. 스트림을 올바로 사용하려면 수집기에 대해서 알아야한다
   1. toList, toSet, toMap, groupingBy, joining