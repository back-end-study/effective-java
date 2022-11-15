## 스트림 API
스트림 API는 다량의 데이터 처리 작업(순차적이든 병렬적이든)을 돕고자 자바 8에 추가됨.

### 제공하는 추상 개념의 두 가지 핵심
1. 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻함.
2. 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현함.

- 스트림의 원소들은 어디로부터든 올 수 있음.
대표적으로 컬렉션, 배열, 파이프, 정규표현식 패턴 매처(matcher), 난수 생성기, 혹은 다른 스트림이 있다.
- 스트림 안의 데이터 원소들은 객체 참조나 기본 타입 값이다.
기본 타입 값으로 int, long, double 이렇게 세 가지를 지원한다.

### 스트림 파이프라인
소스 스트림에서 시작해 종단 연산으로 끝나며, 그 사이에 하나 이상의 중간 연산이 있을 수 있다.


#### 스트림 파이프 라인 생애 주기 
소스 스트림 -> 1개 이상의 중간 연산 -> 종단 연산
- 스트림  중간 연산들은 모두 한 스트림을 다른 스트림으로 변화시키는데, 변환된 스트림의 원소 타입 값은 기존과 다를 수 있다.
- 종단 연산은 마지막 중간 연산이 내놓은 스트림에 최후 연산을 한다.
- 스트림 파이프라인은 지연 평가된다.

#### 중간 연산
각 중간 연산은 스트림을 어떠한 방식으로 변환한다. 예를 들면, 각 원소에 함수를 적용하거나 특정 조건을 걸어 필터링할 수 있는 식이다.

map	    입력 T 타입 요소를 R 타입 요소로 변환
filter	    조건을 충족하는 요소를 필터링
flatMap	    중첩된 구조를 한 단계 평탄화하고 단일 원소로 변환한 스트림 생성
peek	    스트림 내의 각각의 요소를 대상으로 특정 연산을 수행
skip	    처음 n개의 요소를 제외하는 스트림 생성
limit	    maxSize까지의 요소만 제공하는 스트림 생성
distinct	    스트림 내의 요소의 중복 제거
sorted	    스트림 내 요소를 정렬
 
 
- 마지막 중간 연산의 스트림에 최후의 연산으로 1개 이상의 중간 연산들은 계속 합쳐진 후 종단 연산 시 수행된다.
- 종단 연산이 수행될 때 평가되며, 종단 연산이 수행되지 않으면 아무일도 일어나지 않는다.
- 종단 연산을 꼭 수행해야 무한 스트림을 다룰 수 있다. 종단 연산을 빼먹으면 안 된다.

#### 종단 연산
종단 연산은 중간 연산이 내놓은 스트림에 최후 연산을 가한다. 원소를 정렬해 컬렉션에 담거나 모든 원소를 출력하는 식으로 사용할 수 있다.

forEach	    스트림을 순회
reduce	    연산을 이용해 모든 스트림 요소를 처리하여 하나의 결과로 만듦
collect	    스트림의 연산 결과를 컬렉션 형태로 모아줌

