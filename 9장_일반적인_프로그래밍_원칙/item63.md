# 아이템 63. 문자열은 느리니 주의해라

문자열 연결 연산자(+)는 여러 문자열을 하나로 합쳐주는 편리한 수단이다.
문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례한다. 문자열은 불변이라서 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야하므로 성능 저하는 피할 수 없는 결과다.

```java
// 문자열 연결을 잘못 사용한 예 - 느리다!
public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++) {
        result += lineForItem(i);
    }
    return result;
}
```
각 item의 원소 개수만큼 문자열을 잇는다. 품목의 개수가 많아지면 많아질수록 성능 저하가 심해진다.

성능을 포기하고 싶지 않다면 String 대신 StringBuilder를 사용하자!
```java
// 문자열 연결 성능이 크게 개선된다!
public String statement2() {
    StringBuilder sb = new StringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++) {
        sb.append(lineForItem(i));
    }
    return sb.toString();
}
```

>성능에 신경 써야 한다면 많은 문자열을 연결할 때는 문자열 연결 연산자(+)를 피하자.
대신 StringBuilder의 append 메서드를 사용하라
문자 배열을 사용하거나 문자열을 (연결하지 않고)하나씩 처리하는 방법도 있다.

 
