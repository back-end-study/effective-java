
# Item80 스레드보다는 실행자, 태스크, 스트림을 애용하라

## 스레드로 작업을 처리하는방식

```java
public class Test {
	public static void main(String[] args) throws IOException {
		ServerSocket socket = new ServerSocket(80);
        
		while (true) {
			Socket connection = socket.accept();
			Runnable task = () -> handle(connection);
			new Thread(task).start();
		}
	}

	private static void handle(Socket connection) {
		//..
	}
}
```
위와같이 요청이 올때마다 제한없이 스레드를 생성하게되면 생기는 문제

1. 스레드 생성, 제거비용

2. 자원 낭비 가능성 
=> 프로세서보다 많은 수의 스레드가 만들어진다면, 대기 상태에 머무르는 스레드가 많아질것이고 많은 메모리, GC부하, 스레드 경쟁등의 자원을 소모하게된다.

3. 안정성 문제
=> 플랫폼과 운영체제마다 제한되어있는 스레드의 개수가 다를수도있는데 초과한다면 OutOfMemoryError를 야기할수있다. 

> 일정 수준까지는 스레드를 추가로 만들어서 여러 요청을 처리할수있는 이점을 얻을수있지만 특정 수준을 넘으면 성능이 악영향을 주게된다.    
=> 애플리케이션이 만들어 낼수있는 **스레드의 수에 제한을 두는것이 현명한 방법**이다.


### Executor
java.util.concurrent 패키지에 있는 인터페이스로, **작업 등록과 작업 실행을 분리**하기위해 사용할수있는 표준적인방법 

![](https://velog.velcdn.com/images/rodlsdyd/post/6d719431-e969-4f37-a1fa-a8be92c493bb/image.png)


이 인터페이스를 구현한 클래스는 작업의 라이프사이클을 관리하거나, 작업 실행과정을 모니터링 하기위한기능, 통계 등도 볼수있는 기능을 갖고있다.


### ExecutorService 
스레드의 생성, 할당, 생명주기 관리, 성능향상 등을 다루는 자바 스레드풀 Executor인터페이스보다 더많은 메서드를 제공하고, 좀더 복잡하고 포괄적인 인터페이스 

### 부가기능
- 정상적으로 중지시킨다(shutdown()) 
- 특정 태스크가 완료되기를 기다린다.
- 태스크 모음중 아무것하나 혹은 모든 태스크가 완료되길 기다린다
- 실행자 서비스가 종료하기를 기다린다
- 완료된 태스크들의 결과를 차례로 받는다
- 태스크를 특정시간에 혹은 주기적으로 실행하게한다


둘이상의 스레드가 큐를 처리하게 하고싶다면 다른종류의 실행자 서비스를 생성하면된다.    

우리에게 필요한 실행자 대부분은 java.util.concurrent.Executors의 정적팩터리들을 이용해 생성할수있을것이다.


### Executors
헬퍼 클래스

![](https://velog.velcdn.com/images/rodlsdyd/post/f6944a6a-36e7-4399-afeb-424c4a3dca68/image.png)

고정된 수의 스레드와 무제한 큐로 이뤄진 스레드풀   
처리할 작업이 등록되면 그에따라 실제 스레드를 하나씩 생성한다. (최대 개수는 정해져있으며, 꽉차면 더이상 생성하지 않고 유지한다)


```java
public class Test {
	public static void main(String[] args) throws IOException {
		ServerSocket socket = new ServerSocket(80);
        
		while (true) {
			Socket connection = socket.accept();
			Runnable task = () -> handle(connection);
			// 변경부분
			Executors.newFixedThreadPool(100).execute(task);
		}
	}
}
```


> p.429
무거운 프로덕션 서버에서는 스레드 개수를 고정한 Executors.newFixedThreadPool을 선택하거나 완전히 통제할수있는 ThreadPoolExecutor를 직접 사용하는편이 훨씬낫다.

![](https://velog.velcdn.com/images/rodlsdyd/post/6a55d045-6906-470d-85fd-e024e8863821/image.png)   

작업 가로채기 알고리즘에 기반한 스레드풀 

![](https://velog.velcdn.com/images/rodlsdyd/post/40c3b93c-bfc0-43d3-8498-a2303f92099a/image.png)   

무제한 큐로 스레드 하나만 관리하는 스레드풀. 한번에 한작업만 실행한다

![](https://velog.velcdn.com/images/rodlsdyd/post/5627bb5c-f596-4a02-93cb-e65e7e86de2a/image.png)   

새 스레드를 생성하고 필요에 따라 유휴스레드를 제거하는 스레드풀(스레드 수의 제한을 두지않음)

> p.429 
작은 프로그램이나 가벼운서버라면 좋은 선택이다   
그러나, 무거운 프로덕션서버에서 사용한다면  **사용가능한 스레드가 없으면 새로 하나를 생성**하기때문에 CPU이용률이 100%로 치닫고 스레드를 계속 생성하게 될것이다.

### 왜 실행자(Executor)를 써야하나?  - 작업 등록과 실행을 분리

- 어떤 순서로 작업을 실행할지(FIFO, LIFO ...) , 몇개의작업을 동시에 병렬로 실행할지, 최대 몇개까지 큐에서 대기할수있게 할건지...     
등의 실행이 애플리케이션 실제상황에 따라 달라질수있는데 이것을 분리시켜준다.

```java
Executors.newFixedThreadPool(100).execute(task);
```
> 위와 같은 예시에서 **태스크 수행정책 선택(스레드 풀, 부가적인 실행전략), 유연한 변경이 가능**하다


 - 자바7부턴 실행자 프레임워크는 포크조인 태스크를 지원하도록 확장되었다.    
=> 실행자 프레임워크는 실행 메커니즘의 선택지가 점점 늘어나고 , 유연해진다.



### ThreadPoolExecutor (ExecutorService 의 하위클래스)

> 평범하지 않은 실행자를 원한다면 ThreadPoolExecutor 클래스를 직접 사용해도 된다. 

![](https://velog.velcdn.com/images/rodlsdyd/post/6adff317-af9b-40ad-adb0-1ec4b6002b96/image.png)

풀에 유지할 스레드의 개수, 최대 스레드 수 , 풀에서 제거하기까지 유지할 시간 설정 등 조금서 세세하게 최적화할수있는 클래스 



### 결론

- 작업큐를 손수 만드는일은 삼가고, 스레드를 직접 다루는것도 삼가야한다

- 핵심은 실행자 프레임워크가 작업수행을 담당해준다는것이다.(컬렉션 프레임워크가 데이터 모음을 담당하듯)

- 스레드를 직접다루면 작업단위, 수행메커니즘 역할을 모두 수행해야하지만, 실행자 프레임워크에서는 작업단위와 실행 메커니즘이 분리된다. 

  - Executor : execute() 메서드 하나만을 제공하는 작업등록, 작업 실행을 분리하기위한 표준 인터페이스
  
  - ExecutorService : Executor를 구현한 더많은 메서드를 제공하고, 좀더 복잡하고 포괄적인 인터페이스
  
  - Executors : 여러 스레드풀 실행전략을 선택을 도와주는 헬퍼클래스
  
  - ThreadPoolExecutor : 풀에 유지할 스레드의 개수, 최대 스레드 수 , 풀에서 제거하기까지 유지할 시간 설정 등 조금서 세세하게 최적화할수있는 클래스
