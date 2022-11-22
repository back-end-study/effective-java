# item51 메서드 시그니처를 신중히 설계하라

## 메서드 이름을 신중히 짓자
항상 표준 명명 규칙(item68)을 따라야 한다.  
이해할 수 있고, 같은 패키지에 속한 다른 이름들과 일관되게 짓는게 최우선 목표다.  
긴 이름은 피하자.  
애매하면 자바 라이브러리의 API를 참조하라.

## 편의 메서드를 너무 많이 만들지 말자
편의 메서드 : 복잡한 작업을 더 쉽게 해결할 수 있도록 편의를 위해 만든 메서드  
모든 메서드는 각각 자신의 소임을 다해야 한다. 메서드가 너무 많은 클래스는 익히고, 사용하고, 문서화하고, 테스트하고, 유지보수하기 어렵다.  
인터페이스도 메서드가 너무 많으면 이를 구현하는 사람과 사용하는 사람 모두를 고통스럽게 한다.  
클래스나 인터페이스는 자신의 각 기능을 완벽히 수행하는 메서드로 제공해야 한다. 아주 자주 쓰일 경우에만 별도의 약칭 메서드를 두기 바란다.  
**확신이 서지 않으면 만들지 말자**

## 매개변수 목록은 짧게 유지하자
4개 이하가 좋다. 4개가 넘어가면 매개변수를 전부 기억하기 쉽지 않다.
**같은 타입의 매개변수 여러 개가 연달아 나오는 경우가 특히 해롭다.** 실수로 순서를 바꿔 입력해도 그대로 컴파일되고 실행된다. 단지 의도와 다르게 동작할 뿐이다.

### 매개 변수 목록을 짧게 줄여줄이는 방법
#### 여러 메서드로 쪼갠다.
쪼개진 메서드 각각은 원래 매개변수 목록의 부분집합을 받는다. 잘못하면 메서드가 너무 많아 질 수 있지만 직교성(공통점이 없는 기능들이 잘 분리되어 있다)을 높여 오히려 메서드 수를 줄여주는 효과도 있다.
java.util.List 지정된 범위에서 주어진 원소의 인덱스를 찾아야 한다면 하나의 메서드로 부분 리스트의 시작, 부분리스트의 끝, 찾을 원소까지 총 3개의 매개변수가 필요하다.
하지만 List의 subList, indexOf 두개의 메서드를 사용하면 원하는 목적을 이룰 수 있다.

#### 도우미 클래스를 만든다.
일반적으로 이런 도우미 클래스는 정적 멤버 클래스(item24)로 둔다. 특히 잇따른 매개변수 몇 개를 독립된 하나의 개념으로 볼 수 있을 때 추천하는 기법이다.
카드 게임을 클래스로 만든다고 해보자. 메서드를 호출할 때 카드의 숫자와 무늬를 뜻하는 두 매개변수를 항상 같은 순서로 전달할 것이다. 따라서 이 둘을 묶는 도우미 클래스를 만들어 하나의 매개변수로 주고받으면 API는 물론 클래스 내부 구현도 깔끔해질 것이다.

#### 객체 생성에 사용한 빌더 패턴을 메서드 호출에 응용한다.
매개변수가 많을 때, 특히 그중 일부는 생략해도 괜찮을 때 도움이 된다.
https://stackoverflow.com/questions/13954672/adapting-the-builder-pattern-for-method-invocation
> A third technique that combines aspects of the first two is to adapt the Builder pattern (Item 2) from object construction to method invocation. If you have a method with many parameters, especially if some of them are optional, it can be beneficial to define an object that represents all of the parameters and to allow the client to make multiple “setter” calls on this object, each of which sets a single parameter or a small, related group. Once the desired parameters have been set, the client invokes the object’s “execute” method, which does any final validity checks on the parameters and performs the actual computation.

## 매개변수의 타입으로는 클래스보다는 인터페이스가 더 낫다. (item64)
매개변수로 적합한 인터페이스가 있다면 그 인터페이스를 직접 사용하자.
예를 들어 메서드에 HashMap을 넘길 일은 전혀 없다. 대신 Map을 사용하자. 그러면 HashMap뿐 아니라 TreeMap, ConcureentHashMap, TreeMap의 부분맵 등 어떤 Map 구현체도 인수로 건넬 수 있다. 심지어 아직 존재하지 않는 Map도 가능하다. 인터페이스 대신 클래스를 사용하면 클라이언트에게 특정 구현체만 사용하도록 제한하는 꼴이며, 혹시라도 입력 데이터가 다른 형태로 존재한다면 명시한 특정 구현체의 객체로 옮겨 담느라 비싼 복사 비용을 치러야 한다.

## 매개변수의 타입으로는 boolean보다 원소 2개짜리 열거 타입이 낫다. (boolean을 받아야 의미가 명확할 때는 예외)
온도계 클래스의 정적 팩터리 메서드가 화씨/섭씨를 입력 받아 적합한 온도계 인스턴스를 생성해준다고 해보자.
```java
public enum TemperatureScale { FAHREHEIT, CELSIUS }

Thermometer.newInstance(ture);
Thermometer.newInstance(TemperatureScale.CELSIUS); // 하는 일을 훨씬 명확히 해준다.
```
나중에 캘빈온도도 지원해야 한다면 열거 타입에 캘빈온도(KELVIN)를 추가하면 된다.
또한, 온도 단위에 대한 의존성을 개별 열거 타입 상수의 메서드 안으로 리팩터링해 넣을 수도 있다. double 값을 받아 섭씨온도로 변환해주는 메서드를 열거 타입 상수 각각에 정의해둘수도 있다.
