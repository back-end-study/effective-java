# Item.16 pulibc 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
---

필드만 정의된 것 외에는 아무 목적도 없는 클래스를 작성하는 경우가 있을 것이다

```java
public class Point {
	public double x;
	public double y;
}
```
이런 클래스는 외부에서 필드로 직접 접근 할 수 있어 캡슐화의 이점을 제공하지 못한다
또한,
- 클라이언트 코드 수정 없이는 표현 방식을 바꿀 수도 없고
- 불변식을 보장할 수도 없으며
- 부수 작업을 수행할 수도 없다

### public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하자
위 클래스의 필드를 모두 private로 바꾸고 public 접근자를 추가해보자
```java
public class Point {  

    private double x;  
    private double y;  
  
    public Point() {  
    }  
  
    public Point(double x, double y) {  
        this.x = x;  
        this.y = y;  
    }  
  
    public double getX() {  
        return x;  
    }  
  
    public double getY() {  
        return y;  
    }  
  
    public void setX(double x) {  
        this.x = x;  
    }  
  
    public void setY(double y) {  
        this.y = y;  
    }  
}
```
위 코드처럼 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 갖게 하는 것이 좋다

### package-private 클래스 혹은 private 중첩 클래스라면?
package-private 클래스 혹은 private 중첩 클래스에서 데이터 필드를 노출하는 것은 본질적으론 문제가 없다

package-private 클래스에서 public으로 필드를 노출하는 경우 동일 패키지 내에서 접근할 때 더 간결하다는 장점이 있다
또한 만약 변경되는 것이 바람직하다면 패키지 외부의 소스 수정 없이 동일 패키지 내에서만 수정을 하면 된다
(그 패키지가 엄청 커서 수정을 많이 해야된다면, 패키지 분리를 잘 하지 못한거겠지?)

private 클래스의 경우 '해당 클래스에서만'이라는 더 강한 제약이면서 해당 클래스에서만 수정하면 된다


### Java awt 패키지의 Point와 Dimension
java.awt 패키지의 Point와 Dimension 클래스를 타산지석으로 삼자.


### 불변 필드는 public으로 해도 괜찮은가?
불변 필드의 경우 불변식은 보장할 수 있지만, 
클라이언트 코드를 수정하지 않고선 API 표현을 바꾼다거나 부수적인 작업을 할 수 없다는 단점은 여전히 존재한다
