# 아이템 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

## 1. 마커 인터페이스란

아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스

### 1-1. Serializable 마커 인터페이스

```java
public interface Serializable {
}
```

> 249p 책 내용  
> Serializable 인터페이스를 구현한 클래스의 인스턴스는 ObjectOutputStream을 통해 쓸(write) 수 있다고, 즉 직렬화(serialization) 할 수 있다고 알려준다.

```java
@AllArgsConstructor
@ToString
public class OutPutStreamSample implements Serializable {
    @Serial
    private static final long serialVersionUID = -2073279213197039699L;

    private String email;
    private transient String password;
    private int age;
}
```

위 코드와 같이 `Serializable`을 implements 받기만 해도 `ObjectOutputStream` 으로 직렬화 할 수 있게 된다.

```java
public class App {
    public static void main(String[] args) {
        OutPutStreamSample sample = new OutPutStreamSample(
                "youngjun108059@gmail.com",
                "password",
                28);

        byte[] serializedSample;
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {

            oos.writeObject(sample);

            // 직렬화된 Sample 객체
            serializedSample = baos.toByteArray();
            System.out.println("serializedSample = " + Arrays.toString(serializedSample));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

        ByteArrayInputStream bais = new ByteArrayInputStream(serializedSample);
        try (ObjectInputStream ois = new ObjectInputStream(bais)) {

            // 역직렬화된 Sample 객체 읽기
            Object objectSample = ois.readObject();
            OutPutStreamSample sampleDeserialized = (OutPutStreamSample) objectSample;

            System.out.println("sampleDeserialized = " + sampleDeserialized);
        } catch (IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
}
```

![image](https://user-images.githubusercontent.com/42997924/200190235-dd12b858-80c2-4d47-95e1-a2a0042605f9.png)
(↑ 실행 결과)

- 참고로 `transient` 키워드를 넣으면 직렬화되지 않도록 막아준다.

### 1-2. **Custom 마커 인터페이스**

- 객체를 제거할 수 있는지 여부를 나타내는 마커 인터페이스 정의

```java
public interface Deletable {
}
```

- 데이터베이스에서 Entity를 제거하려면 Deletable 마커 인터페이스를 꼭 구현하도록 설계한다

```java
public class Entity implements Deletable {
    private Long id;
    private String data;
}
```

- Dao 클래스에 delete 메서드 내부에서 Deletable 마커 인터페이스가 구현되어있는 Object인지 검사하면 끝!

```java
public class ShapeDao {

    // other dao methods

    public boolean delete(Object object) {
        if (!(object instanceof Deletable)) {
            return false;
        }
        // delete 기능 구현
        return true;
    }
}
```

### 1-3. **Set 마커 인터페이스**

저자는 Set도 마커 인터페이스로 명시하고 있다.

- Collection의 하위 타입에만 적용할 수 있음
- Collection이 정의한 메서드 외에는 새로 추가한 것 없음
- 다만 add, equals, hashCode 등 Collection의 메서드 몇 개의 규약을 조금 수정

### 1-4. **마커 인터페이스의 용도**

- 객체의 특정 부분을 불변식(invariant)으로 규정
    - 불변식이란 : 프로그램 실행 일정 시간동안 항상 참이 되는 조건
    - 타입을 통해 명시함
- 그 타입의 인스턴스는 다른 클래스의 특정 메서드가 처리할 수 있다는 사실을 명시
    - ex) Serializable 인터페이스 → ObjectOutputStream 메서드

## 2. 마커 애너테이션과 차이점

> 249p 책 내용  
> 마커 애너테이션이 등장하면서 마커 인터페이스는 구식이 되었다는 이야기를 들어보았을 것이다. 하지만 사실이 아니다.

**마커 인터페이스**

- 새로 추가하는 메서드 없음
- 단지 타입 정의가 목적

**마커 애너테이션**

- 클래스나 인터페이스 외의 프로그램 요소에 마킹해야 함
- 애너테이션을 적극 활용하는 프레임워크에서 활용

### 2-1. 마커 인터페이스 장점

#### 2-1-1. 컴파일타임 오류 검출 가능

1. 마커 인터페이스를 구현한 클래스의 **인스턴스들을 구분하는 타입**으로 쓸 수 있음
2. 하지만 설계에 따라 런타임에 오류 검출을 하는 경우가 있음
    1. ObjectOutputStream.writeObject 메서드는 당연히 인수로 받은 객체가 Serializable을 구현했을 거라고 가정하고 있음
    2. 그런데 이 메서드는 Serializable이 아닌 Object 객체를 받도록 설계됨
    3. 즉, 직렬화할 수 없는 객체를 넘겨도 런타임에야 문제를 확인할 수 있음

