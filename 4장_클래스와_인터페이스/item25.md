# item25 톱레벨 클래스는 한 파일에 하나만 담아라

소스파일 하나에 톱레벨 클래스를 여러개 선언하더라고 자바 컴파일러는 잘 작동한다.
하지만 이러한 행위는 심각한 위험을 발생시키는 행위이다.

### Why?
어느 소스 파일을 먼저 컴파일 하느냐에 따라 작동이 달라지기 때문이다.

아래 예시를 봐보자

```java
// Utensil.java
class Utensil {
    static final String NAME = "PAN";
}

class Dessert {
    static final String NAME = "CAKE";
}
```

```java
// Dessert.java
class Utensil {
    static final String NAME = "PAN";
}

class Dessert {
    static final String NAME = "CAKE";
}
```


```java
// MAIN
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

위와같이 코드를 작성 한 후 컴파일을 해보자
1) `javac Main.java Dessert.java`
컴파일 오류가 발생하고, Utensil과 Dessert가 중복되었다고 알려준다.

2) `javac Main.java Utensil.java` or `javac Main.java`
컴파일 하면 Dessert.java를 작성하기 전처럼 `pancake`를 출력한다.

3) `javac Dessert.java Main.java`
컴파일 하면 `potpie`를 출력한다.

위와 같이 어느 소스파일을 먼저 컴파일러에게 건네느냐에 따라 동작 방식이 달라지는 문제가 있다.

### 해결방안
1. 톱레벨 클래스들을 정적 멤버 클래스를 이용하여 서로다른 클래스로 분리해보자
2. 인텔리제이가 빨간글씨로 문제가 있다고 알려준다..!



