# 아이템 13: clone 재정의는 주의해서 진행하라


## Cloneable 인터페이스란?

복제해도 되는 클래스임을 명시하는 믹스인 인터페이스이다.


```java
public interface Cloneable{
}
```

Cloneable을 상속 받아 만든 모습

```java
public class Food implements Cloneable {

    @Override
    public Food clone() {
        try {
            return (Food) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

Object 내에 있는 clone

```java
protected native Object clone() throws CloneNotSupportedException;
```

## Clone의 문제점

### 허술한 일반 규약

- x.clone() != x (복사한 객체는 원본 객체와 독립적)
- x.clone().getClass() == x.getClass() (복사한 객체와 원본 객체는 같은 클래스)
- x.clone().equals(x)  (clone() 은 super.clone() 을 통해 객체를 얻어서 반환한다.)

위 조건은 필수가 아닌 `선택 사항` 이다.

이런 강제성이 없는 형태는 생성자 연쇄 패턴과 매우 유사하다

=> 생성자 연쇄 패턴이란?

- 어떤 클래스에서 clone()을 new 키워드를 통해서 생성자를 생성했다
- 컴파일러는 당연 문제가 없기에 잡아 주지 않는다.
- 하지만 해당 클래스를 상속한 클래스에서 super.clone()을 하게 된다면, 하위 클래스의 타입이 아닌 상위 클래스의 타입이 만들어져 잘 작동하지 않는다.



### 해결 방법

1. super.clone()을 호출한다.
2. 그렇게 얻은 객체는 완벽한 복제본
3. 모든 필드가 기본 타입이거나, 불변 객체를 참조한다면 이 객체는 더욱 완벽하다 (그러나 불변클래스를 굳이 clone을 해야하나? 제공안하는게 좋다)


## 가변 객체를 참조하는 문제

가변 객체를 참조하는 순간 끔찍한 일이 벌어진다.

```java
public class Stack {
   private Object[] elements;
   private int size = 0;
   private static final int DEFAULT_INITIAL_CAPACITY = 16;
   //...
}
```

해당 클래스에 clone을 이용하여 그대로 복제를 하게 된다면, 

size의 경우 올바른 값을 갖게되지만, elements 필드의 경우 원본 Stack 인스턴스와 똑같은 배열을 참조하게 된다.

즉 다른 하나를 수정하면 다른 하나도 수정이 된다는 뜻이다.

그래서 원본 객체에 아무런 영향을 끼치지 않도록 보장해야한다.

### 해결 방법

### 1. clone을 재귀적으로 호출해주는 방법이다.

```java
@Override
public Stack clone() {
   try {
      Stack result = (Stack) super.clone();
      result.elements = elements.clone();
      return result;
   } catch (CloneNotSupportedException e) {
      throw new AssertionError();
   }
}
```

### 2. 연결리스트를 재귀적으로 복사

재귀적은 clone() 호출만으로는 부족 할 수 있는데 이때 DeepCopy를 이용한다.
```java
    public Entry deepCopy() {
        return new Entry(key, value,
        next == null ? null : next.deepCopy());
    }
```

하지만 위 방식은 원소 수만큼 연결리스트를 만드므로, 원소 수만큼의 스텍프레임을 소비하여, 스텍오버 플로우가 발생한다

그럴땐 반복자를 이용하여 구한다.

### 3. 연결리스트를 반복적으로 복사

```java
public Entry deepCopy() {
 	Entry result = new Entiry(key, value, next);
 	for (Entry p = result; p.next != null; p p.next)
 		p.next = new Entry(p.next.key, p.next.value, p.next.next);
 	return result;
}
```
deepCopy를 재귀적으로 호출 하는 대신 반복자를 쓰도록 순회하여 StackOverFlow가 발생하지 않도록 한다.


## 복제가 필요한 클래스 생성시 주의사항

1. 상속용 클래스는 cloneable을 구현해서는 안된다.
2. Object의 방식을 모방하여 clone() 메서드를 구현해 protected로 두고 ClassNotSupportedException을 던지도록 선언한다
   1.  cloneable 구현 여부를 하위클래스에 선택하도록 여지를 준다.
3. cloneable을 구현한 thread-safe한 클래스를 작성할 때는 clone 메서드는 적절히 동기화 해줘야한다.
→ Object의 clone 메서드는 동기화를 신경 쓰지 않는다. 그러므로 동기화 해줘야한다.

## 내용 정리

1. Cloneable을 구현 하는 모든 클래스는 clone을 재정의 해야하며, 
2. 이 때 접근 제한자는 public 이여야 하며 반환 타입은 자기 자신 이여야한다.
3. 기본타입 필드와 불변 객체 참조만 갖는 클래스라면 아무 필드도 수정할 필요는 없지만, 일련번호나 고유 ID는 기본타입이고, 불변이겠지만 고유한 값이므로, 수정해줘야한다.

## 다른 방법은 없는가?

복사생성자와 복사 팩터리라는 객체 복사 방식을 사용해보자.

### 복사 생성자란 ?
단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.

```java
public Yum(Yum yum) { ... } ;
```

### 복사 팩터리란?

```java
public static Yum newInstance(Yum yum) { ... } ;
```

위 두방식은

1. 객체 생성 매커니즘을 사용하지 않아도 될 뿐만 아니라
2. 엉성하게 문서화 된 규약에 기대지 않아도 된다
3. 정상적인 final 필드 용버ㅓㅂ과 충돌하지 않는다
4. 불필요한 예외 검사를 던지지도 않는다.
5. 형변환도 하지 않아도 된다.
6. 인터페이스 타입의 인스턴스를 인수로 받을 수 있다. (HashSet 객체를 TreeSet객체로 복제 할 수 있다.)