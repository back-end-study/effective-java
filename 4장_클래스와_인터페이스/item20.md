# item20 추상 클래스보다는 인터페이스를 우선하라

## 자바가 제공하는 다중 구현 메커니즘
### 인터페이스 [JLS, 9](https://docs.oracle.com/javase/specs/jls/se19/html/jls-9.html)
- 자바 8부터 디폴트 메서드를 제공할 수 있게 되어 인스턴스 메서드를 구현 형태로 제공할 수 있다.
- 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급된다.

### 추상 클래스 [JLS, 8.1.1.1](https://docs.oracle.com/javase/specs/jls/se19/html/jls-8.html#jls-8.1.1.1)
- 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다.
- 자바는 단일 상속만 지원하니, 새로운 타입을 정의하는 데 커다란 제약을 안게 된다.

## 인터페이스 이점
### 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다.
인터페이스가 요구하는 메서드를 추가하고, 클래스 선언에 implements 구문만 추가하면 끝이다.
반면 기존 클래스 위에 새로운 추상 클래스를 끼워 넣기는 어렵다. 두 클래스가 같은 추상 클래스를 확장하기 원한다면 그 추상 클래스는 계층구조에 커다란 혼란을 일으킨다. 새로 추가된 추상 클래스의 모든 자손이 이를 상속하게 되는 것이다.

### 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다.
믹스인이란 ? [wiki](https://en.wikipedia.org/wiki/Mixin)
- 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다. 대상 타입의 주된 기능에 선택적 기능을 '혼합(mixed in)'한다고 해서 믹스인이라고 부른다.
    - ex) Comparable은 자신을 구현한 클래스의 인스턴스끼리는 순서를 정할 수 있다고 선언하는 믹스인 인터페이스다.
- 클래스는 두 부모를 섬길 수 없고, 클래스 계층구조에는 믹스인을 삽입하기에 합리적인 위치가 없기 때문에 추상 클래스로는 믹스인을 정의할 수 없다.

> 모던 자바 인 액션 p422 (13장 디폴트 메서드 > 디폴트 메서드 활용 패턴 > 동작 다중 상속 > 인터페이스 조합)
> 인터페이스를 조합해서 게임에 필요한 다양한 클래스를 구현할 수 있다.
> 움직일 수 있고, 회전할 수 있으며, 크기를 조절할 수 있는 괴물 클래스 구현
> public class Monster implements Rotatable, Moveable, Resizable { ... }
> 움직일 수 있으며 회전할 수 있지만, 크기는 조절할 수 없는 Sun 클래스 구현
> public class Sun implements Moveable, Rotatable { ... }


