## 1. 제네릭의 쓰임

1. 컬렉션
    1. ex) Set<E>, Map<K, V> 등
2. 단일원소 컨테이너
    1. ex) ThreadLocal<T>, AtomicReference<T> 등

이런 쓰임에서 매개변수화되는 대상은 (원소가 아닌) 컨테이너 자신이다.

- 용어 정리
    - 매개변수화 타입(parameterized type) → ThreadLocal
    - 제네릭 클래스 → ThreadLocal.class
    - 제네릭 타입 → ThreadLocal
    - 원소 : <T> 

### 1-1. 문제 상황 (다이나믹한 매개변수 타입 개수)

> 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수가 제한된다.

**요구사항**

- 데이터베이스의 행(row)은 임의 개수의 열(column)을 가질 수 있음
- 각 행은 열을 타입 안전해야 함

## 2. 타입 안전 이종 컨테이너 패턴

> Type safe heterogeneous container pattern

- 컨테이너 대신 키를 매개변수화한다.
- 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공한다.

> `값의 타입 = key` 보장

### 2-1. 예시 코드 (Favorites 클래스)

Type 별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 Favorites 클래스

- 각 Type의 Class 객체를 매개변수화한 키 역할로 사용하면 됨

> Q) 이 방식이 동작하는 이유?  
> A) class의 클래스가 제네릭이기 때문


```java
public class Favorites {
    public <T> void putFavorite(Class<T> type, T instance);
    public <T> T getFavorite(Class<T> type);
}
```

- 키가 매개변수화 되었다 → `Class<T> type`
- 클라이언트는 즐겨찾기를 저장하거나 얻어올 때 Class 객체를 알려주면 된다.

```java
public class Client {
    public static void main(String[] args) {
        Favorites f = new Favorites();

        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 0xcafebabe);
        f.putFavorite(Class.class, Favorites.class);

        String favoriteString = f.getFavorite(String.class);
        int favoriteInteger = f.getFavorite(Integer.class);
        Class<?> favoriteClass = f.getFavorite(Class.class);

        System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName());

    }
}
```

- 즐겨 찾는 String, Integer, Class 인스턴스를 put/get
	- String.class의 타입 → Class<String>
	- Integer.class의 타입 → Class<Integer>
- Favorites 인스턴스는 Type Safe
- Map과 비슷하지만 여러 type의 원소를 담을 수 있다

Favoites API 구현은 아래와 같다.

```java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

- `Map<Class<?>, Object>`는 비한정적 와일드카드 타입이라 아무것도 넣을 수 없다고 생각할 수 있지만 아니다
- 와일드카드 type이 중첩(nested)되었기 때문에 Map이 아니라 Key가 와일드카드 type이다
- 이는 모든 키가 서로 다른 매개변수화 타입일 수 있다는 뜻

### 2-2. putFavorite 메서드 구현

![image](https://user-images.githubusercontent.com/42997924/199226280-c6252327-4d50-40d5-aa1e-3ed6c8702634.png)

(↑ 타입 안정 이종 컨테이너의 활용과 컴파일 에러)

책에는 아래와 같이 명시되어있다.

> 200p 이 맵은 키와 값 사이의 타입 관계를 보증하지 않는다는 말이다.  
> 즉, 모든 값이 키로 명시한 타입임을 보증하지 않는다.  
> 사실 자바의 타입 시스템에서는 이 관계를 명시할 방법이 없다.  

하지만, 위 코드 샘플과 컴파일 에러를 확인해보면 키와 값 사이의 타입 관계를 보증하는 것처럼 보인다.

Java 버전이 올라가면서 추가된 것인지, `보증하지 않는다`는 의미가 다른 것인지 잘 모르겠다.

<img width="634" alt="image" src="https://user-images.githubusercontent.com/42997924/199226805-db591f4e-2aae-44e6-b354-0feba1a0b845.png">

### 2-3. **getFavorite 메서드** 구현

```java
public <T> T getFavorite(Class<T> type) {
	return type.cast(favorites.get(type));
}
```

- Class 객체에 해당하는 값을 favorites Map에서 꺼낸다
- 컴파일타임 타입을 맞춰주기 위해 `java.lang.Class`의 cast 메서드를 사용한다
- cast 메서드로 동적 형변환 가능

<img width="543" alt="image" src="https://user-images.githubusercontent.com/42997924/199228255-15ff8221-ae3b-41fd-97a2-045131263ecb.png">

java.lang.Class → cast 메서드

- 유효성검사를 하여 ClassCastException을 던진다
- 하지만 위에서 작성한 getFavorite를 호출하는 클라이언트 코드가 컴파일된다면 예외가 발생하는 경우는 없다 (favorites Map의 Key-Value 타입은 항상 일치 함)

### 2-4. 첫 번째 제약 : 로 타입으로 타입 안전성 깨기

- 클라이언트가 악의적으로 Class 객체를 로 타입으로 넘기면 타입 안전성이 깨진다.

<img width="649" alt="image" src="https://user-images.githubusercontent.com/42997924/199229211-5cb71b23-19f7-447e-88e3-bcbb537d16fa.png">

(↑ 로 타입으로 Class 객체를 넘긴 클라이언트 코드)

<img width="626" alt="image" src="https://user-images.githubusercontent.com/42997924/199229156-e3cbd984-089c-4bca-a34b-4fb36a733165.png">

(↑ ClassCastException 발생)

- 클라이언트 코드에 비검사 경고를 띄우고 런타임 에러가 발생한다.
- HashSet, HashMap에서도 같은 제약이 있다.
- 예를 들면, 아래 코드도 컴파일된다. (꺼내기 전까지 에러 발생하지 않음)

    ```java
    HashSet<Integer> set = new HashSet<>();
    ((HashSet)set).add("문자열 넣어버리기~");
    ```


### 2-5. 런타임 타입 안전성이라도 확보하기

Favorites 객체가 타입 불변식을 어기는 일이 없도록 보장하려면 putFavorite 메서드를 다음과 같이 수정하면 된다.

<img width="1143" alt="image" src="https://user-images.githubusercontent.com/42997924/199230557-cff0a5f2-d36a-4d51-b672-8fdf305cfdc2.png">

(↑ 동적 형변환으로 런타임 타입 안전성 확보)

instance의 타입이 type으로 명시한 타입과 같은지 체크해주면 put할 때 ClassCastException을 발생시킨다.

**Collections는 어떻게 방어했을까**

- `java.util.Collections`에 checkedSet, checkedList 메서드는 컴파일타임 타입이 같음을 보장한다.
- `public static <E> List<E> checkedList(List<E> list, Class<E> type)`
- 위 메서드에서 실체화도 해주기 때문에 클라이언트에서 로 타입을 넣을 수 없게 한다.

> 📌  실체화란  
> 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인 함

### 2-6. 두 번째 제약 : 실체화 불가 타입에는 사용 불가

- List<String>는 저장 불가능 ↔  String, String[] 저장 가능
- List<String>.Class 는 문법 오류 발생
    - List<Integer>와 List<String>은 List.class라는 같은 Class 객체를 공유 함
    - 따라서, 둘 다 똑같은 타입의 객체 참조를 반환하면 안 됨
- 아직 완벽히 만족스러운 우회 방법은 없다

### 2-7. 슈퍼 타입 토큰으로 우회하기

- 슈퍼 타입 토큰으로 우회하는 방법을 ParameterizedTypeReference 클래스를 Spring 프레임워크에서 구현해두었다.
- 상속과 Refelction을 기발하게 조합해서 원래 사용할 수 없었던 클래스 리터럴을 타입 토큰으로 사용하는 것과 같은 효과를 낼 수 있다.

잘 정리해둔 발표 자료가 있으니 읽어보면 좋을 것 같다!

- [https://sungminhong.github.io/spring/superTypeToken/](https://sungminhong.github.io/spring/superTypeToken/)

## 3. 한정적 타입 토큰

- Favorite가 사용하는 타입 토큰은 비한정적이다 (`getFavorite`와 `putFavorite`는 어떤 Class 객체든지 받아 들인다)
- 허용하는 타입을 제한하고 싶다면, 한정적 타입 토큰을 활용하면 된다!

**한정적 타입 토큰이란?**

- 단순히 한정적 타입 매개변수(아이템 29)나 한정적 와일드카드(아이템 31)를 사용하여 표현 가능한 타입을 제한하는 타입 토큰
- ex) 애너테이션 API는 한정적 타입 토큰을 적극적으로 사용한다.

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
```

