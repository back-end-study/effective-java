## 아이템86 Serializable을 구현할지는 신중히 결정하라

### Serializable

클래스의 인스턴스를 직렬화 할 수 있게 하려면 클래스 선언에 `implements Serializable` 를 추가하면된다.

```java
// 직렬화 선언 예시
public class User implements Serializable {
    private final String userId;

    public User(String userId) {
        this.userId = userId;
    }
}
```

정말 쉽게 직렬화 적용이 가능하다.

하지만 알고보면 이는 매우 신중하게 선택해서 Serializable 를 구현할지 생각해야한다.

### Serializable을 구현하면 릴리스한 뒤에는 수정하기 어렵다.

클래스가 Serializable을 구현하면 직렬화된 바이트 스트림 인코딩(직렬화형태) 도 하나의 공개 API가 된다.

그래서 이 클래스가 널리 퍼진다면 그 직렬화 형태도 계속해서 지원해야한다.

즉 클래스의 private 과 package-private 인스턴스 필드마저 API로 공개하는 꼴이 된다.

뒤늦게 클래스 내부 구현을 손보면 원래의 직렬화 형태와 달라지게 된다.

한쪽은 구버전 인스턴스를 직렬화하고 다른 쪽은 신버전 클래스로 역질렬화한다면 오류가 발생하게 된다.

직렬화 예제

```java
public class SerialTest {
    void serializable() {
        User user = new User("sawhong");
        byte[] serializedMember;

        try (ByteArrayOutputStream stream = new ByteArrayOutputStream()) {
            try (ObjectOutputStream outPutStream = new ObjectOutputStream(stream)) {
                outPutStream.writeObject(user);
                serializedMember = stream.toByteArray();
            }
        }
        System.out.println(Base64.getEncoder().encodeToString(serializedMember));
    }
}
```
위 코드를 돌리게 되면 `rO0ABXNyAARVc2Vy38q4XzPAJHQCAAFMAAZ1c2VySWR0ABJMamF2YS9sYW5nL1N0cmluZzt4cHQAB3Nhd2hvbmc=` 다음과 같은 코드를 얻게 된다

그리고 이를 역직렬화 하게 된다면

```java
public class SerialTest {
    void deserializable() throws IOException, ClassNotFoundException {
        String serialize = "rO0ABXNyAARVc2Vy51MQda07LpMCAAFMAAZ1c2VySWR0ABJMamF2YS9sYW5nL1N0cmluZzt4cHQAB3Nhd2hvbmc=";
        byte[] decode = Base64.getDecoder().decode(serialize);
        User user;
        try (ByteArrayInputStream bais = new ByteArrayInputStream(decode)) {
            try (ObjectInputStream ois = new ObjectInputStream(bais)) {
                Object objectMember = ois.readObject();
                user = (User) objectMember;
            }
        }

        // sawhong 이 나오게됨
        System.out.println(user.getUserId());
    }
}
```

기존 코드를 수정하기 전까지는 직렬화한 데이터를 역직렬화하면 기존 원하던 데이터를 그대로 받을 수 있다.

하지만! 

user 클래스에 새로운 값을 추가하거나 하면 바로 역직렬화 할 수 없다.


<img width="1243" alt="스크린샷 2023-01-03 오후 8 34 09" src="https://user-images.githubusercontent.com/43979984/210349510-718b1f41-56d7-432b-afb6-17b372653cef.png">


해당 오류가 발생하는 이유는 직렬화된 클래스는 serialVersionUID 를 통해서 고유 식별 번호를 부여 받게된다.

하지만 serialVersionUID 를 클래스내에 static fianl long 필드로 명시하지 않으면 시스템이 런타임에 암호해시 함수(SHA-1) 을 적용해 자동으로 클래스 안에 생성한다.

(우리의 예제가 바로 적용하지 않은 상황)

그래서 나중에 클래스를 수정하게 된다면 serialVersionUID 값도 변하게된다. 그래서 오류가 발생하게 된다.

