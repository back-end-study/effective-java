# [Item50] 적시에 방어적 복사본을 만들라


## 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사하라

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + " after " + end);
        }
        this.start = start;
        this.end   = end;
    }

    public Date getStart() {
        return start;
    }
    public Date getEnd() {
        return end;
    }
}
```

Period 클래스는 한번 값이 정해지면 변하지 않도록 하기 위해 final 클래스로 선언하였다

언뜻보면 불변처럼 보이는 이 Period 클래스는 Date가 가변이라는 사실을 이용하면 쉽게 불변식을 깨뜨릴 수 있다

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // end 객체의 참조를 통해 p의 내부를 수정 할 수 있다
```

위 코드에 대해서 몇가지 알아야 할 점은

- Date는 Deprecated API이므로 이제는 사용하지 말아야 한다
- Date 대신 immutable인 LocalDate나 LocalDateTime을 쓴다면 해결가능하다

하지만 꼭 Date 클래스만의 문제가 아니므로 다음 방법을 보자

**→ 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사하자**

```java
public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end   = new Date(end.getTime());
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + " after " + end);
        }
    }
```

순서가 부자연스러워 보일 수 있지만, 반드시 이렇게 작성해야한다

멀티스레딩 환경일 경우 원본 객체의 유효성을 검사한 뒤 복사본을 만드는 찰나에 다른 스레드가 원본 객체를 수정할 위험이 있기 때문이다

또한, 

**→ 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해선 안된다** 

공격자가 악의적인 clone 메서드를 심은 Date 클래스의 하위 클래스로 Period를 생성하는 경우, 공격자에게 Period 인스턴스 통제권을 그냥 내어준 셈이기 때문이다

## 가변 필드의 방어적 복사본을 반환하라

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.getEnd().setYear(78); // p객체의 end 멤버변수 참조를 통해 p의 내부를 수정 할 수 있다
```

**→ 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사하자**

```java
public Date getStart() {
    return new Date(start.getTime());
}

public Date getEnd() {
    return new Date(end.getTime());
}
```

생성자에서와는 달리 메서드에서는 방어적 복사에 clone을 사용해도 된다. Period가 가지고 있는 Date 객체는 java.util.Date가 확실하기 때문이다!

하지만 [**아이템13 clone 재정의는 주의해서 진행하라**](https://github.com/back-end-study/effective-java/blob/main/3%EC%9E%A5_%EB%AA%A8%EB%93%A0_%EA%B0%9D%EC%B2%B4%EC%9D%98_%EA%B3%B5%ED%86%B5_%EB%A9%94%EC%84%9C%EB%93%9C/item13.md) 에서 설명한 이유 때문에 인스턴스를 복사하는 데는 일반적으로 생성자나 정적 팩터리를 쓰는게 좋다

## 방어적으로 복사하는 목적이 불변 객체를 만들기 위해서만은 아니다

메서드든 생성자는 클라이언트가 제공한 객체를 내부 자료구조에 보관해야 하는 경우 변경 될 수 있는 객체라면 복사본을 만들어 저장해라

반대로, 내부 객체를 클라이언트에 건네주기 전도 마찬가지다

**→ 어떠한 참조에 대해서 객체의 불변을 확신할 수 없다면 원본을 노출하지말고 방어적 복사본을 반환하는게 좋다**

혹은, `Collections`의 `unmodifiable`을 사용하여 배열의 불변 뷰를 반환하는 방법을 사용하자

## 방어적 복사에는 성능 저하가 따르고, 또 항상 쓸 수 있는것도 아니다

- 복사 비용이 너무 크거나 클라이언트를 신뢰할 수 있는 상황이라서 방어적 복사를 생략한다면
    
    **→ 매개변수나 반환값을 수정하지 말아야 함을 명확히 문서화 하자**
    
- 때로는 방어적 복사 생략은 통제권을 이전함을 뜻하기도 한다
    
    → 클라이언트가 건네주는 가변 객체의 통제권을 (그에 따른 사이드 이펙트를)명확히 문서화 하자
    
- 다만, 상호 신뢰할 수 있는 경우 그리고 불변식이 깨지더라도 그 영향이 오직 호출한 클라이언트로 국한될 때로 한정해서 쓰는것이 좋다([**래퍼 클래스 패턴**](https://github.com/back-end-study/effective-java/blob/main/4%EC%9E%A5_%ED%81%B4%EB%9E%98%EC%8A%A4%EC%99%80_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4/item18.md#%EB%9E%98%ED%8D%BC%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%BD%9C%EB%B0%B1%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC-self-%EB%AC%B8%EC%A0%9C))