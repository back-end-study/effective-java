# 아이템65. 리플렉션보다는 인터페이스를 사용하라

## 1. 리플렉션이란?

> java.lang.reflect API

- 실행 중에 임의의 클래스에 접근할 수 있다.
    - 클래스, 인터페이스, 메서드를 찾을 수 있다.
    - 객체를 생성하거나 변수를 변경할 수 있고 메서드를 호출 할 수 있다.
    - 나아가 Constructor, Method, Field 인스턴스를 이용해 실제 생성자, 메서드, 필드를 조작할 수 있다.

## 2. 리플렉션 단점

1. **컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없다.**

   예외 검사도 마찬가지다. 프로그램이 리플렉션 기능을 써서 존재하지 않는 혹은 접근할 수 없는 메서드를 호출하려고 하면 런타임 오류가 발생한다.

2. **리플렉션을 이용하면 코드가 지저분하고 장황해진다.**
3. **성능이 떨어진다.**

   리플렉션을 통한 메서드 호출은 일반 메서드 호출보다 훨씬 느리다.

## 3. 리플렉션 예제 코드

- 코드 분석 도구나 의존관계 주입 프레임워크처럼 리플렉션을 써야 하는 복잡한 애플리케이션이 몇 가지 있다.
    - 하지만 단점이 명확하기 때문에 리플렉션 사용을 점차 줄이고 있다.

**리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피하고 이점만 취할 수 있다.**

컴파일타임에 이용할 수 없는 클래스를 사용해야만 하는 프로그램은 비록 컴파일타임이라도 적절한 인터페이스나 상위 클래스를 이용할 수는 있을 것이다.

다행히 이런 경우라면 **리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자.**

```java
public static void main(String[] args) {
    // 클래스 이름을 Class 객체로 변환
    Class<? extends Set<String>> cl = null;
    try {
        cl = (Class<? extends Set<String>>) Class.forName(args[0]); // 비검사 형변환!
    } catch (ClassNotFoundException e) {
        fatalError("'" + args[0] + "' 클래스를 찾을 수 없습니다.");
    }

    // 생성자를 얻는다
    Constructor<? extends Set<String>> cons = null;
    try {
        cons = cl.getDeclaredConstructor();
    } catch (NoSuchMethodException e) {
        fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
    }

    // 집합의 인스턴스 생성
    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch (IllegalAccessException e) {
        fatalError("생성자에 접근할 수 없습니다.");
    } catch (InstantiationException e) {
        fatalError("클래스를 인스턴스화할 수 없습니다.");
    } catch (InvocationTargetException e) {
        fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
    } catch (ClassCastException e) {
        fatalError("Set을 구현하지 않은 클래스입니다.");
    }

    // 생성한 집합을 사용한다
    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}

private static void fatalError(String msg) {
    System.err.println(msg);
    System.exit(1);
}
```

- 이 코드는 `Set<String>` 인터페이스의 인스턴스를 생성하는데, 정확한 클래스는 명령줄의 첫 번째 인수로 확정한다.
- 그리고 생성한 집합(Set)에 두 번째 이후의 인수들을 추가한 다음 화면에 출력한다.
- 첫 번째 인수와 상관없이 이후의 인수들에서 중복은 제거한 후 출력한다.
- 반면, 이 인수들이 출력되는 순서는 첫 번째 인수로 지정한 클래스가 무엇이냐에 따라 달라진다.
    - **java.util.HashSet** 을 지정하면 무작위 순서, **java.util.TreeSet** 을 지정하면 정렬되어 출력된다.

이 코드는 `리플렉션의 단점` 두 가지를 보여준다.

- 런타임에 총 여섯가지 되는 예외를 던질 수 있다.

  인스턴스를 리플렉션 없이 생성했다면 컴파일타임에 잡아낼 수 있었을 예외들이다.

- 클래스 이름만으로 인스턴슬르 생성해내기 위해 무려 25줄이나 되는 코드를 작성했다.

  리플렉션이 아니라면 생성자 호출 한 줄로 끝났을 일이다.


두 단점 모두 객체를 생성하는 부분에만 국한된다.

객체를 일단 만들면 그 후의 코드는 Set 인스턴스를 사용할 때와 똑같다.

그래서 실제 프로그램에서는 이런 제약에 영향받는 코드는 일부에 지나지 않는다.

## 4. 리플렉션이 사용되는 경우

