# Item 24. 멤버 클래스는 되도록 static 으로 만들라

중첩클래스

- static 멤버 클래스
- non-static 멤버 클래스
- 익명 클래스
- 지역 클래스

## static 멤버 클래스

- 바깥 클래스의 private 멤버에 접근할수있다

```java
public class OuterClass {
    private String outerClassName = "MyClass";
	
    //정적 멤버 클래스
    private static class PrivateStaticClass {
        void print() {
            OuterClass outerClass = new OuterClass();
            System.out.println(outerClass.outerClassName);
        }
    }
}
```

### 바깥 클래스와 함께 쓰일때 유용하게 사용될수있다.
- public 도우미 클래스로 쓰인다
- Calculator.Operation.PLUS 같은 형태로 원하는 연산을 참조할수있다. 


```java
public class Calculator {
	public static enum Operation {
      PLUS("+", (x, y) -> x + y),
      MINUS("-", (x, y) -> x - y);
  }
}
```


## non-static 멤버 클래스

- 비정적 멤버클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다

- 바깥 인스턴스 없이 생성할수없다.


```java
public class OuterClass {
    private String className = "MyClass";

    // 비정적
    public class nonStaticClass {
        private String className = "nonStatic";

        void print() {
            OuterClass.this.outerPrint(); //정규화된 this로 바깥인스턴스의 메서드를 호출
        }
    }

	// 바깥 인스턴스의 메서드
    public void outerPrint() {
        System.out.println(className);
    }
    
    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        OuterClass.nonStaticClass inner =  outerClass.new nonStaticClass();
        inner.print(); // MyClass 
    }    
}
```

### non-static 멤버클래스의 사용
- 어댑터를 정의할때 자주쓰인다.
어댑터? 어떤클래스의 인스턴스를 감싸 다른클래스의 인스턴스처럼 보이게하는 뷰로 사용

- Set, List같은 컬렉션 인터페이스 구현체들도 반복자를 구현할때 non-static 멤버클래스를 주로 사용한다.


![](https://velog.velcdn.com/images/rodlsdyd/post/bf80e196-c129-449d-952d-4d00ec4b8082/image.png)

![](https://velog.velcdn.com/images/rodlsdyd/post/ce693435-9e3a-4469-83c8-0a50eb491a26/image.png)

- 위의 ListItr 멤버클래스는 바깥클래스를 참조한다. 그래서 non-static으로 선언한듯하다
 


## non-static 멤버클래스 단점

- 멤버 클래스에서 바깥 인스턴스에 접근할일이 없다면 무조건 static을 붙여서 정적멤버 클래스로 만들자 

  - non-static 멤버 클래스는 바깥인스턴스의 외부참조를 가지게 되어 시간,공간이 낭비되기때문이다.
  - gc가 바깥 클래스의 인스턴스를 수거하지 못하는 메모리누수가 생길수도있다.
  
![](https://velog.velcdn.com/images/rodlsdyd/post/f1bd334c-8629-4432-b1bd-bb03d2f12ab4/image.png)

- 모든 엔트리가 맵과 연관되어있지만 엔트리의 메서드들(getKey, geyValue ..)은 맵을 직접 사용하지는 않는다.  따라서 엔트리를 private static 메서드로 선언하는것이 알맞다 
  - 실수로 static을 빠트려도 동작은 하나 모든 엔트리가 바깥 맵의 참조를 갖게되어 공간,시간을 낭비하게된다.
  

  
  
## 익명클래스

- 즉석에서 작은 함수객체나 처리객체를 만드는데 주로 사용된다. 

- non-static 문맥에서만 바깥 클래스의 인스턴스를 참조할수있다.

- 쓰이는 시점에 선언+객체 생성이 동시에 이루어진다

- static문맥에서 사용될때엔 static 변수외의 필드는 가질수없다.

```java
static List<Integer> intArrayAsList(int[] a) {

	Objects.requireNonNull(a);
	return new AbstractList<>() {
		@Override
		public Interger get(int i) { return a[i]; }
	
		@Override
		public Integer set(int i, Integer val) {
			int oldVal = a[i];
			a[i] = val;
			return oldVal;
		}

		@Override
		public int size() { return a.length; }
	}
}
```

- static 팩터리 메서드를 만들때 사용되기도 한다

## 지역클래스

```java
    public void foo() {
        class LocalClass{
            private String className;

            public LocalClass(String className) {
                this.className = className;
            }

            public void print() {
                // 비정적 문맥에선 바깥 인스턴스를 참조 가능
                System.out.println("로컬클래스= " + className);
                // foo()가 static이면 className을 참조할수없다 컴파일 에러
            }
        }
        LocalClass localClass = new LocalClass("로컬");
        localClass.print();
    }
```

- 지역변수를 선언할수있는 곳이면 어디서든 선언가능하다
- 지역변수와 같은 스콥을 가진다
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 클래스 객체를 참조할 수 있으며, 정적 필드는 가질 수 없고 가독성을 위해 짧게 작성해야 한다.


  
### 정리

- 바깥 클래스를 참조할일이 없다면 static멤버클래스로 만들어라 (non-static으로 만든다면 쓸데없는 외부참조가생겨 낭비가 생긴다)

- 메서드 밖에서도 사용해야하거나 메서드안에 정의하기엔 너무 길다면 멤버클래스로 만든다.
