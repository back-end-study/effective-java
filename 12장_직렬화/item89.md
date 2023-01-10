# 아이템89. 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라

## 1. 싱글턴과 직렬화

> **싱글턴 패턴의 클래스가 직렬화를 구현하면 싱글턴을 보장하지 못한다**

아래에 작성한 싱글턴 클래스는 객체를 메모리에 한번만 올리는 것을 보장한다.

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	
	private Elvis() {
	}

	public static Elvis getINSTANCE() {
		return INSTANCE;
	}
}
```

하지만 싱글턴 클래스가 직렬화 가능한 클래스가 되기 위해 `Serializable` 인터페이스를 구현하면 싱글턴을 보장하지 못한다.

이유는 직렬화를 통해 초기화해둔 인스턴스가 아닌 다른 인스턴스가 반환되기 때문이다.

`readObject`를 제공하는 방식도  해결할 수 없다. 어떻게 직렬화할 수 있을까?

## 2. 싱글턴 직렬화하기 : readResolve

`readResolve` 기능을 이용해 `readObject`가 만들어낸 인스턴스를 대체할 수 있다.

1. 역직렬화 된 후 새로 생성된 객체를 인수로 readResolve 메서드가 호출
2. 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환
3. 새로 생성된 객체는 참조되는 곳이 없어 가비지 컬렉션

```java
private Object readResolve() {
	return INSTANCE;
}
```

- 이렇게 되면 직렬화 형태가 어떠한 데이터도 가질 필요가 없다 (인스턴스 개수 통제 목적)
- 모든 인스턴스 필드는 `transient` 한정자를 붙여주자
- 만약 그렇지 않으면 역직렬화(Deserialization) 과정에서 역직렬화된 인스턴스를 가져올 수 있다. → **싱글턴이 깨지게 된다.**

## 3. 어떻게 공격당할까

싱글턴이 `transient`가 아닌 참조 필드를 가지고 있다면, 그 필드의 내용은 readResolve 메서드가 실행되기 전에 역직렬화된다.

1. readResolve 메서드와 인스턴스 필드 하나를 포함한 도둑 클래스를 만든다.
2. 도둑 클래스의 인스턴스 필드는 직렬화된 싱글턴을 참조하는 역할을 한다.
3. **직렬화된 스트림에서 싱글턴의 비휘발성 필드를 도둑의 인스턴스 필드로 교체**한다.
4. 싱글턴이 도둑을 포함하므로 역직렬화시 도둑 클래스의 readResolve가 먼저 호출된다.
5. 도둑 클래스의 인스턴스 필드에는 역직렬화 도중의 싱글턴의 참조가 담겨있게 된다.
6. 도둑 클래스의 readResolve 메서드는 인스턴스 필드가 참조한 값을 정적 필드로 복사한다.
7. 싱글턴은 도둑이 숨긴 transient가 아닌 필드의 원래 타입에 맞는 값을 반환한다.
8. 이 과정을 생략하면 직렬화 시스템이 도둑의 참조를 이 필드에 저장하려 할 때 VM에서 `ClassCastException` 을 발생시킨다.

```java
// transient가 아닌 참조 필드를 가지는 싱글턴
public class Elvis implements Serializable {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { }
	// transient 아님
  private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};

  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }

  private Object readResolve() {
    return INSTANCE;
  }
}
```

```java
// 싱글턴의 비휘발성 인스턴스 필드를 훔쳐러는 도둑 클래스
public class ElvisStealer implements Serializable {
  private static final long serialVersionUID = 0;
  static Elvis impersonator;
  private Elvis payload;

  private Object readResolve() {
    // resolve되기 전의 Elvis 인스턴스의 참조를 저장
    impersonator = payload;

    // favoriteSongs 필드에 맞는 타입의 객체를 반환
    return new String[] {"A Fool Such as I"};
  }
}
```

임시로 만든 스트림을 이용해 2개의 싱글턴 인스턴스를 만들어낸다.

```java
// 직렬화의 약점을 이용해 싱글턴 객체를 2개 생성한다.
public class ElvisImpersonator {
  private static final byte[] serializedForm = new byte[]{
      (byte) 0xac, (byte) 0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
      // 코드 생략
  };

  private static Object deserialize(byte[] sf) {
    try (ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(sf)) {
      try (ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream)) {
        return objectInputStream.readObject();
      } catch (IOException | ClassNotFoundException e) {
        throw new IllegalArgumentException(e);
      }
    }
  }       

  public static void main(String[] args) {
    // ElvisStealer.impersonator 를 초기화한 다음,
    // 진짜 Elvis(즉, Elvis.INSTANCE)를 반환
    Elvis elvis = (Elvis) deserialize(serializedForm);
    Elvis impersonator = ElvisStealer.impersonator;
    // GC 처리 됐어야하는 Elvis를 갖고 있음
    elvis.printFavorites(); // [Hound Dog, Heartbreak Hotel]
    impersonator.printFavorites(); // [A Fool Such as I]
  }
}
```

서로 다른 2개의 Elvis 인스턴스가 생성된다.

```
[Hound Dog, HeartBreak Hotel]
[A Fool Such as I]
```

## 4. 해결 방법 : enum

- `enum` 을 사용하면 모든 것이 해결된다!
- `enum` 클래스는 자바가 선언한 상수 외에 다른 객체가 없음을 보장해준다.
    - 단, `AccessibleObject.setAccessible` 메서드와 같은 리플렉션을 사용했을 때는 예외다.

```java
public enum Elvis {
	INSTANCE;
	private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

	public void printFavorites() {
		System.out.println(Arrays.toString(favoriteSongs));
	}
}
```

### 4-1. readResolve 메서드가 필요할 때

- 인스턴스 통제를 위해 `readResolve` 메서드를 사용하는 것이 중요할 때도 있다.
- 직렬화 가능 인스턴스 통제 클래스를 작성해야 할 때, 컴파일 타임에는 어떤 인스턴스들이 있는지 모를 수 있는 상황이라면 열거 타입으로 표현하는 것이 불가능하기 때문에 `readResolve` 메서드를 사용할 수 밖에 없다.

## 5. readResolve 메서드의 접근성

- `final` 클래스라면 private 접근 제어자를 사용해야 한다.
- `final` 이 아닌 경우 클래스일 때 주의할 점
    - `private` 선언시 하위 클래스에서 사용할 수 없다.
    - `package-private` 으로 선언시 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다.
    - `protected`, `public` 은 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다.
    - `protected`, `public` 이면서 하위클래스에서 재정의 하지 않으면, 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 `ClassCastException` 을 일으킬 수 있다.

## 6. 정리

- 불변식을 지키기 위해 인스턴스를 통제해야 한다면, enum 클래스가 가장 좋다.
- enum 클래스를 사용할 수 없는 상황에서 직렬화와 인스턴스 통제가 필요하다면, `readResolve` 메서드를 작성하고, 모든 필드를 `transient` 한정자로 선언해야 한다.
- 이 때, `readResolve` 메서드의 접근성은 매우 중요하다.
    - final 클래스에선 private 접근 제어자를 사용해야 한다.
    - final 이 아닌 클래스에서는 하위 클래스를 고려해서 작성할 필요가 있다.