# Item32. 제네릭과 가변인수를 함께 쓸 때는 신중하라

- 가변인수 메서드를 호출하면 가변인수를 담기위한 배열이 자동으로 생성된다
=> 이 배열이 클라이언트에 노출되어 매개변수에 제네릭,매개변수화 타입이 포함되면 컴파일경고가 발생한다   
   
![](https://velog.velcdn.com/images/rodlsdyd/post/9bba9d95-d4c6-4df2-b5d9-cc9eee60e0fe/image.png)
   
실체화 불가타입(제네릭, 매개변수화 타입)으로 매개변수를 선언하면 컴파일러의 경고를 볼수있다.


![](https://velog.velcdn.com/images/rodlsdyd/post/c3dc3dc0-589f-41f0-bb9f-76279f52458c/image.png)

제네릭 매개변수로 인해서 안전하지않은 런타임 예외를 발생시킬수 있고 힙 오염이 발생한다. 

### 힙오염

- 매개변수화 타입의 변수가 타입이 다른 객체를 참조할때 발생하는 현상.    
컴파일 당시 에러가 발생하지않고 런타임에 ClassCastException 이 발생하게된다.



위의코드는 Item28에서 봤었던 코드와 매우 유사한데 제네릭배열은 컴파일에러가 발생하지만위의코드는 컴파일에러가 발생하지않는다. 



### 이유
![](https://velog.velcdn.com/images/rodlsdyd/post/f44730a7-fbdc-438e-8c14-0736b3cbfeb9/image.png)

- 제네릭이나 매개변수화 타입의 매개변수를 받는 메서드가 매우 유용하기때문이다.
=> 이런 모순을 수용한 이유

- 자바7에 @SafeVarags 애너테이션이 추가되어 이들은 타입안전하다    
=> 가변인자가 안전하게 사용되고있다는(경고를 무시) 마킹   
=> @SuppressWarnings("unchecked") 보다 범위가 좁고 구체적이다.    

### 안전한걸 어떻게 아는지?
가변인수 메서드를 호출할때 매개변수들을 담는 제네릭 배열이 만들어 진다.    
=> 이 매개변수들이 **순수하게 인수들을 전달하는 일만 한다면** 안전하다고 볼수있다.

- varargs 매개변수 배열에 아무것도 저장하지않는다.   
=> 뭔가를 넣는순간 위험성이 생긴다

- 컴파일러가 만드는 제네릭배열(혹은 복제본)을 신뢰할수없는 코드에 노출하지않는다.

### 주의점

```java
public class Test {

    public static void main(String[] args) {
        String[] attributes = pickTwo("좋은", "빠른", "저렴한"); 
        // ClassCastException 
    }
    
    static <T> T[] pickTwo(T a, T b, T c) {
        switch (ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, b);
        }
        throw new AssertionError(); // 도달 할 수 없음
    }

    static <T> T[] toArray(T... args) {
        return args; // 제네릭배열을 리턴하여 위험함
    }
}
```

- 제네릭 varargs 매개변수 배열에 다른메서드가 접근하도록 허용하면 안전하지않다.   
=> @SafeVarargs로 제대로 애노테이트 되었거나, 이 배열의 내용의 일부 함수를 호출만하는 일반 메서드에 넘기는것도 안전하다 

> 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든메서드에 @SafeVarargs를 달아라.

- 메서드에 힙 오염 경고가 뜬다면, 진짜 안전한지 점검해라

### @SafeVarargs이 아닌 또다른방법

#### @SafeVarargs + 제네릭 varargs
![](https://velog.velcdn.com/images/rodlsdyd/post/165e75d1-82f1-4aa3-90c1-05c61932740f/image.png)

#### 제네릭 varargs -> List매개변수

![](https://velog.velcdn.com/images/rodlsdyd/post/e1ca1f6a-ba95-4835-b75f-29c4db085b72/image.png)

- 클라이언트 코드가 살짝 지저분해질수있지만 컴파일러가 메서드의 타입안정성을 검증할수있다.



### 정리

- 가변인수와 제네릭을 함께쓴다면 신중해야한다. 배열과 제네릭의 타입 규칙이 서로 다르기때문이다.

- 제네릭 varargs 매개변수는 타입 안전하지않지만 허용된다. 

- 메서드에 제네릭 varargs 매개변수를 사용하려고한다면 안전성을 체크한후 @SafeVarargs 애너테이션을 달아라 

- 내가 사용한다고 하면 제네릭 varargs도 안전하게 사용할 방법이 있겠지만 List로 변환하는 방식을 사용할것 같다.