- 드물긴 하지만, 리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합하다. (`Spring의 DI`)
- 이 기법은 버전이 여러 개 존재하는 외부 패키지를 다룰 때 유용하다.
- 가동할 수 있는 최소한의 환경, 즉 주로 가장 오래된 버전만을 지원하도록 컴파일한 후, 이후 버전의 클래스와 메서드 등은 리플렉션으로 접근하는 방식이다.
- 이렇게 하려면 접근하려는 새로운 클래스나 메서드가 런타임에 존재하지 않을 수 있다는 사실을 반드시 감안해야 한다.

즉, 같은 목적을 이룰 수 있는 대체 수단을 이용하거나 기능을 줄여 동작하는 등의 적절한 조치를 취해야 한다.

## 5. 리플렉션 성능은 진짜 느릴까

책에서는 입력 매개변수 없이 int를 반환하는 메서드로 실험해보니 11배나 느렸다고 한다. 실제로 느린지 확인해보자.

리플렉션을 통해 Method도 호출해보고 Field 값도 변경해보려고 한다.(무려 private을 접근, 변경해봤다)

먼저 리플렉션을 위한 샘플 class를 작성했다.

```java
public class ReflectionSample {

    public String str1 = "publicField";
    private String str2 = "privateField";

    public ReflectionSample() {

    }

    public int methodZero(int n) {
        System.out.println("methodZero = " + n);
        return n;
    }

    private int methodPrivate(int n) {
        System.out.println("methodPrivate = " + n);
        return n;
    }
}
```

private 필드와 메서드를 가지고 있다.

```java
public class ReflectionTest {
    public static void main(String[] args) throws
            ClassNotFoundException,
            NoSuchMethodException,
            InvocationTargetException,
            IllegalAccessException,
            NoSuchFieldException {
        // 1. class 찾기
        Class<?> clazz = Class.forName("study.yjpark.chapter09.item65.ReflectionSample");
        System.out.println("class = " + clazz.getName());

        // 2. constructor 찾기
        Constructor<?> constructor = clazz.getDeclaredConstructor();
        System.out.println("constructor = " + constructor);

        // 3. Method 찾기
        Method[] methods = clazz.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println("method = " + method);
        }

        // 4. Field 찾기
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            System.out.println("field = " + field);
        }

        ReflectionSample sampleInstance = new ReflectionSample();

        // 5. Method 호출
        Method methodZero = clazz.getDeclaredMethod("methodZero", int.class);
        methodZero.invoke(sampleInstance, 10); // arg1 : 호출 객체, arg2 : 전달할 파라미터
        Method methodPrivate = clazz.getDeclaredMethod("methodPrivate", int.class);
        methodPrivate.setAccessible(true); // private 메서드 접근 가능 설정
        methodPrivate.invoke(sampleInstance, 20); // private 그대로 호출 시 IllegalAccessException 발생

        // 6. Field 변경
        Field str2 = clazz.getDeclaredField("str2");
        str2.setAccessible(true); // private 필드 접근 가능 설정
        str2.set(sampleInstance, "변경했지롱");
        System.out.println("changedStr2 = " + str2.get(sampleInstance));
    }
}
```

작성된 리플렉션의 최소한의 기능을 돌려보면 잘 동작하는 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/42997924/207665931-d6fa399e-9d2c-41a6-b909-b99bac1ef07a.png)

이제 속도가 느린지 확인해보자.

```java
public class ReflectionSpeed {
    public static void main(String[] args) throws
            ClassNotFoundException,
            NoSuchMethodException,
            InvocationTargetException,
            IllegalAccessException,
            NoSuchFieldException, InstantiationException {
        StopWatch stopWatch = new StopWatch("리플렉션 성능 비교");

        stopWatch.start("리플렉션 Method 호출");
        Class<?> clazz = Class.forName("study.yjpark.chapter09.item65.ReflectionSample");
        Constructor<?> constructor = clazz.getDeclaredConstructor();
        ReflectionSample sampleInstance = (ReflectionSample) constructor.newInstance();
        Method methodZero = clazz.getDeclaredMethod("methodZero", int.class);
        methodZero.invoke(sampleInstance, 30);
        stopWatch.stop();

        stopWatch.start("일반적인 Method 호출");
        ReflectionSample common = new ReflectionSample();
        common.methodZero(30);
        stopWatch.stop();

        System.out.println(stopWatch.prettyPrint());
    }
}
```

![image](https://user-images.githubusercontent.com/42997924/207668144-89ba6d2a-95db-4bfb-923a-c2818a24fcf4.png)

1091배 정도 느렸다. (이게 맞나..)

## 6. 정리

- 리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합하다.
- 하지만 단점도 많기 때문에 되도록 객체 생성에만 사용하고, 적절한 인터페이스나 상위 클래스로 형변환해 사용하자.

**Reference**

- [이펙티브자바 서적](http://www.yes24.com/Product/Goods/65551284)