# 아이템81. wait와 notify보다는 동시성 유틸리티를 애용하라

wait과 notify를 올바르게 사용하는것은 아주 까다롭다.

고수준 동시성 유틸리티를 사용하자 (ex: `java.util.concurrent`)

## 1. java.util.concurrent 패키지

`java.util.concurrent`의 고수준 유틸리티는 세 범주로 나눌 수 있다.

1. 실행자 프레임워크(executor framework) - *아이템80*
2. 동시성 컬렉션(concurrent collection)
3. 동기화 장치(synchronizer)

## 2. 동시성 컬렉션

- List, Queue, Map과 같은 표준 컬렉션 인터페이스에 동시성을 가미한 고성능 컬렉션이다.
- 내부적으로 동기화를 각자의 내부에서 수행한다. - *아이템79*

**따라서, 동시성 컬렉션에서 동시성을 무력화하는건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.**

- 동시성을 무력화하지 못하기에 여러 메서드가 원자적으로 묶여 호출하지는 못하고, 여러 기본 동작을 하나의 원자적 동작으로 묶는 ‘**상태 의존적 수정**’ 메서드들이 추가되었다.
    - 예시) `Map`의 `putIfAbsent(key, value)` 메서드

        ```java
        // Map의 putIfAbsent 메서드
        default V putIfAbsent(K key, V value) {
            V v = get(key);
            if (v == null) { // key에 매핑된 값이 없을 때만 put
                v = put(key, value);
            }
        
            return v;
        }
        ```

    - *이 메서드 덕에 스레드 안전한 정규화 맵(canonicalization map)을 쉽게 구현*
    - 자바 8에서는 일반 컬렉션 인터페이스에도 디폴트 메서드 형태로 추가 됨

### 2-1. String.intern() 구현하기

intern() 메서드는 String 비교에서 == 연산자의 사용이 필요한 경우 활용할 수 있다.

```java
String a = "apple";
String b = new String("apple");
String c = b.intern()

System.out.println(a == b); // false
System.out.prrintln(a == c); // true
```

- a와 b의 비교는 앞서 예시처럼 참조하는 위치가 다르므로 false를 반환
- b와 c는 intern()을 통해 생성하여 ==을 true로 반환
- **intern() 메서드는 String pool에서 리터럴 문자열이 이미 존재하는지 체크하고 존재하면 해당 문자열을 반환하고, 아니면 리터럴을 String pool에 넣어줌**

**2-1-1. ConcurrentMap으로 구현한 동시성 정규화 맵 - 최적은 아니다**

```java
private static final ConcurrentMap<String, String> map = new ConcurrentHashMap<>();

public static String intern(String s) {
  String previousValue = map.putIfAbsent(s, s);
  return previousValue == null ? s : previousValue;
}
```

- `ConcurrentHashMap` 은 get 같은 검색 기능에 최적화되었다.
- 따라서 get을 먼저 호출하여 필요할 때만 `putIfAbsent`를 호출하면 더 빠르다.

**2-1-2. ConcurrentMap으로 구현한 동시성 정규화 맵 - 더 빠르다**

```java
public static String intern(String s) {
  String result = map.get(s);
  if (result == null) {
    result = map.putIfAbsent(s, s);
    if (result == null)
      result = s; 
  }
  return result;
}
```

- `ConcurrentHashMap`은 동시성이 뛰어나며 속도도 무척 빠르다.
- ~~저자의 로컬에서 String.intern보다 6배 빠르다고 함~~
    - `String.intern()`에는 오래 실행되는 프로그램에서 메모리 누수를 방지하는 기술도 들어가 있음을 감안하자.

**2-1-3. 동시성 컬렉션 vs 동기화한 컬렉션**

- 동시성 컬렉션은 동기화한 컬렉션을 옛날 레거시로 만들어 버렸다.
- 대표적으로 이제는 `Collections.synchronizedMap` 보다는 `ConcurrentHashMap` 을 사용하자
- 동기화된 Map을 동시성 Map으로 교체하는 것만으로 동시성 애플리케이션의 성능은 극적으로 개선 된다.

> 동기화 컬렉션보다 안전과 속도 모두 개선되었기에 동시성 컬렉션을 사용하자

## 3. 동기화 장치

> 스레드가 다른 스레드를 기다릴 수 있게 함

- 컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때까지 기다리도록 확장 됨
- 예를들어, Queue를 확장한 `BlockingQueue`에 추가된 `take` 메서드가 있음
    - 큐의 첫 번째 원소를 꺼냄
    - 만약 큐가 비었다면 새로운 원소가 추가될 때까지 기다린다.
- 이런 특성 덕에 `BlockingQueue`는 작업 큐(생산자-소비자 큐)로 쓰기 적당하고 `ThreadPoolExecutor`나 실행자 서비스*(아이템 80)* 구현체에서 `BlockingQueue`를 사용한다.

그 밖에 또 자주 사용되는 동기화 장치로 다음과 같은 장치들이 있다.

### 3-1. 동기화 장치의 종류

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해줌

- CountDownLatch
- Semaphore
- CyclicBarrier
- Exchanger
- Phaser

**자주 쓰이는 CountDownLatch**

- 일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.
- CountDownLatch의 유일한 생성자는 int 값을 받으며, 이 값이 래치의 countDown 메서드를 몇 번 호출해야 대기 중인 스레드를 깨우는지 결정한다.