![image](https://user-images.githubusercontent.com/42997924/200191322-8bea1d75-ec91-4fb0-9ff7-bfd0224f175a.png)
↑ ObjectOutputStream.writeObject 설명과 파라미터

#### 2-1-2. 적용 대상을 더 정밀하게 지정 가능

**마커 애너테이션의 한계**

- 적용 대상(`@Target`)을 `ElemetType.TYPE`으로 선언한 애너테이션은 모든 타입(클래스, 인터페이스, 열거타입, 애너테이션)에 달 수 있다.
- 부착할 수 있는 타입을 더 세밀하게 제한하지는 못한다는 뜻이다.

**마커 인터페이스가 필요한 경우**

- 특정 인터페이스를 구현한 클래스에만 마커를 적용하고 싶음
- 마킹하고 싶은 클래스에서만 그 인터페이스를 구현(인터페이스라면 확장)하면 된다.
- 그러면 마킹된 타입은 자동으로 그 인터페이스의 하위 타입임을 보장

위에서 살펴봤던 `Deletable` 마커 인터페이스에 “Shape” 타입의 인스턴스만 삭제해달라는 요구사항이 추가되었다고 하자.

```java
public interface Shape {
    double getArea();
    double getCircumference();
}
```

그럼 아래와 같이 마커 인터페이스를 상속받아서 사용할 수 있다.

```java
public interface DeletableShape extends Shape {
}
```

DeletableShape를 구현하면 “Shape” 타입의 인스턴스임을 명시할 수 있다.

```java
public class Rectangle implements DeletableShape {
    @Override
    public double getArea() {
        return 0;
    }

    @Override
    public double getCircumference() {
        return 0;
    }
}
```

마커 애너테이션 방식은 이렇게 정밀하게 지정할 수 없다.

### 2-2. 마커 애너테이션 장점

- 거대한 애너테이션 시스템의 지원을 받는다는 점
- 애너테이션이 많이 활용되는 프레임워크에서는 마커 애너테이션을 쓰는 쪽이 일관성을 지키는 데 유리할 것 이다.

## 3. 그래서 선택 기준은

![image](https://user-images.githubusercontent.com/42997924/200153227-43bfcff0-cb64-4be6-a1be-378b453a33bc.png)

### 3-1. 마킹 대상이 클래스(or 인터페이스)인가

**클래스 또는 인터페이스가 아닌 경우**

- 모듈, 패키지, 필드, 지역변수 등에 마킹해야 할 때 `애너테이션`을 선택
- 확장(구현)할 수 없기 때문이다

클래스 또는 인터페이스라면 아래 질문을 해보면 된다.

### 3-2. 마킹된 객체를 매개변수로 받는 메서드를 작성할 일이 있나

마킹이 된 객체를 매개변수로 받는 메서드를 작성할 일이 있을까?

- No → **`마커 애너테이션` 선택**
    - 거대한 애너테이션의 시스템 지원도 받고 일관성을 유지할 수 있음
- Yes → `**마커 인터페이스**` 선택
    - 마커 인터페이스를 해당 메서드의 매개변수 타입으로 사용하여 컴파일타임에 오류를 잡아낼 수 있다

## 4. 일반 인터페이스와 차이는

위에서 구현했던 `ShapeDao`를 일반 인터페이스인 `Shape`로 구분하도록 구현해보자

```java
public class ShapeDao {
		
    public boolean delete(Object object) {
        if (!(object instanceof Shape)) { // Shape로 구분
            return false;
        }
        // delete 기능 구현
        return true;
    }
}
```

정확하게 동일하게 기능하고 있지만, 만약 “Animal” 타입도 지울 수 있도록 요구사항이 변경된다면 아래와 같이 수정될 것 이다.

```java
public class ShapeDao {
		
    public boolean delete(Object object) {
        if (!(object instanceof Shape || object instanceof Animal)) {
            return false;
        }
        // delete 기능 구현
        return true;
    }
}
```

제거하려는 Type이 늘어날 때마다 추가하는 것은 좋은 설계가 아니다.

> 기본 인터페이스도 마커로 사용할 수 있지만, 좋지 않은 설계로 가는 경우가 많다

## 5. 정리

마커 인터페이스와 마커 애너테이션은 각자의 쓰임이 있다.

마커 적용 대상이 `ElementType.TYPE`인 마커 애너테이션을 작성하고 있다면, 마커 인터페이스가 낫지는 않을지 고민해보자.

**Reference**

- [이펙티브 자바 3/E](https://product.kyobobook.co.kr/detail/S000001033066)
- [Baeldung marker interfaces](https://www.baeldung.com/java-marker-interfaces)