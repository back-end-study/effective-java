## 아이템 47 반환 타입으로는 스트림보다 컬렉션이 낫다

원소 시퀸스를 반환하는 메서드는 수없이 많다. 
자바 7까지는 Collections, Set, List 같은 컬렉션 인터페이스, 혹은 Iterable이나 배열을 썼다.
그런데 Java8에 스트림이 등장하면서 이 선택이 복잡한 일이 되어버렸다. 
스트림이 반복(iteration)을 지원하지 않는다.

Stream 인터페이스는 Iterable 인터페이스의 추상 메서드를 모두 포
함하고 정의한 방식대로 동작하지만 Iterable 인터페이스를 확장하지 않았다.
그래서 for-each로 스트림을 반복할 수 없다.


```java
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
    // 프로세스를 처리한다.
}
```
아래의 코드는 컴파일 오류가 난다.
```java
for (ProcessHandle ph : (Iterable<ProcessHandle>)
    ProcessHandle.allProcess()::iterator) {
    // 프로세스를 처리한다.
}
```
메소드 참조를 적절히 형변환해주면 위와 같다.<br>
작동은 하지만 실전에 쓰기에는 너무 난잡하고 직관성이 떨어진다.<br>
어댑터 메서드를 사용하면 스트림을 iterate로 변경할 수 있다.<br>
다음 코드와 같이 쉽게 만들어낼 수 있다.

```java
//Stream<E>를 Iterable<E>로 중개해주는 어댑터
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
return stream::iterator;
}

for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
    // 프로세스를 처리한다.
}
```
반대로 API가 Iterable을 받아서 스트림을 반환하는 어댑터도 구현이 가능하다.
```java
//Iterabel<E>를 Stream<E>로 중개해주는 어댑터
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
return StreamSupport.stream(iterable.spliterator(), false);
}
```

### Collection VS Stream
#### Collection

- Collection 인터페이스는 Iterable 인터페이스의 하위 타입이고 
stream 메서드도 제공하니 일반적으로는 Collection을 반환하는편이 좋다.
- 하지만 Collection의 각 원소는 메모리에 올라가므로, 시퀀스의 크기가 크다면 고민해보는 것이 좋다.
#### Stream
- 컬렉션의 contains와 size를 시퀀스의 내용을 확정하기 전 까지 구할 수 없는 경우(i.e. 실제 반복을 돌려보기 전 까지는 무엇이 얼마나 들어갈지 예측이 불가능한 경우)에는 Stream을 반환하는 것이 좋다.

### 정리
Stream이나 Iterable을 리턴하는 API에는 Stream <-> Iterable로 변환할 수 있도록 어댑터 메서드가 필요하다.<br>
어댑터는 클라이언트 코드를 어수선하게 만들고 더 느리다.<br>
원소 시퀀스를 반환하는 메서드를 작성할 때는 Stream, Iterator를 모두 지원할 수 있게 작성해라<br>
원소의 개수가 많다면, 멱집합의 예처럼 전용 컬렉션을 리턴하는 방법도 고민해라.<br>
