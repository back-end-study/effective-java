# item29 이왕이면 제네릭 타입으로 만들라

제네릭 타입과 메서드를 사용하는 일은 일반적으로 쉬운 편이지만, 제네릭 타입을 새로 만드는 일은 조금 더 어렵다.

## Object 기반 스택 - 제네릭이 절실한 강력 후보
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if(size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if(elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

클라이언트가 스택에서 꺼낸 객체를 형변환할때 런타임 오류가 날 위험이 있다.
제네릭 타입으로 변경하면 컴파일타임에 타입검사를 해주어 런타임에 형변환 오류 발생을 방지해주므로 제네릭 타입으로 변경하는게 좋다.
기존 클래스를 제네릭으로 바꾼다고 해도 현재 버전을 사용하는 클라이언트에는 아무런 해가 없다.

## 일반 클래스를 제네릭 클래스로 변경하기
첫 단계는 클래스 선언에 타입 매개변수 추가
Object를 타입 매개변수로 바꾸고 컴파일 해보자.
```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if(size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if(elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

여기서 컴파일 오류가 발생한다. E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다.
```java
Stack.java:8:
        java: generic array creation
        elements = new E[DEFAULT_INITIAL_CAPACITY];
```

해결책은 두 가지가 있다.
첫번째는 제네릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법이다.
```java
elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
// Unchecked cast: 'java.lang.Object[]' to 'E[]' 
```
컴파일러는 오류 대신 경고를 내보낼 것이다.
컴파일러는 이 프로그램이 타입 안전한지 증명할 방법이 없지만 우리는 할 수 있다. 이 비검사 형변환이 프로그램의 타입 안전성을 해치지 않음을 우리 스스로 확인해야 한다.
elements는 private 필드에 저장되고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 전혀 없다. push 메서드를 통해 배열에 저장되는 원소의 타입은 항상 E다. 따라서 이 비검사 형변환은 확실히 안전하다.
@SuppressWarnings 애너테이션으로 해당 경고를 숨긴다.
```java
// 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.  
// 따라서 타입 안전성을 보장하지만, 이 배열의 런타임 타입은 E[]가 아닌 Object[]다!  
@SuppressWarnings("unchecked")  
public Stack() {  
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];  
}
```

두번째 방법은 elements 필드의 타입을 E[] 에서 Object[]로 바꾸는 것이다.
```java
public class Stack<E> {  
    private Object[] elements;  
    ...
  
    public Stack() {  
        elements = new Object[DEFAULT_INITIAL_CAPACITY];  
    }  

	...
	
    public E pop() {  
        ...
        // java: incompatible types: java.lang.Object cannot be converted to E
        E result = elements[--size];  
        ...
    }  
	...
}
```
컴파일 오류가 발생한다.
배열이 반환한 원소를 E타입으로 형변환하면 오류 대신 경고가 뜬다.
```java
// Unchecked cast: 'java.lang.Object' to 'E' 
E result = (E) elements[--size];
```
E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변한이 안전한지 증명할 방법이 없다. 이번에도 우리가 직접 증명하고 경고를 숨길 수 있다.
```java
// push에서 E타입만 허용하므로 이 형변환은 안전하다.
@SuppressWarnings("unchecked")  
E result = (E) elements[--size];
```

첫 번째 방법은 가독성이 더 좋다. 배열의 타입을 E[]로 선언하여 오직 E 타입 인스턴스만 받음을 확실히 어필한다.
첫 번째 방식은 형변환을 배열 생성 단 한 번만 해주면 되지만, 두 번째 방식에서는 배열에서 원소를 읽을 때마다 해줘야 한다. 현업에서는 첫 번째 장식을 더 선호하며 자주 사용한다. 하지만 (E가 Object가 아닌 한) 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염(item32)을 일으킨다. (이 상황에서는 힙 오염을 일으키지 않는다.)

제네릭 Stack을 사용하는 클라이언트에서 명시적으로 형변환을 수행하지 않아도 된다.
```java
public static void main(String[] args) {  
    Stack<String> stack = new Stack<>();  
    for (String arg : args) {  
        stack.push(arg);  
    }  
    while (!stack.isEmpty())  
        System.out.println(stack.pop().toUpperCase());  
}
```


## 제네릭 타입 안에서 리스트를 사용하는게 항상 가능하지도, 꼭 좋은 것도 아니다.
"배열보다는 리스트를 우선하라"는 아이템28과 모순돼 보인다.
자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList 같은 제네릭 타입도 결국은 기본 타입인 배열을 사용해 구현해야 한다. 또한 HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.

제네릭 타입은 타입 매개변수에 기본 타입은 사용할 수 없다. 이는 자바 제네릭 타입 시스템의 근본적인 문제이나, 박싱된 기본 타입을 사용해 우회할 수 있다.

## 한정적 타입 매개변수를 이용해 타입 매개변수에 제약을 둘 수 있다.
ex) `class DelayQueue<E extends Delayed> implements BlockingQueue<E>`
DelayQueue는 Delayed의 하위 타입만 받는다는 뜻이다. 클라이언트는 형변환 없이 Delayed의 메서드를 호출 할 수 있다.

## heap pollution 참고
- [[wiki] heap pollution](https://en.wikipedia.org/wiki/Heap_pollution)
- [[GenericsFAQ] heap pollution](http://www.angelikalanger.com/GenericsFAQ/FAQSections/TechnicalDetails.html#Topic2)
- [JLS, 4.12.2 Variables of Reference Type](https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.12.2)
- [OBJ03-J. Prevent heap pollution](https://wiki.sei.cmu.edu/confluence/display/java/OBJ03-J.+Prevent+heap+pollution)