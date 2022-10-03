# item 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

상속을 염두에 두지 않은 외부 클래스(*주: 프로그래머의 통제권 밖에 있어 변경 시점을 알 수 없는 클래스*)를 상속할 경우 `여러 문제`가 발생할 수 있다.

상속을 고려한 설계와 문서화는 정확히 무얼 뜻할까?

## 1. 재정의 메서드 문서화

상속이 목적인 클래스를 설계한다면 기능 확장 및 재정의할 수 있는 메서드들이 내부적으로 어떻게 이용되는지 문서로 남겨야 한다. `자기사용`

> 상속할 클래스의 메서드 재정의가 어떤 사이드 이펙트를 줄 수 있는지 명확히 가이드할 필요가 있다

- 재정의할 수 있는 메서드 : public or protected 메서드 중 final이 아닌 모든 메서드

**호출할 수 있는 모든 상황**

- API로 공개된 메서드로부터 호출되는 ***재정의 메서드*** 의 사이드 이펙트를 적어야 한다
- 백그라운드 스레드, 정적 초기화 과정 중 호출이 일어날 수 있는 상황 등

### 1-1. @implSpec

API 문서의 끝부분을 보면 `implementation Requirements`라는 항목이 존재하는데, 이 메서드의 내부 동작 방식을 설명하는 절이다.  메서드 주석에 `@impleSpec` 태그를 붙여주면 JavaDoc 도구가 생성해준다.

`java.util.AbstractCollection` 의 `remove` 메서드 JavaDoc 예를 보면 아래와 같다.

