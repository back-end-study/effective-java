# item83 지연 초기화는 신중히 사용하라

## 지연 초기화(lazy initialization)
초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법으로 초기화 비용은 줄지만 지연 초기화하는 필드에 접근하는 비용은 커진다.

## 지연 초기화가 필요할 때
해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화하는 비용이 크다면 지연 초기화가 제 역할을 해줄 것이다.

## 멀티스레드 환경에서의 지연 초기화
지연 초기화는 필드를 둘 이상의 스레드가 공유한다면 어떤 형태로든 반드시 동기화해야 한다.
대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.

### 인스턴스 필드를 초기화하는 일반적인 방법
```java
private final FieldType field = computeFieldValue();
```
final 한정자를 사용했음에 주목하자(아이템17)

### 지연 초기화가 초기화 순환성을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용 하자.
```java
private FieldType field;  
  
private synchronized FieldType getField() {  
    if(field == null)  
        field = computeFieldValue();  
    return field;  
}
```

초기화 순환성
```java
public class Test {  
    private final Test2 test2 = computeTest2();  
  
    private Test2 computeTest2() {  
        return new Test2();  
    }  
  
    public static void main(String[] args) {  
        Test test = new Test();
//    Exception in thread "main" java.lang.StackOverflowError  
//        at Test2.computeTest(Test2.java:7)  
//        at Test2.<init>(Test2.java:4)  
//        at Test.computeTest2(Test.java:7)  
//        at Test.<init>(Test.java:4)  
//        at Test2.computeTest(Test2.java:7)  
//        at Test2.<init>(Test2.java:4)  
//        at Test.computeTest2(Test.java:7)  
//        at Test.<init>(Test.java:4)  
    }  
}
public class Test2 {  
    private final Test test = computeTest();  
  
    private Test computeTest() {  
        return new Test();  
    }
}
```

두 관용구는 정적 필드에도 똑같이 적용된다. 물론 필드와 접근자 메서드 선언에 static 한정자를 추가해야 한다.

### 지연 초기화 홀더 클래스
성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스 관용구를 사용하자. 클래스는 클래스가 처음 쓰일 때 비로소 초기화된다는 특성을 이용한 관용구다.[JLS, 12.4.1](https://docs.oracle.com/javase/specs/jls/se17/html/jls-12.html#jls-12.4.1)

```java
private static class FieldHolder {  
    static final FieldType field = computeFieldValue();  
}  
  
private static FieldType getField() {  
    return FieldHolder.field;  
}
```
getField가 처음 호출되는 순간 FieldHolder.field가 처음 읽히면서, FieldHolder 클래스 초기화를 촉발한다. getField 메서드가 필드에 접근하면서 동기화를 전혀 하지 않으니 성능이 느려질 거리가 전혀 없다는 것이다. 일반적인 VM은 오직 클래스를 초기화할 때만 필드 접근을 동기화할 것이다. 클래스 초기화가 끝난 후에는 VM이 동기화 코드를 제거하여, 그다음부터는 아무런 검사나 동기화 없이 필드에 접근하게 된다.

### 이중검사
성능 때문에 인스턴스 필드를 지연 초기화해야 하다면 이중검사 관용구를 사용하자. 필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 반드시 [volatile](https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.3.1.4)로 선언해야 한다.
```java
private volatile FieldType field;

private FieldType getField() {  
    FieldType result = field;  
    if(result != null)  
        return result;  
    synchronized (this) {  
        if(field == null)  
            field = computeFieldValue();  
        return field;  
    }  
}
```
result는 필드가 이미 초기화된 상황에서는 그 필드를 딱 한 번만 읽도록 보장하는 역할을 한다. 반드시 필요하지는 않지만 성능을 높여주고, 저수준 동시성 프로그래밍에 표준적으로 적용되는 더 우아한 방법이다.
이중 검사를 정적 필드에도 적용할 수 있지만 굳이 그럴 이유는 없다. 이보다는 지연 초기화 홀더 클래스 방식이 더 낫다.

### 단일 검사
반복해서 초기화해도 상관없는 경우라면 이중검사에서 두 번째 검사를 생략할 수 있다. 이 변종의 이름은 단일 검사 관용구가 된다.
```java
private volatile FieldType field;  
  
private FieldType getField() {  
	FieldType result = field;  
	if (result == null)  
		field = result = computeFieldValue();  
	return result;  
}
```

모든 초기화 기법은 기본 타입 필드와 객체 참조 필드 모두에 적용할 수 있다.

### 짜릿한 단일검사(racy single-check)
모든 스레드가 필드의 값을 다시 계산해도 상관없고 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 단일 검사의 필드 선언에서 volatile 한정자를 없애도 된다. 이 변종은 짜릿한 단일검사(racy single-check) 관용구라 불린다. 이 관용구는 어떤 환경에서는 필드 접근 속도를 높여주지만, 초기화가 스레드당 최대 한 번 더 이뤄질 수 있다. 아주 이례적인 기법으로, 보통은 거의 쓰지 않는다.

[[Baeldung] java volatile](https://www.baeldung.com/java-volatile)
[[Baeldung] java final](https://www.baeldung.com/java-final)
[Non-atomic Treatment of `double` and `long`](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.7)