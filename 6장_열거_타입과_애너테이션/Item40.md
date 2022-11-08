# Item40. @Override 애너테이션을 일관되게 사용하라

```java
// 매개변수가 Object 타입이어야하는데 Bigram으로, 오버로딩 되었다
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }
```



![](https://velog.velcdn.com/images/rodlsdyd/post/d2dfa102-1fd2-47b3-a6d9-882480bd88e7/image.png)

위 메서드에 @Override를 붙이면 컴파일러가 오류를 찾아준다


### 메서드를 재정의하려거든 모든 메서드에 @Override 애너테이션을 달자

- 컴파일시 버그를 발견할수있다

- 구체 클래스에서 상위 클래스의 추상메서드를 재정의하는경우엔 굳이 @Override를 달지 않아도 된다.
-  (저는 무조건 달려있는게 구조파악에 좋았던것 같은데 다른분들은 궁금하네요..)