![image](https://user-images.githubusercontent.com/42997924/193596835-c6055c3e-df2f-4b03-9b0e-ac52f51f616b.png)

이 설명에 따르면 iterator 메소드를 재정의하면 remove 메서드의 동작에 영향을 준다는 것을 알 수 있다.

### 1-2. @implSepc 활성화

```shell
-tag "implSpec:a:Implementation Requirements:"
```

- 자바 8에 도입되어 자바 11버전까지 JavaDoc 에서도 기본값이 아닌 선택사항으로 남겨져 있는데, 이를 활성화 하기 위해서는 명령줄 매개변수로 위에 작성한 명령어를 지정해주면 된다.

> 자바 17버전에서 기본값으로 활성화되는지 찾아보려 했지만 찾지 못했다. 아직 선택사항인 것 같다.


### 1-3. {@inheritDoc}

추가적으로 `{@inheritDoc}` 키워드가 궁금하여 찾아보니 재정의한 상위 메서드의 내용을 그대로 복사해서 보여주는 태그이다.

![image](https://user-images.githubusercontent.com/42997924/193596869-dd90bacd-cc35-4be7-9c25-f9d04e1cece5.png)

자세한 JavaDoc 태그는 wikipedia 문서에 잘 정리되어있다. 예제도 같이 있으니 궁금하면 읽어봐도 좋을 것 같다.

[https://en.wikipedia.org/wiki/Javadoc](https://en.wikipedia.org/wiki/Javadoc)

![image](https://user-images.githubusercontent.com/42997924/193596888-f19cae6a-cc0e-4531-8099-5ddd529215d7.png)

잘 작성된 문서(JavaDoc)을 보면 어떤 예외가 발생하는지, 다른 메서드에 어떤 영향을 받는지 등 재정의하여 개발할 때 주의할 부분을 알 수 있다.

### 1-4. 한계점

> 좋은 API문서란 `어떻게` 가 아니라 `무엇`을 하는지를 설명해야 한다.

- 재정의 메서드 문서화는 위 격언 내용과 대치된다.
- 상속이 캡슐화를 깨트리기 때문인데, 클래스를 안전하게 상속할 수 있도록 하기 위해서는 내부 구현 방식을 설명해야만 하기 때문이다.

## 2. 상속 설계 : Hook 선별

클래스 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별해서 protected 메서드 형태로 공개해야 할 수도 있다.

`java.util.AbstractList`에 `removeRange` 메서드를 예로 살펴보자.

![image](https://user-images.githubusercontent.com/42997924/193596908-cc221e7d-7ef8-469c-88f2-ff54396dfae0.png)

- clear 연산이 이 메서드를 호출한다고 직접 명시
- 리스트 구현의 내부 구조를 활용하도록 이 메서드를 재정의하면, clear 연산 성능을 크게 개선할 수 있음

### 2-1. protected Hook 메서드 목적

List 구현체를 사용하는 개발자는 removeRange 메서드에 관심이 없다.(재정의되어있지도 않다)

그런데 어째서 이 메서드도 문서가 제공되는 것일까?

이는, List 구현체 사용 개발자가 아니라 AbstractList의 하위 클래스를 만들 개발자가 이 메서드를 사용하는 clear 메서드를 고성능으로 만들기 쉽게 하기 위해서인데, clear 메서드를 보면 내부적으로 removeRange()를 호출한다.

![image](https://user-images.githubusercontent.com/42997924/193596944-d4d0a265-397d-41f8-bc91-bda2e535c167.png)

만약 removeRange() 메서드가 없는 상태에서 clear 메서드를 구현해야 한다면 성능이 느리거나(제거할 원소 수의 제곱) 부분 리스트의 메커니즘을 새로 구현해야 했을 것이다.

### 2-2. 어떤 메서드를 protected로 노출해야 할까

- 명확한 기준은 없음
- 잘 예측해보고, 실제 하위 클래스를 만들어 테스트해야 함
- protected 메서드는 내부 구현이기 때문에 최소한으로 유지
- 그렇다고 너무 적게 노출하면 상속의 이점을 잃을 수 있음

> 적당히 노출해라..?

![image](https://user-images.githubusercontent.com/42997924/193596978-ca0ad219-ac09-4391-b6c5-7260ed9eb87b.png)

### 2-3. 상속용 클래스 Test 방법

직접 하위 클래스를 만들어보는 것이 `유일`하다.

- 하위 클래스를 여러 개(3개 정도) 만들 때까지 전혀 쓰지 않는 protected 멤버는 private일 가능성이 크다
- 배포 전에 반드시 하위 클래스를 만들어 검증해야 한다.

> 상속을 위한 설명과 일반적인 API 설명에 대한 구분이 명확하지 않은데 `@implSpec`을 파싱해서 표시해주는 tool이 없을까..

## 3. 제약 : 생성자에서 재정의 가능 메서드 호출 금지

**상속용 클래스의 생성자가 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.**

다음 코드를 보자.

```java
public class Super {
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
        System.out.println("Super override method!");
    }
}
```

```java
public class Sub extends Super {
    private final Instant instant;
    public Sub() {
        instant = Instant.now();
    }

    @Override
    public void overrideMe() {
        System.out.println("Sub overrideMe Method!! " + instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

![image](https://user-images.githubusercontent.com/42997924/193597028-3eece394-1d9f-4414-8708-f2d7abb59f32.png)
(↑ 실행 결과)

### 3-1. null이 출력되는 이유

- 상위 클래스 생성자가 하위 클래스 생성자보다 항상 먼저 호출되기 때문
- Sub 생성자에서 명시적으로 `super()` 를 호출하지 않더라도, 상위 클래스의 생성자에 기본 생성자 `super()`가 호출된다.
- 이때 실제 구현체는 Sub 클래스이기 때문에 재정의된 `overrideMe()` 메서드가 실행됨
- 이 시점에는 아직 `instant`가 초기화되지 않아 `null` 출력
- **final 필드의 상태가 두 가지이다(정상이라면 하나)**
- 로직에 따라 NullPointException이 발생할 수 있다.

***private, final, static 메서드는 재정의가 불가능하기에 생성자에서 호출해도 된다.***

## 4. Cloneable, Serializable 인터페이스

둘 다 상속할 수 있게 설계한다면, 상속받아 확장하려는 개발자에게 엄청난 부담을 준다.

하위 클래스에서 원한다면 구현하도록 하는 방법이 있다.

### 4-1. clone과 readObject 메서드

- clone과 readObject 메서드는 생성자와 비슷한 효과를 낸다 (**새로운 객체를 만든다**)
- 따라서 상속용 클래스에서 구현할 때 제약도 생성자와 비슷하다.
    - **재정의 가능 메서드를 호출하면 안 된다.**

**Cloneable - clone 메서드 (아이템 13)**

- 하위 클래스의 clone 메서드가 복제본의 상태를 수정하기 전에 재정의한 메서드 호출
- 공부한 것처럼 배열을 제외하면 비추천

**Serializable - readObject 메서드 (아이템 86)**

- 하위 클래스의 상태가 역직렬화되기 전에 재정의한 메서드부터 호출
- `readResolve`나 `writeReplace` 메서드를 protected로 선언해야 함(상속 때문에 내부 구현을 클래스 API로 공개해야 하는 예)
- 아이템 86에서 자세히 살펴볼 예정

## 5. **상속용이 아닌 일반 구체 클래스는 상속 금지**

- 일반 구체 클래스
    - final도 아니고 상속용으로 설계되거나 문서도 없는 클래스
- 일반 구체 클래스를 상속하게 되면?
    - 클래스가 변경될 때마다 하위 클래스에서 문제가 발생하는 경우가 많음

### 5-1. 상속을 금지하는 방법

- 하나의 클래스를 final로 선언
- 모든 생성자를 private (or package-private)으로 선언하고 public 정적 팩터리를 만든다
    - 정적 팩터리 방법은 내부에서 다양한 하위 클래스를 만들어 쓸 수 있는 유연성을 제공(아이템 17)

## 6. 상속을 꼭 해야 한다면

1. 구현 상속보다는 인터페이스 상속이 더 낫다.
2. 상속을 허용한다면 재정의 가능 메서드를 사용하지 않게 만들고 문서로 남기자.
3. 클래스 동작을 유지하고 재정의 가능 메서드 사용 코드를 제거하자.
    - 재정의 가능 메서드는 private 도우미 메서드로 옮기기
    - 이 도우미 메서드를 호출하도록 수정

위 방법으로 3번 생성자 제약 관련 예제를 변경해보자.

```java
class Super {
    public Super() {
        // overrideMe();
        helpMethod();
    }

    public void overrideMe() {
        helpMethod();
    }

    // 도우미 메서드
    private void helpMethod() {
        System.out.println("Super help Method !");
    }
}
```

```java
class Sub extends Super {
    private String str;

    public Sub() {
        str = "Sub OverrideMe Method!!";
    }

    @Override
    public void overrideMe() {
        System.out.println(str);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```
![image](https://user-images.githubusercontent.com/42997924/193597055-e756c828-0aa7-4987-8ee3-60f24566ac59.png)

(↑ 실행 결과)

하위 클래스의 멤버 변수가 최기화 되기도 전에 호출되지 않고, 하위 클래스에서 재정의된 메서드가 정상적으로 동작한다.

## 7. 정리

- 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어 보는 것이 **유일**하다.
- 하위 클래스를 여러 개 만드는 동안 쓰이지 않는 메서드는 private로 제한하는 게 좋다.
- 공개될 상속용 클래스는 문서화된 내부 사용방식과 protected 메서드와 필드가 정해지며 영원히 책임져야 한다.
- **상속용으로 설계한 클래스는 배포 전 반드시 하위 클래스를 만들어 검증해봐야 한다.**

> 상속용 클래스는 영원히 책임져야 하지만 알잘딱깔센으로 설계해야 하는 아이러니가 있기 때문에 조심하자.
