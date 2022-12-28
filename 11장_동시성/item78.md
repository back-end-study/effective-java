## 아이템 78 공유 중인 가변 데이터는 동기화해 사용해라

synchronized 키워드는 해당 메서드나 블록을 한 번에 한 스레드씩 수행하도록 보장한다.

동기화는 일관된 상태를 가진 객체에 접근하는 메서드가 그 객체에 락을 걸고,

락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정하여 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다.

즉, 동기화를 제대로 사용한다면 어떤 객체도 일관되지 않는 상황을 볼 수 없을 것이다.

```java
public class StopThread {

    private static boolean stopRequested;
    
    public static void main(String[] args) {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) {
                i++;
            }
        });
        backgroukndTrehad.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

위 코드는 `stopRequested` 를 `false`로 시작 하고 다른 스레드가 `true` 로 변경 하면 반복문을 빠져나오는 코드이다.

하지만 슬프게도 위 코드는 여원히 수행 되게 된다.

그 이유는 동기화 때문이다.

메인스레드가 수정한 값을 `backgroukndTrehad` 가 언제쯤 보게 될지 보증 할 수 없다.

동기화에 빠지면 가상 머신은 다음과 같이 끌어올리기 (hoisting)라는 최적화 기법을 수행하게 된다.

```java
//원래 코드
while (!stopRequested) i++;
```


```java
// 최적화한 코드
if (!stopRequested) {
    while(true) i++;
}
```

이 결과 프로그램은 응답 불가 상태가 되어 더이상 진전이 없다. 이 문제를 해결 하기 위해서는 

`stopRequested` 필드를 동기화 해 접근하면 해결이 가능하다

```java
public class StopThread{
	private static boolean stopRequested;
    
    private static synchronized void requestedStop(){
    	stopRequested = true;
    }
    
    private static synchronized boolean stopRequested(){
    	return stopRequested;
    }
    
    public static void main(String[] args) throws InterruptedException{
    	Thread backgroundThread = new Thread(() -> {
        	int i=0;
            while(!stopRequested()){
            	i++;
            }
        })
        backgroundThread.start();
        
        TimeUnit.SECONDS.sleep(1);
        requestedStop();
    }
}
```

쓰기 메서드와 읽기 메서드 모두 동기화 했음을 주목 해야한다. 

쓰기만 동기화 하면 안되고, 쓰기와 읽기 모두 동기화하지 않으면 동작을 보장하지 않는다.


## volatile
`volatile` 한정자는 배타적 수행과는 상관없지만 항상 최근에 기록된 값을 읽는 것을 보장해주는 역할을 한다. 

따라서 위와 같은 상황에서 사용할 수 있는 한정자다. 

`volatile`은 매번 동기화하는 비용이 그렇게 크지 않고, 속도가 더 빠른 대안이 된다.


```java
// volatile를 적용한 예시
public class StopThread{
	private static volatile boolean stopRequested;
    
    public static void main(String[] args) {
    	Thread backgroundThread = new Thread(() -> {
        	int i=0;
            while(!stopThread){
            	i++;
            }
        })
        backgroundThread.start();
        
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

하지만 이런 `volatile`는 조심해서 사용해야한다
#### volatile 주의사항
```java
private static volatile int nextSerialNumber = 0;
public static int generateSerialNumber(){
	return nextSerialNumber++;
}
```
이 메서드는 매번 고유한 값을 반환할 의도로 만들어졌다. 

nextSerialNumber 라는 값은 원자적으로 접근할 수 있고, 

어떤 값이든 허용하기 때문에 동기화없이도 불변식을 보호할 수 있어 보인다. 

하지만 동기화 없이 올바로 작동하지 않는다.

증가 연산자(++)는 코드상으로 하나지만 실제로는 nextSerialNumber 

1) 필드에 값을 읽을 때 한번
2) 그 다음 새로운 값을 저장할 때 한번

총 두번 접근하게 된다.

만약 다른 스레드가 이 두 접근 사이를 비집고 들어와서 값을 읽어간다면  두 스레드는 동일한 값을 읽어가게 된다. 

즉 오류가 발생하게 되고 이런 문제를 안전 실패(safety failure)라고 한다.

이는 `generateSerialNumber()` 메서드에 `synchronized`를 붙임으로써 해결할 수 있다. 

동시에 호출해도 서로 간섭하지 않아 이전 호출이 변경한 값을 읽게 된다. 

다만 메서드에 `synchronized`를 붙였다면 `volatile` 한정자는 제거해야 한다.



#### AtomicLong

`volatile`는 동기화의 두 효과 중 통신만 지원하지만 이 패키지는 배타적 실행 가지 지원을 한다.
그뿐만 아니라 성능도 더 우수하다.

java.util.concurrent.atomic 을 이용한 락-프리 동기화
```java
private static final AtomicLong nextSerialNum = new AtomicLong();

    public static long generateSerialNumber() {
        return nextSerialNum.getAndIncrement();
    }
```
동기화로 인한 문제를 피하는 최고의 방법은 가변 데이터를 공유하지 않는 것이다. 

가변 데이터는 단일 스레드에서만 쓰는것이 좋다.

한 스레드가 데이터를 다 수정하고 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다. 

그러면 그 객체를 다시 수정할 일이 생기기 전까지는 

다른 스레드들은 동기화없이 자유롭게 값을 읽어갈 수 있기 때문이다. 

이런 객체를 불변 이라 하고 다른 스레드에 이런 객체를 건네는 행위를 안전 발행(safe publication)이라고 한다.
