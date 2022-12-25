# [Item82] 스레드 안전성 수준을 문서화하라

배정: 김세용
상태: Effective Java 3E

스레드 안전성(Thread-safe) 수준을 문서화 하지 않으면 클래스 사용자는 나름의 가정을 해야만 한다.

만약 그 가정이 틀리면 클라이언트 프로그램은 동기화를 충분히 하지 못하거나 지나치게 한 상태일 것이며 이 경우 모두 심각한 오류로 이어질 수 있다.

### synchronized 한정자를 선언했는데?

소스 코드에 synchronized 한정자를 사용했다고 하더라도 이는 구현상에서만 쓰일 뿐 JavaDoc 기본 옵션에서 생성한 API 문서에는 드러나지 않는다

**즉, synchronized 한정자를 선언하였다고 해서 Thread-safe함을 문서화 한 것이 아니다!**

### synchronized 한정자를 사용했다고해서 그 클래스가 100% Thread-safe한가?

스레드 안전성에도 수준이 나뉜다. **멀티스레드 환경에서도 API를 안전하게 사용하게 하려면 클래스가 지원하는 스레드 안전성 수준을 전확히 명시해야 한다.**

다음은 스레드 안전성이 높은 순으로 나열한것이다.

(**스레드 안전성 어노테이션과 유사한 것을 `≈` 통해 표현)**

- **불변(immutable)  ≈**  @Immutable
: 이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화가 필요 없다 (ex. String, Long, BigInteger)
- **무조건적 스레드 안전(unconditionally thread-safe)  ≈**  @ThreadSafe
: 우리가 흔히 얘기하는 thread-safe한 클래스로 별도의 동기화 없이 동시에 사용해도 안전하다 (ex. AtomicLong, ConcurrentHashMap)

