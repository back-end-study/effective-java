# item67 최적화는 신중히 하라

모든 사람이 마음 깊이 새겨야 할 최적화 격언 세 개를 소개 한다.
> (맹목적인 어리석음을 포함해) 그 어떤 핑계보다 효율성이라는 이름 아래 행해진 컴퓨팅 죄악이 더 많다. (심지어 효율을 높이지도 못하면서) - 윌리엄 울프

> (전체의 97% 정도인) 자그마한 효율성은 모두 잊자. 섣부른 최적화가 만악의 근원이다. - 도널드 크누스

> 최적화를 할 때는 다음 두 규칙을 따르라.
> 첫 번째, 하지 마라.
> 두 번째, (전문가 한정) 아직 하지 마라. 다시 말해, 완전히 명백하고 최적화되지 않은 해법을 찾을 때까지는 하지 마라.
> - M. A 잭슨

최적화는 좋은 결과보다는 해로운 결과로 이어지기 쉽고, 섣불리 진행하면 특히 더 그렇다. 빠르지도 않고 제대로 동작하지도 않으면서 수정하기는 어려운 소프트웨어를 탄생시키는 것이다.

## 빠른 프로그램보다 좋은 프로그램을 작성하라.
성능 때문에 견고한 구조를 희생하지 말자. 좋은 프로그램이지만 원하는 성능이 나오지 않는다면 그 아키텍처 자체가 최적화할 수 있는 길을 안내해줄 것이다. 좋은 프로그램은 정보 은닉 원칙을 따르므로 개별 구성요소의 내부를 독립적으로 설계할 수 있다. 따라서 시스템의 나머지에 영향을 주지 않고도 각 요소를 다시 설계할 수 있다.

## 설계 단계에서 성능을 반드시 염두에 두어야 한다.
구현상의 문제는 나중에 최적화해 해결할 수 있지만, 아키텍처의 결함이 성능을 제한하는 상황이라면 시스템 전체를 다시 작성하지 않고는 해결 불가능할 수 있다.

### 성능을 제한하는 설계를 피하라.
완성 후 변경하기 가장 어려운 설계 요소는 컴포넌트끼리 혹은 외부 시스템과의 소통 방식이다. API, 네트워크 프로토콜, 영구 저장용 데이터 포맷 등이 대표적이다. 이런 설계 요소들은 완성 후 변경하기 어렵거나 불가능할 수 있으며, 시스템 성능을 심각하게 제한할 수 있다.

### API를 설계할 때 성능에 주는 영향을 고려하라.
public 타입을 가변으로 만들면 불필요한 방어적 복사를 수없이 유발할 수 있다. 컴포지션으로 해결할 수 있음에도 상속 방식으로 설계한 public 클래스는 상위 클래스에 영원히 종속되며 그 성능 제약까지도 물려받게 된다. 또 구현체에 종속되면 나중에 더 빠른 구현체가 나오더라도 이용하지 못하게 된다.

[java.awt.Component](https://github.com/openjdk/jdk/blob/master/src/java.desktop/share/classes/java/awt/Component.java) 클래스의 getSize를 생각해보자. 이 메서드는 [Dimension](https://github.com/openjdk/jdk/blob/master/src/java.desktop/share/classes/java/awt/Dimension.java) 인스턴스를 반환한다.
Dimension은 가변이라 getSize를 호출하는 모든 곳에서 Dimension 인스턴스를 새로 생성해야만 한다.
더 나은 설계는 Dimension을 불변으로 만드는게 이상적이다. getSize를 getWidth와 getHeight로 나누는 방법도 있다. Dimension 객체의 기본 타입 값들을 따로 반환하는 방식이다. 실제로 자바 2에서는 성능을 해결하고자 Component 클래스에 이 메서드들을 추가 했다. 하지만 기존 클라이언트 코드는 여전히 getSize 메서드를 호출하며 결정의 폐해를 감내하고 있다.

잘 설계된 API는 성능도 좋은 게 보통이다. 그러니 성능을 위해 API를 왜곡하는 건 매우 안 좋은 생각이다.
신중하게 설계하여 깨끗하고 명확하고 멋진 구조를 갖춘 프로그램을 완성한 다음에야 최적화를 고려해볼 차례가 된다. 물론 성능에 만족하지 못할 경우에 한정된다.

### "최적화 시도 전후로 성능을 측정하라"
일반적으로 90%의 시간을 단 10%의 코드에서 사용한다는 사실을 기억해두자.

프로파일링 도구는 최적화 노력을 어디에 집중해야 할지 찾는 데 도움을 준다. 자바 코드의 상세한 성능을 알기 쉽게 보여주는 마이크로 벤치마킹 프레임워크 [jmh](https://github.com/openjdk/jmh) 도 언급해둘만 하다.

최적화 시도 전후의 성능 측정은 성능 모델이 덜 정교한(C와 C++ 같은 전통적인 언어에 비해) 자바에서는 중요성이 더욱 크다. 자바는 기본 연산에 드는 상대적인 비용을 덜 명확하게 정의하고 있다. 프로그래머가 작성하는 코드와 CPU에서 수행하는 명령 사이의 '추상화 격차'가 커서 최적화로 인한 성능 변화를 일정하게 예측하기가 그만큼 더 어렵다.

자바의 성능 모델은 정교하지 않을뿐더러 구현 시스템, 릴리스, 프로세서마다 차이가 있다. 여러 가지 자바 플랫폼이나 여러 하드웨어 플랫폼에서 구동한다면 최적화의 효과를 각각에서 측정해야 한다. 그러다 보면 다른 구현 혹은 하드웨어 플랫폼 사이에서 성능을 타협해야 하는 상황도 마주할 것이다.