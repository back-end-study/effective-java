# item59 라이브러리를 익히고 사용하라


## 0과 명시한 수 사이의 무작위 정수 하나를 생성하는 프로그램
무작위 정수를 하나 생성하고 싶다고 해보자. 값의 범위는 0부터 명시한 수 사이다.

흔하지만 문제가 심각한 코드
```java
static Random rnd = new Random();
static int random(int n) {
	return Math.abs(rnd.nextInt()) % n;
}
```

3가지 문제를 내포하고 있다.
1. n이 그리 크지 않은 2의 제곱수라면 얼마 지나지 않아 같은 수열이 반복된다.
2. n이 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주 반환된다. n 값이 크면 이 현상은 더 두드러진다.
```java
public static void main(String[] args) {  
    int n = 2 * (Integer.MAX_VALUE / 3);  
    int low = 0;  
    for (int i = 0; i < 1_000_000; i++)  
        if(random(n) < n/2)  
            low++;  
    System.out.println(low); 
    // random 메서드가 이상적으로 동작한다면 50만에 가까운 숫자가 출력돼야 하지만, 실제로 돌려보면 666,666에 가까운 값을 얻는다. 무작위로 생성된 수 중에서 2/3가량이 중간값보다 낮은 쪽에 쏠린 것이다. (Random.nextInt(int)를 호출하면 50만에 가까운 숫자가 나온다.)
}
```
3. 지정한 범위 '바깥'의 수가 종종 튀어나올 수 있다. rnd.nextInt()가 반환한 값을 Math.abs를 이용해 음수가 아닌 정수로 매핑하기 때문이다. nextInt()가 Integer.MIN_VALUE를 반환하면 Math.abs도 Integer.MIN_VALUE를 반환하고, 나머지 연산자(%)는 음수를 반환해버린다. (n이 2의 제곱수가 아닐 때)이 결함을 해결하려면 의사난수 생성기, 정수론, 2의 보수 계산등에 조예가 싶어야 한다.
   https://stackoverflow.com/questions/27779177/effective-java-item-47-know-and-use-your-libraries-flawed-random-integer-meth

**직접 해결할 필요는 없다. Random.nextInt(int)가 이미 해결해놨다.**

## 표준 라이브러리 사용의 이점
### 1. 코드를 작성한 전문가의 지식과 여러분보다 앞서 사용한 다른 프로그래머들의 경험을 활용할 수 있다.

자바 7부터는 Random을 더 이상 사용하지 않는게 좋다. ThreadLocalRandom으로 대체하면 대부분 잘 작동한다. Random보다 더 고품질의 무작위 수를 생성할 뿐 아니라 속도도 더 빠르다. [포크-조인 풀](https://www.baeldung.com/java-fork-join)이나 병렬 스트림에서는 SplittableRandom을 사용하라.

### 2. 핵심적인 일과 크게 관련 없는 문제를 해결하느라 시간을 허비하지 않아도 된다는 것이다.

### 3. 성능이 지속해서 개선된다.
사용자가 많고, 업계 표준 벤치마크를 사용해 성능을 확인하기 때문에 표준 라이브러리 제작자들은 더 나은 방법을 꾸준히 모색할 수밖에 없다. 자바 플랫폼 라이브러리의 많은 부분이 수 년에 걸쳐 지속해서 다시 작성되며, 때론 성능이 극적으로 개선되기도 한다.

### 4. 기능이 점점 많아진다.
라이브러리에 부족한 부분이 있다면 개발자 커뮤니티에서 이야기가 나오고 논의된 후 다음 릴리즈에 해당 기능이 추가되곤 한다.

### 5. 작성한 코드가 많은 사람에게 낯익은 코드가 된다.
자연스럽게 다른 개발자들이 더 읽기 좋고, 유지보수하기 좋고, 재활용하기 쉬운 코드가 된다.

## 자바 표준 라이브러리
메이저 릴리스마다 주목할 만한 수많은 기능이 라이브러리에 추가 된다. 자바는 메이저 릴리스마다 새로운 기능을 설명하는 웹페이지를 공시하는데, 한 번쯤 읽어볼 만하다. [JDK Release Notes](https://www.oracle.com/java/technologies/javase/jdk-relnotes-index.html)

URL의 내용을 가져오는 명령줄 애플리케이션이다.
자바 9에서 InputStream에 추가된 transferTo 메서드를 사용하면 쉽게 구현할 수 있다.
```java
try (InputStream in = new URL(args[0]).openStream()) {  
    in.transferTo(System.out);  
}
```

자바 프로그래머라면 적어도 java.lang, java.util, java.io와 그 하위 패키지들에는 익숙해져야 한다. 다른 라이브러리들은 필요할 때마다 익히기 바란다. 라이브러리는 매년 아주 빠르게 성정하고 있으니 모든 기능을 요약하는 건 무리다.

주요 라이브러리는 컬렉션 프레임워크와 스트림 라이브러리다. java.util.concurrent의 동시성 기능도 마찬가지로 알아두면 큰 도움이 된다. 이 패키지는 멀티스레드 프로그래밍 작업을 단순화해주는 고수준의 편의 기능은 물론, 능숙한 개발자가 자신만의 고수준 개념을 직접 구현할 수 있도록 도와주는 저수준 요소들을 제공한다. (item80, item81)

## 라이브러리가 필요한 기능을 충분히 제공하지 못할 경우
라이브러리를 사용하려 시도해보자. 어떤 영역의 기능을 제공하는지 살펴보고, 여러분이 원하는 기능이 아니라 판단되면 대안을 사용하자. 어떤 라이브러리든 제공하는 기능은 유한하므로 항상 빈 구멍이 있기 마련이다. 자바 표준 라이브러리에서 원하는 기능을 찾이 못하면, 그다음 선택지는 고품질의 서드파티 라이브러리가 될 것이다. 구글의 구아바 라이브러리가 대표적이다. 적합한 서드파티 라이브러리도 찾지 못했다면, 다른 선택이 없으니 직접 구현하자.

[Generating Random Numbers in Java](https://www.baeldung.com/java-generating-random-numbers)  
[New Features in Java 8](https://www.baeldung.com/java-8-new-features)