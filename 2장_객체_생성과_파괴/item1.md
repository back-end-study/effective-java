![](https://velog.velcdn.com/images/jiyeong/post/49c33300-8ae5-49ca-bd5a-289c130b0063/image.jpeg)

# The Java Language Specification

## 자바가 지원하는 타입(자료형)

- 인터페이스(Interface) : Annotation은 인터페이스의 일종
- 클래스(Class) : 열거 타입(Enum)은 클래스의 일종
- 배열(Array)
 ---- 여기까지가 참조 타입(reference type)이라고 불림!
- 기본 타입(Primitive)

클래스의 인스턴스와 배열은 객체인 반면, 기본 타입 값은 그렇지 않음.

### 클래스의 멤버

필드, 메서드, 멤버 클래스, 멤버 인터페이스가 있다.
**메서드 시그니처**는 메서드 이름과 입력 매개변수의 타입들로 이뤄진다.
반환값의 타입은 시그니처에 포함되지 않는다.

-- 자바 명세에 정의되지 않은 기술 용어

## API(Application Programming Interface)

프로그래머가 클래스, 인터페이스, 패키지를 통해 접근할 수 있는 모든 클래스, 인터페이스, 생성자, 멤버, 직렬화된 형태(serialized form)을 말한다.

API를 사용하는 프로그램 작성자를 그 API이 사용자라고 하고, 이를 사용하는 클래스는 그 API의 클라이언트라고 한다.

클래스, 인터페이스, 생성자, 멤버, 직렬화된 형태를 총칭해서 API 요소라고 한다. 

패키지의 공개 API는 그 패키지의 모든 public 클래스와 인터페이스의 public 혹은 protected 멤버와 생성자로 구성된다.

![](https://velog.velcdn.com/images/jiyeong/post/3620e479-5ff3-443e-89b6-ea530eb2b10d/image.jpeg)

## Item 1 : 생성자 대신 정적 팩터리 메서드를 고려하라

클래스는 생성자와 별도로 정적 팩터리 메서드(static factory method)를 제공할 수 있다. 그 클래스의 인스턴스를 반환하는 단순한 정적 메서드를 일컫는다.
    
```
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE: Boolean.FALSE:
}
```
이 메서드는 기본 타입인 boolean 값을 받아 Boolean 객체 참조로 변환해준다.
클래스는 클라이언트에 public 생성자 대신 (혹은 생성자와 함께) 정적 팩터리 메서드를 제공할 수 있다.


### 정적 팩터레 메서드 사용 시 장단점

#### 장점
1. **이름을 가질 수 있다.**
	동일한 시그니처의 생성자 두 개를 가질 수 없다.
    정적 팩토리 메서드를 사용해서 특징을 쉽게 표현할 수 있다.

```
package me.whiteship.chapter01.item01;

import java.util.*;

public class Order {

    private boolean prime;

    private boolean urgent;

    private Product product;

    private OrderStatus orderStatus;

    public static Order primeOrder(Product product) {
        Order order = new Order();
        order.prime = true;
        order.product = product;

        return order;
    }

    public static Order urgentOrder(Product product) {
        Order order = new Order();
        order.urgent = true;
        order.product = product;
        return order;
    }

    public static void main(String[] args) {

        Order order = new Order();
        if (order.orderStatus == OrderStatus.DELIVERED) {
            System.out.println("delivered");
        }
    }

 }
```
    
2. **호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.**

	
    불변 클래스는 인스턴스를 미리 만들어 두거나 새로 생성한 인스턴스를 캐싱해 재활용하는 식으로 불필요한 객체 생성을 피할 수 있음.
    매개변수 별로 각기 다른 리턴 값을 리턴함. 
    예) Boolean.valueOf(boolean) 메서드는 객체를 아예 생성하지 않음. _플라이웨이트 패턴_도 이와 비슷한 기법이다.
    
    **플라이웨이트 패턴**이란? 어떤 클래스의 인스턴스 하나로 여러 개의 ‘가상 인스턴스’를 제공하는 패턴으로 어떤 클래스의 인스턴스가 아주 많이 필요하지만 모두 똑같은 방식으로 제어해야 할 때 유용하게 쓰인다.
    
    이렇게 반복되는 요청에 같은 객체를 반환하는 식으로 인스턴스 활성화를 통제하는 것을 인스턴스 통제(instance-control) 클래스라고 한다.
   
   _왜 인스턴스를 통제하는 것인가?_
    싱글턴으로 만들 수 있으며, 인스턴스화 불가(noninstantiable)로 만들 수도 있다. 또, 불변 값 클래스에서 동치인 인스턴스가 하나뿐임을 보장함
    (a == b일 때만 a.equals(b)가 성립하는 것)
    열거 타입은 인스턴스가 하나만 만들어짐을 보장해준다.
    
  ```
  package me.whiteship.chapter01.item01;

  /**
   * 이 클래스의 인스턴스는 #getInstance()를 통해 사용한다.
   * @see #getInstance()
   */
  public class Settings {

      private boolean useAutoSteering;

      private boolean useABS;

      private Difficulty difficulty;

      private Settings() {}

      private static final Settings SETTINGS = new Settings();

      public static Settings getInstance() {
          return SETTINGS;
      }
    }
```
 ```
package me.whiteship.chapter01.item01;

import java.util.EnumSet;
import java.util.Set;

public class Product {

  public static void main(String[] args) {
        Settings settings1 = Settings.getInstance();
        Settings settings2 = Settings.getInstance();

        System.out.println(settings1);
        System.out.println(settings2);

        Boolean.valueOf(false);
        EnumSet.allOf(Difficulty.class);
    }
}
```
    
3. **반환 타입의 하위 타입 객체를 반환할 수 있다.**
	이는 반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연성을 제공한다.
    API를 만들 때 이를 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 구현 할 수 있음.
    인터페이스 기반 프레임워크, 인터페이스에 정적 메소드
    
    인터페이스 타입을 사용할 수 있다. 하위 클래스를 리턴할 수 있다.
    
    ```
	package me.whiteship.chapter01.item01;

	public interface HelloService {

      String hello();

      static String hi() {
          prepareMessage();
          return "hi";
      }

      static private void prepareMessage() {
      }

      static String hi1() {
          prepareMessage();
          return "hi";
      }

      static String hi2() {
          prepareMessage();
          return "hi";
      }

      default String bye() {
          return "bye";
      }
	}
	```
    
    ```
	package me.whiteship.chapter01.item01;

    import me.whiteship.hello.ChineseHelloService;

    import java.lang.reflect.Constructor;
    import java.lang.reflect.InvocationTargetException;
    import java.util.Optional;
    import java.util.ServiceLoader;

    public class HelloServiceFactory {

    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        ServiceLoader<HelloService> loader = ServiceLoader.load(HelloService.class);
        Optional<HelloService> helloServiceOptional = loader.findFirst();
        helloServiceOptional.ifPresent(h -> {
            System.out.println(h.hello());
        });

        HelloService helloService = new ChineseHelloService();
        System.out.println(helloService.hello());

    //        Class<?> aClass = Class.forName("me.whiteship.hello.ChineseHelloService");
    //        Constructor<?> constructor = aClass.getConstructor();
    //        HelloService helloService = (HelloService) constructor.newInstance();
    //        System.out.println(helloService.hello());
        }

    }
	```
    
4. **입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.**
	반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관 없다. EnumSet.
    
    
5. **정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.**
	**서비스 제공 프레임워크**는 3개의 핵심 컴포넌트로 이뤄진다. 
    1) 구현체의 동작을 정의하는 서비스 인터페이스(service interface)
    2) 제공자가 구현체를 등록할 때 사용하는 제공자 등록 API(provider registration API)
    3) 클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 서비스 접근 API(service access API)
    4) 서비스 제공자 인터페이스(service provider interface) : 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명해 줌. 
   	_유연한 정적 팩터리_ : 클라이언트는 서비스 접근 API를 사용할 때 원하는 구현체의 조건을 명시할 수 있으며, 조건을 명시하지 않으면 기본 구현체를 반환하거나 지원하는 구현체들을 하나씩 돌아가며 반환한다.
    
