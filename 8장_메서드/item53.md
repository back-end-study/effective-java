# 가변인수(varags)
- 가변인자 함수는 함수가 몇 개의 인자를 받을지 정해지지 않은 함수
- 위치, 키워드 인자의 개수가 많아지거나 인자의 수가 미정일 경우 가변인자 사용
- printf() 함수의 매개변수
- 매개변수로 아무것도 넘겨주지 않을 수도 있고, 혹은 여러 개를 넘겨줄 수도 있다.
- 이러한 매크로는 함수가 고정된 수의 필수 인수에 가변 수의 선택적 인수가 붙은 형식을 사용한다고 가정한다.

- 명시한 타입의 인수를 0개 이상 받을 수 있다. 
- 가변인수 메서드를 호출하면 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장해 가변 인수 메서드에 건내준다.

- 가변인자를 사용한 메소드는 오버로딩 x

다음은 입력받은 int 인수들의 합을 계산해주는 가변인수 메서드다. sum(1,2,3)은 6을, sum()은 0을 리턴한다.

#### 53-1 간단한 가변인수 활용 예
```
static int sum(int... args) {
    int sum = 0;
    for (int arg : args) // 배열에서 하나씩 꺼낸다
        sum += arg;
    return sum;
}
```
인수가 1개 이상이어야 할 때는 다음과 같이 설계할 수 있다. 아래 코드는 잘못 구현한 예로써 아래와 같은 문제를 가진다.

#### 53-2 인수가 1개 이상이어야 하는 가변인수 메서드 - 잘못 구현한 예!
```

static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
```
인수를 0개만 넣어 호출하면 런타임에 실패하며, 코드도 지저분하다.
args 유효성 검사를 명시적으로 해야 하고, min의 초기값을 Integer.MAX_VALUE로 설정하지 않고는 (더 명료한)for-each문도 사용할 수 없다.

매개변수를 2개 받도록 하면 더 간단히 구현할 수 있다.

```
static int min(int firstArgs, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
```
첫 번째로는 평범한 매개변수를 받고, 가변인수는 두 번째로 받으면 앞의 문제를 해결할 수 있다. 가변인수는 인수 갯수가 정해지지 않았을 때 아주 유용하다.

#### 성능에 민감한 상황에 사용할 수 있는 패턴
```
public void test() { }
public void test(int a1) { }
public void test(int a1, int a2) { }
public void test(int a1, int a2, int a3) { }
public void test(int a1, int a2, int a3, int… rest) { }
```
- 성능에 민감한 상황일 땐 가변인수가 문제가 될 수 있다.
	- 이유는 가변인수의 유연성 때문. 가변인수 메서드는 호출될 시 배열을 새로 하나 할당 후 초기화한다.
- 가변 인수의 유연성이 필요할 시엔 해당 메서드 호출 95%가 인수를 3개 이하로 사용한다고 가정했을 때, 인수가 0개인 것부터 4개인 것까지 총 5개의 메서드를 다중 정의한다.
- EnumSet의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다

- 가변인수 메서드 호출 과정
	1) 먼저 인수의 개수와 길이가 같은 배열을 만듦.
	2) 만들어진 인수들을 이 배열에 저장함.
	3) 배열을 가변 인수 메서드에 건내줌.
	
### 핵심 정리
- 인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변 인수가 반드시 필요하다.
- 메서드를 정의할 때 필수 매개변수는 가변 인수 앞에 두고, 가변 인수를 사용할 때는 성능 문제 고려.

[참고 링크](https://cotak.tistory.com/215)
[참고 링크](https://devfunny.tistory.com/624)