- `annotationType` 인수는 애너테이션 타입을 뜻하는 한정적 타입 토큰
    - 토큰으로 명시한 타입의 애너테이션이 대상 요소에 달려 있다면 그 애너테이션을 반환하고, 없다면 null을 반환
    - 즉, 애너테이션 요소는 그 키가 애너테이션 타입인, `타입 안전 이종 컨테이너`이다
- 만약  `Class<?>`타입의 객체를 한정적 타입 토큰을 받는 메서드에 넘기려면 어떻게 해야 할까?
    - 객체를  `Class<? extends Annotation>`으로 형변환 하는 것은 비검사 이므로 컴파일 시 경고가 발생한다. (아이템 27)
    - Class 클래스는 이런 형변환을 안전하고 동적으로 수행하게 해주는 인스턴스 메서드를 제공하는데,  `asSubclass`  메서드이다.
        - `asSubclass` : 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환한다.
            - (형변환된다는 것은 이 클래스가 인수로 명시한 클래스의 하위 클래스라는 뜻)

```java
// asSubclass를 사용해 한정적 타입 토큰을 안전하게 형변환한다.
// 타입을 알 수 없는 애너테이션을 asSubclass 메서드를 사용하여 런타임을 읽어내는 예시
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
	Class<?> annotationType = null; // 비한정적 타입 토큰
    try {
    	annotationType = Class.forName(annotationTypeName);
    } catch (Exception ex) {
    	throw new IllegalArgumentException(ex);
    }
    return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```

## 4. 핵심 정리

> 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다.  
> 그러나 컨테이너 자체가 아닌 `키를 타입 매개변수로 바꾸면` 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다.  
>   
> 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이러한 Class 객체를 타입 토큰이라 부른다.  
> 또한 직접 구현한 키 타입도 사용 가능하다.

### 궁금증) 제네릭의 타입 추론은 어떻게 동작할까?

`컴파일타임 타입 정보 ↔  런타임 타입 정보`

서로 다른 타입 정보를 알아내기 위해 메서드들은 타입 토큰(type token)을 주고 받는다.

타입 토큰은 class 리터럴로 구성되어있다.

**class 리터럴?**

- [https://offbyone.tistory.com/98](https://offbyone.tistory.com/98)

### 궁금증) java.lang.Class vs java.lang.Object

- [https://kimg0130.tistory.com/entry/javalangClass](https://kimg0130.tistory.com/entry/javalangClass)
- [https://velog.io/@dudwls0505/java.lang-의-Class-클래스](https://velog.io/@dudwls0505/java.lang-%EC%9D%98-Class-%ED%81%B4%EB%9E%98%EC%8A%A4)
- [https://bangu4.tistory.com/191](https://bangu4.tistory.com/191)
- [**https://stackoverflow.com/questions/24718980/java-lang-class-and-java-lang-object**](https://stackoverflow.com/questions/24718980/java-lang-class-and-java-lang-object)
