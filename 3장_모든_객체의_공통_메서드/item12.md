# item 12. toString을 항상 재정의하라 

## 1️⃣ toString의 일반 규약

### 1-1. 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다.

- Object의 기본 toString : `클래스_이름@16진수_해시코드`
    - 유익하지 않은 경우가 대부분이다.
- 예시(전화번호)
    ```java
    public final class PhoneNumber {
        // 지역코드 + 프리픽스 + 가입자번호
        private final short areaCode, prefix, lineNum;
        // 생성자 생략
    
        @Override
        public String toString() {
            return String.format("%03d-%03d-%04d",
                    areaCode, prefix, lineNum);
        }
    }
    ```

### 1-2. 모든 하위 클래스에서 이 메서드를 재정의하라.

- 직접 호출하지 않아도, 다른 어딘가에서 충분히 사용될 수 있다.
    - 디버거가 객체를 출력
    - 문자열 연결 연산
    - assert 구문
    - 객체를 참조하는 컴포넌트가 오류 메시지를 로깅할 때

> 상황에 맞는 적절한 toString을 항상 재정의하는 것이 일반 규약이라고 한다.


## 2️⃣ 재정의 Guide

### 2-1. 어떤 필드(정보)를 출력할까

- 보통은 객체가 가진 `주요 정보`를 모두 반환하는게 좋다.
- 너무 많다면 `요약 정보`를 담자.

> 실무에서는 외부에 노출할 수 있는 데이터만 반환해야한다.

### 2-2. 반환값의 포맷을 문서화할까

- 포맷을 명시해도 되고 안해도 된다. (`개발자의 의도를 명확하게 밝히면 됨`)
    - 문서화할 경우 정적 팩터리나 생성자를 함께 제공하자.
- 문서화 장점
    - 명확하고, 가독성이 좋음(표준화)
    - 데이터 객체로 저장도 가능
- 문서화 단점
    - 포맷에 얽매이게 됨(의존적이게 된다)
    - 변경과 확장에 불리

```java
/**
 * 이 전화번호의 문자열 표현을 반환한다.
 * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
 * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
 * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
 *
 * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
 * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
 * 전화번호의 마지막 네 문자는 "0123"이 된다.
 */
@Override public String toString() {
    return String.format("%03d-%03d-%04d",
            areaCode, prefix, lineNum);
}

// 정적 팩터리도 함께 제공
public static PhoneNumber of(String phoneNumberString) {
    String[] split = phoneNumberString.split("-");
    PhoneNumber phoneNumber = new PhoneNumber(
            Short.parseShort(split[0]),
            Short.parseShort(split[1]),
            Short.parseShort(split[2]));
    return phoneNumber;
}
```

### 2-3. 반환한 값에 포함된 정보를 API로 제공하면 좋다

- toString의 반환 값을 getter와 같은 `정보 접근 API를 제공`하자

```java
public short getAreaCode() {
    return areaCode;
}

public short getPrefix() {
    return prefix;
}

public short getLineNum() {
    return lineNum;
}
```

### 2-4. 예외(재정의 필요 없음)

- 상위 클래스에서 알맞게 재정의한 경우(Enum 클래스 등)
- 정적 유틸리티 클래스

### 2-5. 자동 생성에 의존해도 될까?

- 자동 생성 가능한 종류
    - AutoValue 프레임워크, IDE, Lombok 프레임워크

> 위 PhoneNumber 클래스와 같이 직접 재정의하는게 의미있는 경우가 더 많기 때문에 자동 생성에 의존하지 않는게 좋다.

## 3️⃣ 핵심 정리

- toString은 필요한 주요 정보를 사람이 읽기쉽게 출력하도록 재정의하자
- 값 클래스라면 포맷을 문서에 명시하는 것이 좋으며 해당 포맷으로 객체를 생성할 수 있는 정적 팩터리나 생성자를 제공하는 것이 좋다.
- toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하라.(getter)
- 경우에 따라 자동 생성보다 직접 재정의하는게 적절할 수 있다.
- 상위 클래스에서 적절히 재정의하였다면, 재정의할 필요 없다.
