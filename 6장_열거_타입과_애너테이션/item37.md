배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드(아이템 35)로 인덱스를 얻는 코드가 있다.

```
public class Plant {
    enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

정원에 심을 식물들을 배열 하나로 관리하고, 이들을 생애주기(한해살이, 여러해살이, 두해살이)별로 묶어보자.
- 생애주기별로 총 3개의 집합을 만들고, 정원 한 바퀴를 돌며 각 식물을 해당 집합에 넣는다.

#### 코드 37-1. ordinal()을 배열 인덱스로 사용 - 따라하지 말 것!

```
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[])new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    plantsByLifeCycle[i] = new HashSet<>();
}
for (Plant p : garden) {
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
}
// 결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

동작은 하지만 문제가 있는 코드이다. 배열은 제네릭과 호환되지 않으나(아이템  28) 비검사 형변환을 수행해야 하고 깔끔하게 컴파일되지 않을 것이다.
배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
가장 심각한 문제는 **정확한 정숫값을 사용한다는 것을 직접 보증해야 한다는 점**이다.
- 정수는 열거 타입과 달리 타입 안전하지 않다.
- 잘못된 값을 사용하면 그대로 동작되거나 아니면 ArrayIndexOutOfBoundsException을 던질 것이다.

### 배열은 실질적으로 열거 타입(Enum) 상수를 값으로 매핑하는 일을 한다.

열거 타입을 키로 사용하도록 설꼐한 아주 빠른 Map 구현체가 존재하는데, 그게 바로 EnumMap이다. 

#### 코드 37-2. EnumMap을 사용해 데이터와 열거 타입을 매핑한다.
```
  Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
  	for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
    	plantsByLifeCycle.put(lc, new HashSet<>());
  	}

	for (Plant p : garden) {
		plantsByLifeCycle.get(p.lifeCycle).add(p);
    }
    System.out.println(plantsByLifeCycle);
```

- 더 짧고 명료하며 안전하고 성능도 원래 버전과 비등하다.
- 안전하지 않은 형변환을 쓰지 않고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 일도 없다.
- 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄된다.

EnumMap의 성능이 ordinal을 쓴 배열에 비견되는 이유는** 그 내부에서 배열을 사용하기 때문**. 
- 내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 성능을 모두 얻어낸 것이다.
- 여기서 EnumMap이 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타입 제네릭 타입 정보를 제공한다. (아이템 33)
- 스트림(아이템 45)을 사용해 맵을 관리하면 코드를 더 줄일 수 있다.

다음의 예외 동작을 거의 그대로 마방한 가장 단순한 형태의 스트림 기반 코드.

#### 코드 37-3. 스트림을 사용한 코드 1 - EnumMapm을 사용하지 않는다!

```
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> plant.lifeCycle)));
```

이 코드에선 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다는 문제가 있다.

매개변수 3개짜리 Collectors.groupBy 메서드는 mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있다.

#### 코드 37-4. 스트림을 사용한 코드 2 - EnumMap을 이용해 데이터와 열거 타입을 매핑했다.
```
System.out.println(Arrays.stream(garden).collect(
                groupingBy(p-> plant.lifeCycle,
                        () -> new EnumMap<>(LifeCycle.class), toSet())));
```

이 예처럼 단순한 프로그램에선 굳이 최적화할 필요가 없으나, 맵을 빈번히 사용하는 프로그램에서는 꼭 필요할 것이다.

스트림을 사용하면 EnumMap만 사용했을 때와는 살짝 다르게 동작한다.
EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다.
ex) 정원에 한해살이와 여러해살이 식물만 살고 두해살이는 없다면, EnumMap 버전에서 맵을 3개를 만들고 스트림 버전에서는 2개만 만든다.

두 열거 타입 값들을 매핑하느라 ordinal을 (두 번이나) 쓴 배열들의 배열을 본 적이 있을 것이다. 다음은 이 방식을 적용해 두 가지 상태(Phase)를 전이(Transition)와 매핑하도록 구현한 프로그램이다.

ex) 액체(LIQUID)에서 고체(SOLID)로의 전이는 응고(FREEZE)가 되고, 액체에서 기체(GAS)로의 전이는 기화(BOIL)가 된다.