[참고 링크](https://jjingho.tistory.com/94)
#### 지연 평가

: 그때 그때 값을 평가하지 않고, 정말 결과값이 필요한 시점까지 평가를 미루는 것으로 무한 스트림을 다룰 수 있게 해준다.
지연 평가를 하지 않는다면 중간 연산이 끝나지 않는다.

1. 필요할 때만 평가가 되므로 메모리를 효율적으로 사용할 수 있다.

2. 무한 자료구조를 만들 수 있음

3. 런타임 에러를 방지 할 수 있다. ( 컴파일 시 에러를 체킹)

4. 컴파일러 최적화 가능

**스트림 API **= 플루언트 API
- 메서드 연쇄를 지원
- 파이프라인 하나를 구성하는 모든 호출을 연결해 하나의 표현식을 만든다.

- 기본적으로 스트림 파이프라인은 순차적으로 진행이 된다.
- 병렬로 실행하기 위해서는 parallel() 을 호출하면 된다. (그러나 효과는 미미)

- 스트림을 과용하면 프로그램이 읽기 어려워지고 유지보수하기 힘들어지는 경우가 있다.


#### 45-1 사전 하나를 훑어 원소 수가 많은 아나그램 그룹들을 출력
```
public static void main(String[] args) {
    File dectionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    Map<String, Set<String>> groups = new HashMap<>();
    try (Scanner s = new Scanner(dectionary)) {
        while(s.hasNext()) {
            String word = s.next();
            groups.computeIfAbsent(alphabetize(word), 
                                   (unused) -> new TreeSet<>()).add(word);
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
    
    for(Set<String> group : groups.values()) {
        if(group.size() >= minGroupSize) {
            System.out.println(group.size() + ": " + group);
        }
    }
}

private static String alphabetize(String s) {
    char[] a = s.toCharArray();
    Arrays.sort(a);
    return new String(a);
}
```

- 자바 8에서 추가된 computeIfAbsent 메서드를 사용했다
- computeIfAbsent : 맵 안에 키가 있는지 찾은 다음, 있으면 단순히 그 키에 매핑된 값을 반환한다.

#### 45-2 스트림을 과하게 쓴 코드

```
public static void main(String[] args) throws IOException {
    File dectionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    try(Stream<String> words = Files.lines(dectionary.toPath())) {
        words.collect(
            groupingBy(word -> word.chars().sorted()
                       .collect(StringBuilder::new,
                                (sb, c) -> sb.append((char) c),
                                StringBuilder::append).toString()))
            .values().stream()
            .filter(group -> group.size() >= minGroupSize)
            .map(group -> group.size() + ": " + group)
            .forEach(System.out::println);
    }
}
```

#### 45-3 스트림을 적절히 활용하면 깔끔하고 명료해짐

```
public static void main(String[] args) throws IOException {
    File dectionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    try(Stream<String> words = Files.lines(dectionary.toPath())) {
        words.collect(groupingBy(word -> alphabetize(word)))
            .values().stream()
            .filter(group -> group.size() >= minGroupSize)
            .forEach(group -> System.out.println(group.size() + ": " + group));
    }
}
```
- 스트림을 적절하게 사용하면 명료해진다.
- 람다에서는 타입 추론 기능을 사용하기 때문에 주로 타입 이름을 생략한다.
- 따라서 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.
- 연산에 적절한 이름을 지어 주고 도우미 메서드를 적정히 활용하는 것은 일반 반복코드보다 스트림 파이프라인에서 훨씬 크다

    
### 스트림 API를 쓸 때 주의할 사항

- char 값들을 처리할 때는 스트림을 사용하지 말자. char의 스트림은 자바는 지원하지 않고 있기 때문이다.
- 모든 반복문을 스트림으로 바꿀 수 있더라도, 바꾸지 말자.
	- 스트림을 사용할 때는 유지보수와 가독성을 언제나 생각해야 한다. 
	- 기존 코드는 스트림을 사용하도록 리팩터링 하되 새 코드가 더 나아 보일 때만 반영하자.
	
- 스트림은 반복 계산을 함수 객체(람다, 메서드 참조)로, 반복문은 코드블록으로 표현한다.
	- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 그러나 람다에서는 final이거나 사실상 final 변수만 가능하며 지역 변수를 수정할 수 없다.
	- 코드 블록에서는 return문이나 break, continue로 반복문을 제어할 수 있다. 그러나 람다는 불가능하다.
	- 위의 로직 이상의 일들을 수행할 때는 스트림을 사용하지 말자.
	
### 스트림을 사용해야 하는 상황들

- 원소들의 시퀀스를 일관되게 변환한다.
- 원소들의 시퀀스를 필터링한다.
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다.
- 원소들의 시퀀스를 컬렉션에 모은다
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.
	
### 스트림으로 처리하기 어려운 일들
- 한 데이터가 파이프라인의 여러 단계를 통과할 때, 이 데이터의 각 단계에서의 값들에 동시 접근하기 어려운 경우
	- 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이기 때문
	- 원래 값과 새로운 값의 쌍을 저장하는 객체를 사용해 매핑하는 우회 방법 존재	
	- 복잡하기 떄문에 사용하지 않는게 좋다.
	- 그냥 매핑을 거꾸로 수행해라.	
- 원본 스트림을 계속 사용해야 할 때 
	- 스트림은 중간 연산을 지나고 나면 원래의 스트림을 잃는다. 파이프라인 순서를 바꿔서 해결할 수 있을 지 확인할 것.
	
#### 메르센 소수
```
static Stream<BigInteger> primes() {
    return Stream.iterator(TWO, BigInteger::nextProbablePrime);
}
```

- 위의 코드는 무한 스트림을 반환하는 메서드이다.
- 아직 종단 연산이 없기 때문에 실행되지는 않는 스트림이다.

```
public static void main(String[] args) {
    primes().map(p -> TWO.pow(p.intValueExact().subtract(ONE)))
            .filter(mersenne -> mersenne.isProbablePrime(50))
            .limit(20)
            .forEach(System.out::println);
}
```

#### 데카르트 곱
```
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for(Suit suit : Suit.values()) 
        for(Rank rank : Rank.values())
            result.add(new Card(suit, rank));
    return result;
}
```
- 카드는 숫자(rank)와 무늬(suit)를 묶은 불변 값 클래스이거 숫자와 무늬는 열거 타입
- for-each를 통해 스트림을 모르는 개발자라도 쉽게 알아 볼 수 있는 코드이다

```
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
    .flatMap(suit -> Stream.of(Rank.values())
                      .map(rank -> new Card(suit, rank)))
    .collect(toList());
}
```

- Stream을 중첩하여 만든 코드
- 중간 연산으로 flatMap은 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음 그 스트림들을 다시 하나의 스트림으로 합침 -> '평탄화(flattening)'
- 중첩된 람다를 사용했음에 주의


 <<핵심 정리>>

- 스트림은 편하고 깔끔하지만, 과용하면 유지보수와 가독성이 나빠지므로 주의해서 사용하자.
- 반복문을 스트림으로 변경하고 싶다면,
- 기존 코드는 스트림을 사용하도록 리팩토링 하되, 새 코드가 더 나아 보일때만 반영하자.

스트림과 반복문, 어느 것을 사용해야 할 지 고민된다면 둘 다 해보고 결정하자.
