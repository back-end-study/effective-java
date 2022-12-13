## 아이템 62 다른 타입이 적절하다면 문자열 사용을 피하라

문자열은 텍스트를 표현하기 위해 설계되었다. 하지만 본래 의도와는 다르게 사용되는 경향이 있다.

### 1. 문자열로 다른값 타입을 대신하는 경우
예륻들어 입력 받을 데이터가 

수치형이라면 int, float 와 같은 형으로 받는것이 좋으며

예/아니오 라고 한다면 열거타입 이나 true/false로 표현하는것이 좋다.

### 2. 문자열은 열거타입을 대신하기에 적합하지 않다. 
열거타입을 사용하게 되면 가독성과 안정성 부분이 뛰어나다. 

### 3. 문자열을 혼합타입을 대신하기에 적합하지 않다.
```java
// 혼합 타입을 문자열로 처리한 부적절한 예
String CompoundKey = className + "#" + i.next()
```
위의 에제는 각 요소를 구분자 `#` 을 이용하여 문자열을 파싱 하고있다.

+ 문자열을 파싱해야하기 때문에 매우 귀찮고
+ 오류도 많을수도 있고
+ `equals` `toString` `comparteTo` 와 같은 메서드를 사용할 수 없다.


```java
// 만약 전용 클래스로 만든다면 이렇게 private 정적 멤버 클래스로 선언한다.
class Compound {
 
    private CompoundKey compoundKey;

    private static class CompoundKey {
        private String className;
        private String delimiter;
        private int index;

        public CompoundKey(String className, String delimiter, int index) {
            this.className = className;
            this.delimiter = delimiter;
            this.index = index;
        }
    }
}
```

### 4. 문자열로 권한을 표현하는 경우

```java
// 잘못된 예 문자열을 사용해 권한을 부여함
public class ThreadLocal {
    private ThreadLocal() {} //객체 생성 불가
    
    // 현 스레드의 값을 키로 구분해 저장한다.
    public static void set(String key, Object value);
    
    // 현 스레드의 값을 반환한다.
    public static Object get(String key);
}
```
위 스레드는 구분용 키로 문자열을 받는다. 이방식의 문제점은

스레드 구분용 문자열 키가 전역에서 공유가 되므로, 운없이 키가 겹치게 된다면 변수를 공유하게 되는 문제점이 있다.


#### 해결방안 - version 1
```java
public class ThreadLocal {
    private ThreadLocal() {} // 객체 생성 불가

    public static class Key { // (권한)
        key() {}
    }

    // 위조 불가능한 고유 키를 생성한다.
    public static Key getKey() {
		    return new Key();
    }

    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```
위와같이 문자열로 구분하는 것이 아니라, key class 로 권한을 부여하여 키가 공유 되는 문제를 해결하였다.
그러나 아직 문제가 더 남아있다.


#### 해결방안 - version 2 (리팩터링 하여 Key를 ThreadLocal로 변경)
```java
public final class ThreadLocal {
    public ThreadLocal();
    public void set(Object value);
    public Object get();
}
```

set() 과 get() 메서드는 더 이상 정적일 필요가 없다. 

Key 클래스 이름일 ThreadLocal로 변경하고 안에 있는 메서드를 전부 인스턴스 메서드로 변경한다

이렇게 변경하게 될경우 더이상 Key는 지역 변수를 구분하기 위해서 쓰이지 않게 되고, 그 자체가 스레드 지역변수가 된다.

그래도 아쉬운 점이 하나 더 있다.

#### 해결방안 - version 3  매개변수화 하여 타입안전성 확보
```java
public final class ThreadLocal<T> {
    public ThreadLocal();
    public void set(T value);
    public T get();
}
```

get 으로 얻은 Object를 실제 타입으로 형변환 해서 사용해야 하기 떄문에 타입이 안전하지 않다.

그래서 위와같이 매개변수화 타입을 이용하여 선언하면 안전하게 사용 가능하다


### 핵심정리
> 더 적합한 데이터 타입이 있거나 새로 작성 할 수 있으면(클래스 등) 문자열을 쓰지 말고 그 타입을 사용해라
