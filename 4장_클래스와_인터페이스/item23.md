# Item.23 태그 달린 클래스보다는 클래스 계층구조를 활용하라
---

### 태그 클래스란?
> 두가지 이상의 의미를 표현할 수 있으며, 현재 가지고 있는 객체의 타입, 값 등을 필드로 표현한 클래스


### 태그 클래스 예시
```java
public class Figure {  
    enum Shape {RECTANGLE, CIRCLE}  
  
    // 태그 필드 - 현재 모양을 나타낸다.  
    final Shape shape;  
  
    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.  
    double length;  
    double width;  
  
    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.  
    double radius;  
  
    // 사각형용 생성자  
    public Figure(double length, double width) {  
        shape = Shape.RECTANGLE;  
        this.length = length;  
        this.width = width;  
    }  
  
    // 원용 생성자  
    public Figure(double radius) {  
        shape = Shape.CIRCLE;  
        this.radius = radius;  
    }  
  
    double area() {  
        switch (shape) {  
            case RECTANGLE:  
                return length * width;  
            case CIRCLE:  
                return Math.PI * (radius * radius);  
            default:  
                throw new AssertionError(shape);  
        }  
    }  
}
```


### 태그 클래스의 단점
1. 여러 구현이 한 클래스에 혼합되어 있어 가독성이 좋지 않다
2. 다른 의미를 위한 코드를 언제나 함께 가지고 있어야하기 때문에 불필요한 메모리를 사용한다
3. 필드를 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해서 써야 하기 때문에 불필요한 코드가 늘어난다
4. 엉뚱한 값으로 필드를 초기화되는 경우, 컴파일 타임에서 알아내기 힘들다 (런타임에서 문제 발생 인지)
5. 다른 의미를 추가하려면 코드 수정이 반드시 필요하다
6. 인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 전혀 없다

즉, 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다

> *태그 달린 클래스는 클래스 계층 구조를 어설프게 흉내낸 아류일 뿐이다.*


### 클래스 계층구조로 바꾸는 방법
- 클래스 계층구조?

> 공통으로 적용되는 기능을 슈퍼클래스로 추출하여 재사용성을 높이고 가독성을 높일 수 있는 방식

1. 계층구조의 루트(root)가 될 추상 클래스를 정의한다
2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다
3. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 선언한다
4. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스에 선언한다
5. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다


### 클래스 계층구조로 바꾼 태그 클래스
```java
// 1. 계층구조의 루트(root)가 될 추상 클래스를 정의한다
abstract class Figure {  
	// 2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다
    abstract double area();  
}

// 5. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다
class Rectangle extends Figure {  
  
    private final double length;  
    private final double width;  
  
    public Rectangle(double length, double width) {  
        this.length = length;  
        this.width = width;  
    }  
  
    public double getLength() {  
        return length;  
    }  
  
    public double getWidth() {  
        return width;  
    }  
  
    @Override  
    double area() {  
        return length * width;  
    }  
}

// 5. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다
class Circle extends Figure{  
  
    private final double radius;  
  
    public Circle(double radius) {  
        this.radius = radius;  
    }  
  
    public double getRadius() {  
        return radius;  
    }  
  
    @Override  
    double area() {  
        return Math.PI * (radius * radius);  
    }  
}
```

### 클래스 계층구조 장점
1. 간결하고 명확하며 불필요한 코드들을 모두 제거해주었다
2. 필드들을 모두 final로 선언하여도 불필요한 코드가 없으며 컴파일 타임에서 문제를 확인 할 수 있게 되었다
3. 루트 클래스의 코드를 건드리지 않고도 계층구조를 확장하고 함께 사용할 수 있다
4. 타입이 의미별로 따로 존재하니 변수의 의미를 명시하거나 제한할 수 있고, 특정 매개변수로 받을 수 있다

또한, 추가적인 예시로 Rectangle 클래스를 확장하여 Square 클래스를 만든다고 할 때
Figure, Rectangle 클래스의 수정 없이도 아래와 같이 가능하다

```java
class Square extends Rectangle {  
    public Square(double side) {  
        super(side, side);  
    }  
}
```
---

### 핵심 정리
- 태그 달린 클래스를 써야 하는 상황은 거의 없다
- 태그 필드가 등장한다면 태그를 없애고 계층구조로 대체하는 방법을 생각해보자
- 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩터링하는 걸 고민해보자
