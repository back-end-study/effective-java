# item04 인스턴스화를 막으려거든 private 생성자를 사용하라
### 정적 메서드와 정적 필드만을 담은 클래스를 만드는 이유
- java.lang.Math, java.util.Arrays 처럼 기본 타입 값이나 배열 과련 메서드를 모아놓을 수 있다.
- java.util.Collections 처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아놓을 수도 있다.
- final클래스와 관련한 메서드들을 모아놓을 때도 사용한다.


### 정적 메서드만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 게 아니다. 
```java
public class UtilityClass {
	// 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용)
	private UtilityClass() {
		throw new AssertionError();
	}
}
```
- 클래스 안에서 생성자를 호출하지 않도록 Exception을 던져준다. 
- 직관적으로 주석을 이용해 생성자가 private인 의도를 전달한다.
- 상속을 불가능하게 하는 효과도 있다.