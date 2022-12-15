# Item 52. 다중정의는 신중히 사용하라

Created: 2022년 11월 21일 오후 9:42
Last Edited Time: 2022년 11월 22일 오후 10:13

# 다중정의는 신중히 사용해라

```java
public String whatIsThisClass(final Set<?> set) {
    return "셋";
}

public String whatIsThisClass(final List<?> list) {
    return "리스트";
}

public String whatIsThisClass(final Collection<?> list) {
    return "컬렉션";
}

public static void main(String[] args) {
    Collection<?>[] collections = {
        new HashSet<String>(), new ArrayList<BigInteger>(), 
        new HashMap<String, Object>().values()
    };

    for (Collection<?> c : collections) {
        System.out.println(whatIsThisClass(c));
    }
}
```

하면 컬렉션만 세번 출력하지 다른 셋이나 리스트는 출력해주지 않는다.

왜냐면 오버라이딩된 세개의 메소드의 매개변수를 보면 각각 Set, List, Collection인데

이 세개중 최상위 타입이 바로 Collection이다. 그렇기 때문에 항상 Collection타입이기에 컬렉션만 세번 출력하게 된다.

### 어긋난 이유

재정의(Overriding) 메소드는 동적으로 선택이 되지만, 다중정의(Overloading) 메소드는 정적으로 선택된다.

### 클래스 상속으로 구현

```java
public class Burger {
    void print() {
        System.out.println("일반 버거");
    }
}

public class ChickenBurger extends Burger {

    @Override
    void print() {
        System.out.println("치킨버거");
    }

}

public class Whopper extends Burger {

    @Override
    void print() {
        System.out.println("와퍼");
    }

}

// Test Class

class BurgerTest {

    @Test
    void test() {
        final List<Burger> burgers = List.of(new Burger(), new ChickenBurger(), new Whopper());

        assertThat(burgers.get(0)).isExactlyInstanceOf(Burger.class);
        assertThat(burgers.get(1)).isExactlyInstanceOf(ChickenBurger.class);
        assertThat(burgers.get(2)).isExactlyInstanceOf(Whopper.class);

				for (Burger burger : burgers) {
            burger.print();
        }
    }
}
```

![스크린샷 2022-11-21 오후 10.08.03.png](Item%2052%20%E1%84%83%E1%85%A1%E1%84%8C%E1%85%AE%E1%86%BC%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%8B%E1%85%B4%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%89%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%AE%E1%86%BC%E1%84%92%E1%85%B5%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%85%E1%85%A1%2075a29e593f85477ba4c29b8955986f5a/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2022-11-21_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_10.08.03.png)

리스트에 들어간 컴파일 타입은 전부 Burger이지만, 이렇게 가장 하위에 정의된 메소드들을 실행시켜준다.

런타임 타입은 중요하지 않고 컴파일타임에 어떤 타입을 넣어주었는지에 의해 결정된다.

> 다중정의는 혼동을 일으키는 상황을 피해야 한다.
> 

안전하고 보수적으로 가기 위해서는 매개변수 수가 같은 다중정의는 만들지 말자.

가변인자를 사용하는 메소드라면 아예 다중정의 ❌ (다중정의 대신 메소드 이름을 달리 해주자 🔥)

## 정리

한 타입을 받는데에 있어 다중정의(Overloading)가 필요하다면 아예 연관되지 않은 자료형과 컬렉션을 사용하여 정의해주거나 모호하게 생성이 된다면 이름을 변경하여 여러개로 만들어주자.

Ex) Integer형의 메소드를 하나 선언했는데 Object를 매개변수로 받는 메소드를 다중정의 하는 경우 등