### 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.
타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있지만, 현실에는 계층을 엄격히 구분하기 어려운 개념도 있다.
```java
public interface Singer {
	AudioClip sing(Song s);
}

public interface Songwriter {
	Song compose(int chartPosition);
}
// 작곡도 하는 가수
public interface SingerSongwriter extends Singer, Songwriter {
	AudioClip strum();
	void actSensitive();
}
```
같은 구조를 클래스로 만들려면 가능한 조합 전부를 각각의 클래스로 정의한 고도비만 계층구조가 만들어질 것이다. 속성이 n개라면 지원해야 할 조합의 수는 2^n개나 된다. 흔히 조합 폭발이라 부르는 현상이다. 거대한 클래스 계층구조에는 공통 기능을 정의해놓은 타입이 없으니 자칫 매개변수 타입만 다른 메서드들을 수없이 많이 가진 거대한 클래스를 낳을 수 있다. [[stackoverflow] Examples for combinatorial explosion in Java?](https://stackoverflow.com/questions/44325301/examples-for-combinatorial-explosion-in-java)

### 래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상 시키는 안전하고 강력한 수단이 된다.
타입을 추상 클래스로 정의해두면 그 타입에 기능을 추가하는 방법은 상속뿐이다.

## 인터페이스 디폴트 메서드
### 문서화
디폴트 메서드를 제공할 때는 상속하려는 사람을 위한 설명을 @impSpec 자바독 태그를 붙여 문서화해야 한다.

### 제약
equals와 hashCode 같은 메서드는 디폴트 메서드로 제공해서는 안된다.
```java
default boolean equals(Object obj) {  
    return (this == obj);  
}
// java: default method equals in interface XXX overrides a member of java.lang.Object
```
[[stackoverflow] Java8: Why is it forbidden to define a default method for a method from java.lang.Object](https://stackoverflow.com/questions/24016962/java8-why-is-it-forbidden-to-define-a-default-method-for-a-method-from-java-lan)
인스턴스 필드를 가질 수 없고 public이 아닌 정적멤버도 가질 수 없다
- static 메소드 가능 (since. java 8)
- private 메소드 가능 (since. java 9)
  직접 만들지 않은 인터페이스에는 디폴트 메서드를 추가 할 수 없다.

? 해석 규칙
인터페이스에 디폴트 메서드가 추가되었으므로 같은 시그니처를 갖는 디폴트 메서드를 상속받는 상황이 생길 수 있다. 자바 컴파일러는 이러한 충돌을 어떻게 해결할까 ?
1. 클래스가 항상 이긴다. 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다.
2. 1번 규칙 이외의 상황에서는 서브인터페이스가 이긴다. 상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브인터페이스가 이긴다.
```java
public interface A {  
    default void hello() {  
        System.out.println("Hello from A");  
    }  
}  
  
public interface B extends A {  
    default void hello() {  
        System.out.println("Hello from B");  
    }  
}  
public class C implements B, A {  
    public static void main(String[] args) {  
        new C().hello();  // Hello from B
    }  
}
```
3. 여전히 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야한다.
```java
public interface A {  
    default void hello() {  
        System.out.println("Hello from A");  
    }  
}  
  
public interface B {  
    default void hello() {  
        System.out.println("Hello from B");  
    }  
}  
public class C implements B, A {  
    public static void main(String[] args) {  
        new C().hello();  
    }  
}
// java: types B and A are incompatible;
// class C inherits unrelated defaults for hello() from types B and A
```

## 인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공
인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드 몇 개도 함께 제공한다. 그리고 골격 구현 클래스는 나머지 메서드들까지 구현한다. 이렇게 해두면 단순히 골격 구현을 확장하는 것만으로 이 인터페이스를 구현하는 데 필요한 일이 대부분 완료된다. ex) 템플릿 메서드 패턴
관례상 인터페이스 이름이 *Interface*라면 그 골격 구현 클래스의 이름은 Abstract*Interface*로 짓는다.
ex) AbstractSet, AbstractList, AbstractMap
```java
// 골격 구현을 사용해 완성한 List 구현체를 반환하는 정적 팩터리 메서드
// int 배열을 받아 Integer 인스턴스의 리스트 형태로 보여주는 어댑터
static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);

        // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
        // 더 낮은 버전을 사용한다면 <Integer>로 추정하자.
        return new AbstractList<>() {
            @Override public Integer get(int i) {
                return a[i];
            }

            @Override public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;
                return oldVal;
            }

            @Override public int size() {
                return a.length;
            }
        };
    }
```
골격 구현 클래스는 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서는 자유롭다.
구조상 골격 구현을 확장하지 못한다면 인터페이스를 직접 구현해야 한다. 이런 경우라도 디폴트 메서드의 이점을 누릴 수 있다.
골격 구현 클래스를 우회적으로 이용할 수도 있다. 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 것이다.
[Skeletal Implementations in Java Explained](https://10kloc.wordpress.com/2012/12/03/abstract-interfaces-the-mystery-revealed/)

### 골격 구현 작성
- 인터페이스에서 다른 메서드들의 구현에 사용되는 기반(primitives) 메서드들을 선정한다.
- 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공 한다.
- **기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아 있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서드들을 작성해 넣는다.**
- 필요시 public이 아닌 필드와 메서드를 추가해도 된다.

[Map.Entry](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/Map.java#L430)
```java
// 골격 구현 클래스
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {
	//변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
	@Override public V setValue(V value) {
		throw new UnsupportedOperationException();
	}

	//Map.Entry.equals의 일반 규약 구현을 구현한다.
	@Override public boolean equals(Object o) {
		if (o == this)
			return true;
		if (!( o instanceof Map.Entry))
			return false;
		Map.Entry<?, ?> e = (Map.Entry) o;
		return Objects.equals(e.getKey(), getKey()) 
			&& Objects.equals(e.getValue(), getValue());
	}

	//Map.Entry.hashCode의 일반 규약 구현을 구현한다.
	@Override public int hashCode() {
		return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
	}

	@Override public String toString() {
		return getKey() + "=" + getValue();
	}
}
```
[guava](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/AbstractMapEntry.java)
[apache commons collections](https://github.com/apache/commons-collections/blob/master/src/main/java/org/apache/commons/collections4/keyvalue/AbstractMapEntry.java)

### 단순 구현
단순 구현은 골격 구현의 작은 변종으로 [AbstractMap.SimpleEntry](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/AbstractMap.java#L606)가 좋은 예다. 단순 구현도 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만 추상 클래스가 아니란 점이 다르다.