#### 코드 37-5. 배열들의 배열의 인덱스에 ordinal을 사용 - 따라하지 말 것!

```
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다.
        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        // 한 상태에서 다른 상태로의 전이를 반환한다.
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}

```

- 컴파일러는 ordinal과 배열 인덱스의 관계를 알 도리가 없다.
- Phase나 Phase.Transition 열거 타입을 수정하면서 상전이 표 TRANSITIONS를 함께 수정하지 않거나 실수로 잘못 수정하면 런타입 오류가 날 것이다.
- ArrayIndexOutOfBoundsException이나 NullPointerException을 던질 수도 있고, 예외도 던지지 않고 작동할 수도 있다.
- 상전이 표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며 null로 채워지는 칸도 늘어날 것이다.

전이 하나를 얻으려면 이전 상태(from)와 이후 상태(to)가 필요하니, 맵 2개를 중첩하면 쉽게 해결할 수 있다. 안쪽 맵은 이전 상태와 전이를 연결하고 바깥 맵은 이후 상태와 안쪽 맵을 연결한다.
전이 전후의 두 상태를 전이 열거 타입 Transition의 입력으로 받아, 이 Transition 상수들로 중첩된 EnumMap을 초기화하면 된다.

#### 코드 37-6. 중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결했다.

```
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>> m = Stream.of(values())
                .collect(groupingBy(transition -> transition.from,
                        () -> new EnumMap<>(Phase.class),
                        toMap(t -> t.to, t -> t, (x, y) -> y, () -> new EnumMap<>(Phase.class))));

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

이 맵의 타입인 Map<Phase, Map<Phase, Transition>>은 "이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵"이라는 뜻이다.
- 맵의 맵을 초기화하기 위해 수집기(java.util.stream.Collector) 2개를 차례로 사용했다.
- 첫 번째 수집기인 groupingBy에서는 전이를 이전 상태를 기준으로 묶기
- 두 번째 수집기의 병합 함수인 (x,y) -> y는 선언만 하고 실제로는 쓰이지 않는데, 이는 단지 EnumMap을 얻으려면 맵 팩터리가 필요하고 수집기들은 점층적 팩터리(telescoping factory)를 제공하기 때문이다.

여기서 새로운 상태인 플라스마(PLASMA)를 추가해보자.
이 상태와 연결된 전이는 2개다.

- 첫 번째는 기체에서 플라스마로 변하는 이온화(IONIZE)
- 두 번째는 플라스마에서 기체로 변하는 탈이온화(DEIONIZE)

배열로 만든 코드 37-5를 수정하려면 새로운 상수를 Phase에 1개, Phase.Transition에 2개를 추가하고, 원소 9개짜리인 배열들의 배열을 원소 16개 짜리로 교체해야 한다.

원소 수를 너무 적거나 많이 기입하거나, 잘못된 순서로 나열하면 이 프로그램은 (컴파일은 통과하더라도) 런타임에 문제를 일으킬 것이다.
반면, EnumMap 버전에서는 상태 목록에 Plasma를 추가하고, 전이 목록에 IONIZE(GAS, PLASMA)와 DEIONIZE(PLASMA, GAS)만 추가하면 끝이다.

#### 코드 37-7. EnumMap 버전에 새로운 상태 추가하기
```
public enum Transition {
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
    BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
    SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
    IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
}
```
나머지는 기존 로직에서 잘 처리해주어 잘못 수정할 가능성이 극히 적다.
실제 내부에선 맵들의 맵이 배열들의 배열로 구현되니 낭비되는 공간과 시간도 거의 없이 명확하고 안전해 유지보수 하기 좋다.

- 코드를 간략히 하기 위해 앞의 코드에선 해당 전이가 없을 때 null을 사용했다.
- null을 사용하는 것은 런타임에 NullPointerException을 일으키는 안 좋은 습관이지만, 이 문제를 깔끔히 해결하려면 코드가 아주 길어져서 어쩔 수 없이 예외 처리 코드를 생략했다.

## 핵심 정리

- **배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용**하라.
- 다차원 관계는 EnumMap<..., EnumMap<...>>으로 표현하라.
- "애플리케이션 프로그래머는 Enum.ordinal을 웬만해선 사용하지 말아야 한다.(아이템 35)"는 일반 원칙의 특수한 사례.
