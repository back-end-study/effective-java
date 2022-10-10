# Item18 상속보다는 컴포지션을 사용하라

책에서 말하는 상속은 클래스가 다른 클래스를 확장(extends)하는 상속

- 상속은 캡슐화를 깨뜨린다 
- 상위 클래스와의 강결합 문제 
- 설계자가 확장을 충분히 고려하고 문서화해도 상위클래스에 맞게 하위클래스도 수정해야만 하는경우가 생긴다 


### addAll메서드가 add메서드를 사용해서 생기는문제(p.115)
 
```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet() {}

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount()); // 3이아닌 6출력
    }
}
```

이 문제를 해결하기위한 방법

1. 상위클래스의 addAll메서드를 호출하지않고 **addAll메서드를 다른방식으로 재정의**하기  
=> 오류를 유발하거나 성능저하시킬 여지가있으며, 하위클래스에 존재하지않는 private필드를 써야하는상황이라면 불가능한 방법

```java
    @Override
    public boolean addAll(Collection<? extends E> c) {
        for (E e : c) {
            add(e);
        }
        
   	return ..
        
//        addCount += c.size();
//        return super.addAll(c);
    }
```


2. 메서드를 재정의하지않고 새로운 메서드를 추가하는방식   
=> 상위클래스에서 추가된 새로운메서드가 시그니처와 반환타입에 따라 컴파일되지않거나, 새 메서드를 재정의한 것이나 다름없는 상황 발생 


3. **확장하는대신 컴포지션(책에서 권장하는방법)**
- 기존클래스가 새로운 클래스의 구성요소로 쓰인다는 의미(Composition)
- 기존클래스를 확장하는대신 새로운 클래스를 만들고 private필드로 기존클래스의 인스턴스를 참조



```java
public class ForwardingSet<E> implements Set<E>{
    private final Set<E> s;

    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    // 오버라이딩
     ...
}

public class InstrumentedSet<E> extends ForwardingSet<E>{
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        Set<Instant> times = new InstrumentedSet<>(new TreeSet<>()); // 다른 Set인스턴스를 감싸고있다, 기능을 추가한다
    }
}
```

InstrumentedSet(래퍼 클래스)
- Set의 인스턴스를 인수로 받는 생성자 제공
- 임의의 Set형태에 기능을 덧씌움(데코레이터 패턴)
- 래퍼객체가 자기자신의 참조를 내부객체에 전달하는경우 위임(컴포지션 + 전달)이라고도 부른다
- 특정 메서드를 추가해 조건,기능을 추가할수도있다.

ForwardingSet(전달 클래스)
- 재사용할수있는 전달클래스

### 래퍼클래스, 콜백프레임워크 (Self 문제)

> 래퍼클래스는 단점이 거의 없다.  
래퍼 클래스가 콜백 프레임워크와 어울리지않는다는 점만 주의하면된다.  
콜백 프레임워크에서는 자기자신의 참조를 다른객체에 넘겨서 다음콜백때 사용하도록한다.  

https://stackoverflow.com/questions/28254116/wrapper-classes-are-not-suited-for-callback-frameworks  
책에서 소개한 스택오버플로의 답변의 예제코드


```java
public interface SomethingWithCallback {

    void doSomething();
    void call();
    
    // 내부 클래스
    class WrappedObject implements SomethingWithCallback {

        private final SomeService service;

        WrappedObject(SomeService service) {
            this.service = service;
        }

        @Override
        public void doSomething() {
            service.performAsync(this); // 문제점  
        }

        @Override
        public void call() {
            System.out.println("WrappedObject callback!");
        }
    }
    
    // 래퍼클래스
    class Wrapper implements SomethingWithCallback {

        private final WrappedObject wrappedObject;

        Wrapper(WrappedObject wrappedObject) {
            this.wrappedObject = wrappedObject;
        }

        @Override
        public void doSomething() {
            wrappedObject.doSomething();
        }

        void doSomethingElse() {
            System.out.println("We can do everything the wrapped object can, and more!");
        }

        @Override
        public void call() {
            System.out.println("Wrapper callback!");
        }
    }


    final class SomeService {

        void performAsync(SomethingWithCallback callback) {
            new Thread(() -> {
                perform();
                callback.call(); // call호출 
            }).start();
        }

        void perform() {
            System.out.println("Service is being performed.");
        }
    }

    public static void main(String[] args) {
        SomeService service = new SomeService(); // call()을 호출하는 클래스 
        WrappedObject wrappedObject = new WrappedObject(service); // 래핑될 클래스
        Wrapper wrapper = new Wrapper(wrappedObject); // 래퍼클래스
        wrapper.doSomething(); // print:  Service is being performed WrappedObject callback! 
    }
}
```

래퍼(Wrapper)가 내부객체(WrappedObject)를 감싸고 dosomething을 호출하는데  
래퍼(Wrapper)의 call()이 아닌 내부객체(WrappedObject)의 call()이 호출됩니다.   
=> 내부객체는 자신을 감싸고있는 래퍼클래스의 존재를알지 못하기때문이다.


### 상속은 is-a 관계일때만 쓰여야한다

- 자바의 Stack, Properties가 is-a 관계가 아닌데 상속을 잘못사용하여 설계한 예시
- 상속은 상위 클래스의 결함까지도 그대로 승계한다.
- 사실, is-a관계이더라도 문제가 생길수있다.





## 궁금한점

책에나온 인터페이스-전달클래스-래퍼클래스 구조가 Set의 하위클래스들에 유연하게 기능도 추가할수있다는 장점으로 이해했습니다.

제가 헛다리를 짚고있는걸수도 있는데 위의구조를 보다가 인터페이스의 default메서드가 생각이났습니다.
만약 새로 설계해야한다면 위의방법대신 Set과같은 역할을하는 **인터페이스 자체에 default메서드를 놓고 하위클래스에서 임의로 오버라이딩**하는 방식을 사용하게된다면
중간에 전달클래스가 필요없게되어 고려해볼만한 방식인것같은데 이방식에 대해서는 어떻게 생각하시나요??

아래에 이미 만들어져있는 Set과 TreeSet 비슷하게 흉내냈는데, Set이아닌 새로운 설계를 해야한다면 이런구조도 어떨지 궁금합니다

```java
public interface CustomSet<E> {
    void clear();
    void contains();
    void isEmpty();
    void size();
    /**
     * 이 인터페이스를 구현한 하위클래스에서 재정의할수있음
     */
    default boolean add(E e) {
        return false;
    }
}


public class CustomTreeSet<E> implements CustomSet<E>{
    
    // TreeSet 에있는 임의의 필드값들
    private transient NavigableMap<E,Object> m;
    private static final Object PRESENT = new Object();

    // 책에있는 부분(기능)
    private int addCount = 0;

    @Override
    public void clear() {//구현..}
    
    @Override
    public void contains() {//구현..}
    
    @Override
    public void isEmpty() {//구현..}
    
    @Override
    public void size() {//구현..}
    
    // 기능추가를 위한 재정의
    @Override
    public boolean add(E e) {
        addCount++;
        return m.put(e, PRESENT) == null;
    }
}

// 래핑하지않고 직접 사용할수있다. 
    public void start() {
        CustomSet<Instant> times = new CustomTreeSet<>();
        CustomSet<E> s = new CustomHashSet<>(10);
    }
```

이렇게되면 인터페이스의 구현체별로 전달클래스없이 특정기능을 추가할수있게되어 중간에 클래스가 하나 없어도 되는방식도 고려해볼수있나? 라는생각이 들었습니다