#### 단점

1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

#### 정적 팩토리 메서드 명명 방식
- from : 매개변수를 하나 받아서 해당 타입의 인스ㅓㄴ스를 반환하는 형변환 메서드
```
Date d = Date.from(instant);
```
- of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
```
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
```
- valueOf : from과 of이 더 자세한 버전
```
BigInteger prime : BigInteger.valueOf(Integer.MAX_VALUE);
```
- instance 혹은 getInstance : (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다.
```
stackWalker luke = StackWalker.getInstance(options);
```
- create 혹은 newInstance : instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
```
Object newArray = Array.newInstance(classObject, arrayLen);
```
- getType : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입.
```
FileStore fs = Files.getFileStore(path);
```
- newType : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입.
```
BufferedReader br = Files.newBufferedReader(path);
```
- type : getType과 newType의 간결한 버전
```
List<Complaint> litany = Collections.list(legacyLitany);
```

### 핵심 정리
정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다.
그래도 정적 팩토리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 쓰지는 말자.

- **열거 타입**은 인스턴스가 하나만 만들어짐을 보장한다
- 같은 객체가 자주 요청되는 상황이라면 **플라이웨이트 패턴**을 사용할 수 있다
- 자바 8부터는 **인터페이스가 정적 메서드**를 가질 수 없다는 제한이 풀렸기 때문에 인스턴스화 불가 동반 클래스를 둘 이유가 별로 없다.
- **서비스 제공자 프레임워크**를 만드는 근간이 된다.
- **서비스 제공자 인터페이스**가 없다면 각 구현체를 인스턴스로 만들 때 리플렉션을 사용해야 한다.
- 브리지 패턴
- 의존 객체 주입 프레임워크


#### 1. 열거 타입 **Enum**eration

- 상수 목록을 담을 수 있는 데이터 타입.
- 특정한 변수가 가질 수 있는 값을 제한할 수 있다.
	**타입-세이프티(Type-Safety)**를 보장할 수 있다.
- **싱글톤 패턴**을 구현할 때 사용하기도 한다.

```
package me.whiteship.chapter01.item01;

public enum OrderStatus {

    PREPARING(0), SHIPPED(1), DELIVERING(2), DELIVERED(3);

    private int number;

    OrderStatus(int number) {
        this.number = number;
    }
}
```
Q1. 특정 enum 타입을 가질 수 있는 모든 값을 순회하여 출력하기
Q2. enum은 자바의 클래스처럼 생성자, 메소드, 필드를 가질 수 있는가?

OrderStatus.**values()** //가져오는 방법!

```
Arrays.stream(OrderStatus.values()).forEach(System.out::println);

```
Q3. enum의 값은 == 연사자로 동일성을 비교할 수 있는가?

```
Order order = new Order();
if(Order.orderStatus == OrderStatus.DELIVERED){
	System.out.println("delivered");
}
```

과제) enum을 key로 사용하는 Map을 정의하시오. 혹은 enum을 담고 있는 Set을 만드시오.

