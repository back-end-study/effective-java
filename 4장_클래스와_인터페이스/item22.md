# item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라
---
인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.
클래스가 어떤 인터페이스를 구현한다 : 인스턴스로 무엇을 할 수 있는지를 클라이언트에 이야기 해주는 것


```java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double boltzmann_constant = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```
위와 같이 메서드 없이, 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스의 형태는 상수 인터페이스이다.
상수 인터페이스 안티패턴은 인터페이스를 잘못 사용한 예이다.
상수 인터페이스를 구현하는 것은 내부구현을 클래스의 API로 노출하는 행위이다.

### 상수를 공개할 목적이라면
1. 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야 한다.
   ex) 모든 숫자 기본 타입의 박싱 클래스가 대표적으로, Integer와 Double에 선언된 MIN_VALUE와 MAX_VALUE상수가 이런 예다.
2. 열거 타입으로 나타내기 적합한 상수라면 열거타입으로 만들어 공개한다.
3. 인스턴스화할 수 없는 유틸리티 클래스에 담아 공개한다.
   아래는 유틸리티 클래스 버전 코드이다.
```java
public class PhysicalConstants {
    
    private PhysicalConstants() { } // 인스턴스화 방지
    
    // 아보가드로 수 (1/몰)
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    public static final double boltzmann_constant = 1.380_648_52e-23;

    // 전자 질량 (kg)
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 클래스 이름까지 함꼐 명시해야 한다.
유틸리티 클래스의 상수를 빈번히 사용한다면 정적 임포트 (static import)하여 클래스 이름은 생략할 수 있다.
```java
import static effective_java.item_22.PhysicalConstants.*;

public class Test {
    
    double atoms(double mols) {
        return AVOGADROS_NUMBER * mols;
    }
}
```