해결을 위해서는 serialVersionUID 을 갖도록 하면된다. 


### 버그와 보안 구멍이 생길 위험이 높아진다.

객체는 생성자를 사용해 만드는것이 기본이다. 하지만 기본 역직렬화를 사용하면 불변식 깨짐과 허가되지 않는 접근에 쉽게 노출된다.

why? 역직렬화는 숨은 생성자인데 생성자가 만족해야 하는 불변식이나 객체 생성중 객체 내부에 공격자가 접근 할 수 없도록 해야하는것을 잊기 쉽다.

```java
public class User implements Serializable {
    
    private final String userId;
    
    public User(String userId) {
        this.userId = userId;

        if (userId.equals("TEST")) {
            throw new IllegalArgumentException("테스트 계정은 회원가입 할 수 없습니다.");
        }
    }
}
```

userId에 TEST라는 값을 넣게 되면 오류가 발생하는것이 당연하지만, 역직렬화를 통한 객체 생성은 해당 생성자를 무시하고, 생성하게 된다.

### 해당 클래스의 신버전을 릴리스할 때 테스트 할 것이 늘어난다.

직렬화 가능 클래스가 수정되면 신버전 인스턴스를 직렬화 하여 구버전으로 역직렬화 가능한지 테스트 해봐야한다. 

따라서 매번 직렬화 가능 클래스가 수정되면 테스트를 진행해야하기 때문에 매우 귀찮고 힘들게 된다.


### Serializable 구현 여부는 가볍게 결정할 사안이 아니다.

객체를 전송하거나 저장할때 자바 직렬화를 이용하는 프레임 워크용으로 만든 클래스라면 선택의 여지는 없다.

Serializable 을 반드시 구현해야 하는 다른 클래스의 컴포넌트로 쓰일 클래스도 마찬가지다.

하지만 이를 구현하는데 따르는 비용이 적지 않으니, 클래스를 설계할 때마다 득과실을 잘 비교해서 사용해야한다.

### 상속용으로 설계된 클래스는 대부분 Serializable을 구현하면 안되며, 인터페이스도 대부분 Serializable을 확장해서는 안된다.

상속용으로 설계된 클래스나 인터페이스가 Serializable를 확장해서 안되는 이유는, 구현체 들에게 위의 문제점을 모두 주기 때문이다.

그러나 Serializable 을 구현한 클래스만 지원하는 프레임워크를 사용하는 경우에는 위 규칙을 지키지 못하는 경우도 있다

ex) throwable, component


### Serializable을 구현하지 않기로 결정했다면?

Serializable을 구현하지 않을 때는 한 가지만 주의하면 된다.

상속용 클래스에서 직렬화를 지원하지 않지만, 하위 클래스에서 직렬화를 지원하려 한다면 부담이 늘어난다. 

이런 클래스를 역직렬화하려면 상위 클래스는 매개변수가 없는 생성자를 제공해야 한다. 왜냐하면 자식 클래스 인스턴스를 직렬화 할때, 부모 클래스의 기본 생성자가
자동으로 호출 되기 때문이다.


이런생성자가 제공되지 않으면 하위 클래스는 어쩔수 없이 직렬화 프록시 패턴을 사용해야한다.

### 내부 클래스 는 직렬화를 구현하지 말아야 한다.

내부 클래스에는 바깥 인스턴스의 참조와 유효 범위 안의 지역변수 값들을 저장하기 위해 컴파일러가 생성한 필드들이 자동으로 추가된다. 

다시말해  내부 클래스의 기본 직렬화 형태가 분명하지 않음을 말한다. 

해당 필드들이 클래스 정의에 어떻게 추가되는지 정의되지 않은 만큼 내부 클래스에서는 직렬화를 지원하면 안된다.

그러나 정적 멤버 클래스는 Serializable을 구현해도 괜찮다.