```java
// 동시 실행 시간을 재는 간단한 프레임워크
public static long time(Executor executor, int concurrency, Runnable action)
            throws InterruptedException {

    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(concurrency);

    for (int i = 0; i < concurrency; i++) {
        executor.execute(() -> {
            ready.countDown(); // 준비 완료
            try {
                start.await(); // 모든 작업자 스레드가 준비되길 기다림
                action.run();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                done.countDown(); // 타이머에게 작업 마침을 알림
            }
        });
    }
    
    ready.await(); // 모든 작업자가 준비되길 기다림
    long startNanos = System.nanoTime();
    start.countDown(); // 작업자 깨움
    done.await(); // 모든 작업자가 일을 마치기를 기다림
    return System.nanoTime() - startNanos;
}
```

**코드 동작 설명**

- 총 3개의 래치로 구성되어있다.
  - ready : 작업자 스레드들이 준비가 완료됐음을 타이머 스레드에 통지할 때 사용
  - start : 통지를 끝낸 작업자 스레드들이 start가 열리기를 기다림 
  - done : 타이머에게 작업을 마쳤음을 알릴 때 사용 

1. `ready.countDown`으로 각 작업자는 준비 완료를 알림
2. 모든 작업자가 준비 완료되면 `ready.await` 대기 종료
3. 타이머 스레드가 시작 시각을 기록(`startNanos 변수`)
4. `start.countDown`을 호출하여 기다리던 작업자 스레드들을 깨움
5. 마지막 남은 작업자 스레드가 종료 `start.await` 호출 - `start.countDown` 대기 종료
6. `done.countDown`호출하여 `done.await()`대기 종료 
7. 타이머 스레드는 `done` 래치 깨어난 직후 종료 시각을 기록

**스레드 기아 교착상태(Thread starvation deadlock) 발생 가능**

- 위와 같은 메서드에 넘겨진 실행자는 concurrency 매개변수로 동시성 수준 만큼의 스레드를 생성할 수 있어야 한다.
- 그렇지 못하면 스레드 생성을 하지 못하고 이 메서드는 끝나지 않는다.
- 이런 상태를`스레드 기아 교착상태`라 한다.

> **참고**  
> 시간 간격을 잴 때는 System.nanoTime을 사용하자.  
> System.currentTimeMillis보다 더 정확하고 정밀하며, 시스템의 실시간 시계의 시간 보정에 영향을 받지 않는다.

## 4. 정리

- 코드를 새로 작성한다면 `wait`과 `notify`를 쓸 이유가 거의(어쩌면 전혀) 없다.
- 하지만 어쩔 수 없이 레거시 코드를 다뤄야 할 때도 있을 것이다.
- `wait`을 사용하는 표준 방법은 아래와 같다.
    - **`wait` 메서드를 사용할 때는 반드시 반복문 안에서 사용하도록 해야한다. 반복문으로 실행가능 조건을 검사함으로써 불필요한 `wait` 호출을 막을 수 있다.**
    - 만약 반복문 밖에 `wait`이 존재한다면 `wait`한 스레드를 언제 다시 `notify` 할지 보장할 수 없다.
    - 대기 후에 조건을 검사하여 조건이 충족되지 않았다면 다시 대기하게 하는 것은 안전실패를 방지하는 조치다. 만약 조건이 충족되지 않았는데 스레드가 동작을 이어가면 락이 보호하는 불변식을 깨뜨릴 위험이 있다.
        1. `wait` 이후에 깨어나는 사이 다른 스레드가 락을 얻고 동기화 블럭 안의 상태를 변경할 수도 있음
        2. 조건이 만족되지 않았는데 악의적인 스레드가 `notifyAll`을 호출할 수도 있음
        3. 대기중인 스레드가 `notify` 없이도 깨어나는 경우가 드물게 있을 수 있음

```java
synchronized (obj) {
	while(조건이 충족되지 않았다) {
		obj.wait(); // (락을 놓고, 깨어나면 다시 잡는다.)
	}
	... // 조건이 충족됐을 때의 동작을 수행한다.
}
```

- `notify`와 `notifyAll` 두개 중 어떤 것을 사용할지에 대한 문제도 존재하지만 `notifyAll`을 사용하는 것이 더 합리적이고 안전하다.
    - 깨어나야 하는 모든 스레드가 깨어남을 보장하기 때문에 항상 정확한 결과를 얻을 것이다.
    - 다른 스레드까지 깨어날 수 있지만 우리의 프로그램 정확성에는 영향을 주지 않을 것이다. 깨어난 스레드들은 기다리던 조건이 충족되었는지 확인하여 충족되지 않았다면 다시 대기상태로 돌아갈 것이기 때문이다.

- 만약 이들을 사용하는 레거시 코드를 유지보수를 해야한다면 `wait`은 항상 표준 관용구에 따라 `while`(반복문)안에서 호출하도록 하자.
- 일반적으로 `notify`보다는 `notifyAll`을 사용해야 한다. 혹시라도 `notify`를 사용한다면 응답 불가 상태에 빠지지 않도록 각별히 주의하자.

> 새로운 코드라면 언제나 wait와 notify가 아닌 동시성 유틸리티를 사용하자