팁 - EnumMap / EnumSet을 사용하기
  - EnumMap : 
    - EnumMap은 Map 인터페이스에서 키를 특정 enum 타입만을 사용하도록 하는 구현체.
    - enum 은 ordinal 이라는 순차적인 정수값을 가짐.
    - EnumMap은 내부에 데이터를 Array에 저장합니다.
    그러면 우리가 많이 사용하는 HashMap 처럼 해시를 만들고 해시 충돌(Hash Collision)에 대응하는 작업자체가 필요 없게 되는 것이죠.

  - EnumSet : 
    - Set인터페이스를 구현하여 어떤 Set 구현체와도 함께 사용할 수 있으며 타입 안전.
    - EnumSet 내부는 비트 벡터로 구현되어있으며, 원소가 총 64개 이하라면 EnumSet 전체를 long 변수 하나로 표현하여 비트필드에 비견되는 성능을 보여줌.
    - removeAll과 retainAll 과 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현하였다.

#### 플라이웨이트 패턴

- 객체를 가볍게 만들어 메모리 사용을 줄이는 패턴
- 자주 변하는 속성(또는 외적인 속성, extrinsit)과 변하지 않는 속성(또는 내적인 속성, intrinsit)을 분리하고 재사용하여 메모리 사용을 줄일 수 있다.

#### 인터페이스에 정적 메소드
자바 8과 9에서 주요 인터페이스의 변화

- 기본 메소드와 정적 메소드를 가질 수 있다.
- 기본 메소드
	- 인터페이스에서 메소드 선언 뿐 아니라, 기본적인 구현체까지 제공할 수 있다.
    - 기존의 인터페이스를 구현하는 클래스에 새로운 기능을 추가할 수 있다.
- 정적 메소드
	- 자바 9부터 private static 메소드도 가질 수 있다.
    - 단, private 필드는 아직도 선언할 수 없다.
 
#### 서비스 제공자 프레임워크
확장 가능한 애플리케이션을 만드는 방법

- 주요 구성 요소
	- 서비스 제공자 인터페이스(SPI)와 서비스 제공자(서비스 구현체)
    - 서비스 제공자 등록 API(서비스 인터페이스의 구현체를 등록하는 방법)
    - 서비스 접근 API(서비스의 클라이언트가 서비스 인터페이스의 인스턴스를 가져올 때 사용하는 API)
    
- 다양한 변형
	- 브릿지 패턴
    - 의존 객체 주입 프레임워크
    
```
package me.whiteship.chapter01.item01;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class App {

    public static void main(String[] args) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        HelloService helloService = applicationContext.getBean(HelloService.class);
        System.out.println(helloService.hello());
    }
}
```

#### 리플렉션
- 클래스로더를 통해 읽어온 클래스 정보(거울에 반사된 정보)를 사용하는 기술
- 리플렉션을 사용해 클래스를 읽어오거나, 인스턴스를 만들거나, 메소드를 실행하거나, 필드의 값을 가져오거나 변경하는 것이 가능하다.
- 언제 사용하는가?
	- 특정 애노테이션이 붙어있는 필드 또는 메소드 읽어오기(JUnit, Spring)
    
    ```
	package me.whiteship.chapter01.item01;

    import me.whiteship.hello.ChineseHelloService;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class AppConfig {

      @Bean
      public HelloService helloService() {
          return new ChineseHelloService();
      }
	}
	```
    - 특정 이름 패턴에 해당하는 메소드 목록을 가져와 호출(getter, setter)
출처 : 이펙티브 자바, 백기선님 이펙티브 자바 강의
