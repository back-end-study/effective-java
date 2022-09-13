# 아이템 8: Finalizer와 Cleaner는 피하라
자바에서는 두 가지 객체 소멸자 `finalizer`와 `cleaner` 두가지를 제공한다.
책에서는 이 두가지를 사용하는것을 지양하라고 말하고 있다.

`finalizer의` 경우 `Object`내에 선언되어 있으며, 자바 9버전 부터는 `deprecated`되어 사용하지 말라고한다.
이를 대체하기 위해 `cleaner` 라는게 소개 된다. 하지만 `cleaner` 또한 `finalizer` 보다는 덜 위험하지만 여전히 예측 불가능하고, 느리다.


Q. 작동 방식

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c8159fad-4263-4c9c-aac4-fd64ff5756c5/Untitled.png)

1. 객체가 사용되지 않으면
2. GC가 Queue에 넣고
3. 해당 객체에 선언된 finalize를 실행시킴
4. 실행이 완료 된후 GC를 기다려서 제거를 함


## 수행여부를 보장하지 않는다.
`finalizer`와 `cleaner`는 둘다 언제 실행될지 알 수 없다. 그래서 객체가 더 이상 필요 없어진 시점에서 바로 실행 되지 않을 수 있다.
그래서 DB Lock을 거는 부분과 같으 공유 자원을 사용하는 곳에서는 절대 사용하면 안된다.

`System.gc`와 `System.runFinalization`의 경우에도 둘의 실행 확률을 높혀줄뿐 보장해주진 않는다.
보장 해주겠다고 만든 `System.runFinalizersOnExit`와 `Runtime.runFinalizersOnExit`은 아직까지도 지탄받고 있다.

Q. 둘은 왜 지탄을 받을까?

→ 쓰레드들 간의 교착상태가 발생 할 수 있어 현재 deprecated 상태

```java
@Deprecated
    public static void runFinalizersOnExit(boolean value) {
        Runtime.runFinalizersOnExit(value);
    }
```

[https://howtodoinjava.com/java/basics/why-not-to-use-finalize-method-in-java/](https://howtodoinjava.com/java/basics/why-not-to-use-finalize-method-in-java/)


## 동작 중 발생한 예외를 무시한다.
`finalizer` 는 동작 중 발생한 예외는 무시되며, 처리할 작업이 남아있더라도 그 순간 종료된다. 
잡지 못한 예외로 인해 해당 객체는 마무리가 덜 된 상태로 남을 수 있다.

## 심각한 성능 문제
`AutoCloseable`객체를 만들고, `try-with-resource`를 사용한 것 대비, `finalizer`는 50배, `cleanable`은 5배나 느렸다.


## 보안 문제
`finalizer`는 심각한 보안 이슈가 존재한다. 아래 예제를 보게 되면 `BrokenUser` 클래스는 `User`클래스를 상속받아 만들어져있다.

```java
public class User {
    private final String userId;
    public User(String userId) {
        this.userId = userId;

        if (userId.equals("TEST")) {
            throw new IllegalArgumentException("테스트 계정은 회원가입 할 수 없습니다.");
        }
    }

    public void signUp(String userName) {
        System.out.println(userName + " 님이 성공적으로 회원가입 하셨습니다.");
    }
}
```

```java
public class BrokenUser extends User {

    public BrokenUser(String userId) {
        super(userId);
    }
}

```
상단에 빈객체 `user`를 만들고,  `BrokenUser`객체를 생성할 때 `TEST` 라는 값을 넣어 예외를 발생시킨다.
`BrokenUser`의 상위클래스 `User`의 생성자에 `TEST`라는 값이 들어오면 예외가 발생하도록 로직이 짜여져있기 때문에 catch문으로 빠지게 되어
예외가 발생한다. 그 후 가비지 컬랙터가 돌게되고 빈객체를 제거하면서 선언된 `finalize`가 실행되면서 `User` 클래스의 `signUp` 이 실행된다. 

```java
class BrokenUserTest {

    @Test
    void broken() throws InterruptedException {
        User user = null;
        try {
            user = new BrokenUser("TEST");
        } catch (Exception e) {
            System.out.println("예외 발생");
        }

        System.gc();
        Thread.sleep(3000L);
    }
}
```
책에서는 이를 방지 하기 위해 `final` 을 이용하여 상속을 막거나, `finalize` 메서드에 final 키워드를 상속해서 오버라이딩 하는것을 막으라고 추천한다.

```java
public class User {
    private final String userId;
    public User(String userId) {
        this.userId = userId;

        if (userId.equals("TEST")) {
            throw new IllegalArgumentException("테스트 계정은 회원가입 할 수 없습니다.");
        }
    }

    public void signUp(String userName) {
        System.out.println(userName + " 님이 성공적으로 회원가입 하셨습니다.");
    }

    @Override
    protected final void finalize() throws Throwable {
        this.signUp("비정상적");
    }
}
```



## 추천하는 해결방법은?
책에서는 두가지 방법을 지양하고, 
1. 클라이언트가 명시적으로 `close`를 호출하거나
2. `AutoCloseable` 인터페이스를 구현하고 `try-with-resource`를 사용하라고한다.

## 그럼 둘은 언제 쓰는것이 좋을까?
그래도 `finalizer`와 `cleaner` 둘다 사용하면 좋은 경우가 두가지가 있다.

### 1. 안전망 역할로 자원을 반납하는 경우
즉시 호출되리라는 법은 없지만, 클라이언트가 `close` 메서드를 호출 하지 않을 경우를 대비하여 사용하는 것이 좋다
`FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor` 3가지 메서드는 finalizer 안전망으로 작동하고있다.

```java
// ThreadPoolExecutor
    @Deprecated(since="9")
    protected void finalize() {}
```

### 2. 네이티브 피어 관련 리소스를 정리해야 하는 경우
네이티브 피어는 자바 객체가 아니기 떄문에 GC가 해당 존재에 대해서 알지 못한다. 네이티브 피어가 들고 있는 리소스가 중요하지 않고 성능상 영향이 크지 않다면
`finalize`, `cleaner`를 사용하여 해당 자원을 반납시킬 수 있다.
