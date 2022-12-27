# Item 64 객체는 인터페이스를 사용해 참조하라


```java
// 좋은 예 (인터페이스를 타입으로 사용)
Set<Son> sonSet = new LinkedHashSet<>();

// 나쁜 예 (클래스를 타입으로 사용)
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```


### 왜 좋은예인가?

LinkedHashSet이 아닌 HashSet으로 교체해야한다면 새 클래스의 생성자(혹은 정적 팩터리)를 호출해주기만 하면된다
```java
Set<Son> sonSet = new HashSet<>();
```

### 주의사항

인터페이스의 일반 규약이외의 특별한 기능을 제공하는 클래스라면 그 점도 고려해야한다

ex) LinkedHashSet으로 순서라는 개념이 필요한 상황에서 HashSet으로 변경하는경우 문제가 발생할수있다.


### 클래스 타입으로 참조해야하는경우

- String,BigInteger 같은 적합한 인터페이스가 없는경우
- 클래스 기반으로 작성된 프레임워크가 제공하는객체들 (OutputStream같은..)
- 인터페이스에는 없는 특별한 메서드를 제공하는 클래스들인경우 (PriorityQueue 클래스)


유연한 구조를위해 가장 덜 구체적인 클래스를 타입으로 사용하자는 얘기