![ConcurrentHashMap의 클래스 상단 주석](https://user-images.githubusercontent.com/68587990/209437057-af00f800-8ed4-4841-a858-93a2fcee2d10.png)

ConcurrentHashMap의 클래스 상단 주석

![ConcurrentLinkedHashMap의 @ThreadSafe 어노테이션](https://user-images.githubusercontent.com/68587990/209437071-c1e854c8-f0c8-4720-9f6c-78d61d79ea59.png)

ConcurrentLinkedHashMap의 @ThreadSafe 어노테이션

- **조건부 스레드 안전(conditionally thread-safe)  ≈**  @ThreadSafe
: 전반적으로는 thread-safe 하지만 일부 메서드가 동기화가 필요한 경우 (ex. Collections.synchronized 래퍼 메서드가 반환한 컬렉션들)

![Collections.java의 synchronizedSet을 반환하는 synchronizedSet 메서드의 경우 외부 동기화의 예시를 주석으로 남기고 있다](https://user-images.githubusercontent.com/68587990/209437403-0c454647-e050-4f8f-99e3-5a56ba47e642.png)

Collections.java의 synchronizedSet을 반환하는 synchronizedSet 메서드의 경우 외부 동기화의 예시를 주석으로 남기고 있다

- **스레드 안전하지 않음(not thread-safe)  ≈**  @NotThreadSafe
: 동시에 사용하려면 각각의 메서드 호출을 클라이언트가 외부 동기화 메커니즘으로 감싸야 하는 경우 (ex. ArrayList, HashMap 등)

![클래스 상단 주석으로 not synchronized라고 명시하고 있는 ArrayList 클래스](https://user-images.githubusercontent.com/68587990/209437472-769c858a-b65b-43d3-a6b9-e43a00dc2010.png)

클래스 상단 주석으로 not synchronized라고 명시하고 있는 ArrayList 클래스

![@NotThreadSafe 어노테이션이 명시된 LinkedDeque 클래스](https://user-images.githubusercontent.com/68587990/209437474-00ac7acc-16e6-4090-a6e5-f08ce917e96f.png)

@NotThreadSafe 어노테이션이 명시된 LinkedDeque 클래스

- **스레드 적대적(thread-hostile)  ≈**  @NotThreadSafe (@NotThreadSafe으로 명시해도 되지않을까?)
: 외부 동기화 매커니즘을 사용하더라도 안전하지 않은 경우. 이 경우 문제를 고쳐 재배포하거나 deprecated API로 지정할 필요가 있다

클래스의 스레드 안전성은

- 보통 클래스의 문서화 주석에 기재한다
- 독특한 특성의 메서드라면 해당 메서드 주석으로 기재한다
- 열거 타입은 굳이 불변(혹은 @Immutable)이라고 쓰지 않아도 된다
- 반환 타입만으로는 명확히 알 수 없는 정적 팩터리라면 자신이 반환하는 객체의 스레드 안정성을 반드시 문서화 해야한다(Collections.synchronized 래퍼 메서드 처럼)

### 비공개 락 객체 관용구를 사용하자

클래스가 외부에서 사용할 수 있는 락(public lock instance)을 제공하면 클라이언트 측에서 유연하게 클래스 메서드를 사용할 순 있지만, 

ConcurrentHashMap과 같이 내부에서 복잡하게 고성능 동시성 제어 메커니즘을 사용하고 있는 경우 혼용해서 쓸 수 없다. (가능하더라도 매우매우 복잡하고 번거로울듯 하다)

또한, 공개된 락을 오래 쥐고 놓지 않는 서비스 거부 공격(deial-of-service attack)을 수행할 수도 있다

따라서 synchronized 메서드 대신 비공개 락 객체를 사용하자

```java
// 비공개 락 객체 관용구
private final Object lock = new Object();

public void foo() {
    synchronized(lock) {
        // ...
    }
}
```

> **우연히라도 락 객체가 교체되는 일을 예방하기 위해서 항상 lock 필드는 final로 선언하자**
> 

### 비공개 락 객체는 상속용으로 설계한 클래스에 적합하다

상속용 클래스에서 자신의 인스턴스를 락으로 사용한다면, ‘서로가 서로를 훼방놓는’ 상태에 빠져 의도치 않은 결과를 낳게 된다

```java
/**
 * Bloch05, puzzle 77
 */
@Slf4j
public class Worker extends Thread {
    private volatile boolean quittingTime = false;

    public void run() {
        while (!quittingTime) {
            pretendToWork();
        }
        log.info("end to work!");
    }

    private void pretendToWork() {
        try {
            log.info("working!");
            Thread.sleep(300);
        } catch (InterruptedException e) {
        }
    }

    synchronized void quit() throws InterruptedException {
        log.info("start quit()");
        quittingTime = true;
        join();
        log.info("end quit()");
    }

    synchronized void keepWorking() {
        log.info("start keepWorking()");
        quittingTime = false;
        log.info("end keepWorking()");
    }

    public static void main(String[] args) throws InterruptedException {
        final Worker worker = new Worker();
        worker.start();

        Timer t = new Timer(true); // Daemon thread
        t.schedule(new TimerTask() {
            public void run() {
                worker.keepWorking();
            }
        }, 500);

        Thread.sleep(400);
        worker.quit();
    }
}
```

- **예상 실행 결과**

![https://user-images.githubusercontent.com/68587990/209461487-4cbb3532-b010-4e7d-9dac-56dfd626dc00.png](https://user-images.githubusercontent.com/68587990/209461487-4cbb3532-b010-4e7d-9dac-56dfd626dc00.png)

- **실제 실행 결과**

![https://user-images.githubusercontent.com/68587990/209461490-a784f3c6-8804-465b-a358-8454bccd8c61.png](https://user-images.githubusercontent.com/68587990/209461490-a784f3c6-8804-465b-a358-8454bccd8c61.png)

```java
17:30:46.777 [Thread-0] INFO me.whiteship.chapter11.item82.Worker - working!
17:30:47.084 [Thread-0] INFO me.whiteship.chapter11.item82.Worker - working!
17:30:47.181 [main] INFO me.whiteship.chapter11.item82.Worker - start quit()
17:30:47.278 [Timer-0] INFO me.whiteship.chapter11.item82.Worker - start keepWorking()
17:30:47.278 [Timer-0] INFO me.whiteship.chapter11.item82.Worker - end keepWorking()
17:30:47.389 [Thread-0] INFO me.whiteship.chapter11.item82.Worker - working!
17:30:47.694 [Thread-0] INFO me.whiteship.chapter11.item82.Worker - working!
17:30:48.000 [Thread-0] INFO me.whiteship.chapter11.item82.Worker - working!
17:30:48.301 [Thread-0] INFO me.whiteship.chapter11.item82.Worker - working!
17:30:48.604 [Thread-0] INFO me.whiteship.chapter11.item82.Worker - working!
17:30:48.910 [Thread-0] INFO me.whiteship.chapter11.item82.Worker - working!
17:30:49.215 [Thread-0] INFO me.whiteship.chapter11.item82.Worker - working!
17:30:49.521 [Thread-0] INFO me.whiteship.chapter11.item82.Worker - working!
17:30:49.826 [Thread-0] INFO me.whiteship.chapter11.item82.Worker - working!
                                   ...
```

**따라서 이러한 경우를 막기 위해서 별도의 비공개 락 인스턴스를 만들어, 명시적 잠금을 사용해야한다**

> 명시적 잠금을 사용하는 BetterWorker에 대해서 알고 싶으면  [Bloach05 Puzzle 77 에 대해서 설명한 블로그 글](https://www.javaspecialists.eu/archive/Issue144-Book-Review-Java-Puzzlers.html) 을 참고 하면 된다. 위 Worker 예시도 해당 블로그를 참고하여 조금 변형하였다
> 

### 핵심 정리

- 모든 클래스가 자신의 스레드 안전성 정보를 명확히 문서화 해야한다
- 문서화를 할 땐 명확한 언어로 주석 설명을 하거나, 어노테이션을 사용하자
- synchronized 한정자로는 스레드 안전성 문서화와는 관련이 없다
- 조건부 스레드 안전 클래스는 메소드 주석을 통해 어떤 순서로 호출해야하는지, 어떻게 락을 얻어야하는지 명확히 설명하자
- 무조건적 스레드 안전 클래스를 작성할 때는 synchronized 한정자가 아닌 비공개 락 객체를 사용하자(명시적 잠금)