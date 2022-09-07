# Item.03 private 생성자나 열거 타입으로 싱글턴임을 보증하라
---

### 싱글턴(Singleton)이란?
인스턴스를 오직 하나만 생성할 수 있는 클래스.  즉, 딱 하나의 동일한 인스턴스임을 보장하는 클래스를 말한다

싱글턴 패턴을 사용하면 다음과 같은 장점이 있다
- 동일한 인스턴스를 얻기위해 무분별한 new 연산을 하지 않으므로 메모리 낭비를 방지할 수 있다
- 인스턴스 생성 비용이 큰 경우 딱 한번만 생성하여 재사용할 수 있다
- ~~global access 를 통해 데이터 공유가 가능하다~~
	- (싱글톤 == global accessible instanace) -> false 아닌가?
	- 폐쇄적으로 접근가능하지만 딱 1개만 만들어지는 클래스도 싱글턴아닌가?
	- ==**싱글턴**이라는 개념은 **전역/비전역**이라는 개념과 분리해서 봐야한다==

단점으로는
- 싱글톤 인스턴스가 너무 많은 책임을 갖게되면, 이를 사용하는 클래스들 간의 결합도가 높아지게 되고 유지보수가 어렵다
- 개발자의 실수로 '상태'를 가지게 된다면 동시성 문제가 발생할 수 있다(무상태를 보장해야함)
- 테스트를 하기 위해서는 반드시 인터페이스를 구현해야 한다
	- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워 질 수 있다. 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면, 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없기 때문이다. (책 23p)


### 싱글턴을 만드는 첫번째 방법(private 생성자 + public static final)
생성자는 private으로 감춰두고, public static 멤버를 통해서만 인스턴스에 접근 가능하게 한다
```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	
	private Elvis() {
		// ...
	}

	public void leaveTheBuilding(){
		//...
	}
}
```
public static final 필드인 INSTANCE는 초기화될 때 딱 한번만 호출되고, 클래스 외부에서 접근 가능한 생성자가 없기 때문에 싱글턴을 보장한다

이 방법의 장점으로는 Elvis.INSTANCE 로 접근할 수 있어 명확하고, 소스가 간결하다


### 싱글턴을 만드는 두번째 방법(private 생성자 + 정적 팩터리 메서드)
정적 팩터리 메서드를 public static 멤버로 제공한다
```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	
	private Elvis() {
		// ...
	}

	public static Elvis getInstance() {
		return INSTANCE;
	}

	public void leaveTheBuilding() {
		//...
	}
}
```
첫번째 방법과 동일하게 static final 필드를 초기화 한 후, Elvis.getInstance()로 접근하여 항상 같은 객체의 참조를 반환함으로써 싱글턴임을 보장한다

두번째 방법의 장점은 다음과 같다
1) 클라이언트 코드의 변경 없이 동작 방식을 변경할 수 있다
2) ==정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다(Item.30)==
3) 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다([Java에서 제공하는 함수형 인터페이스](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html))


### 첫번째, 두번째 방식의 문제점

####   리플렉션(Reflection) API 에 취약하다
리플렉션 API를 사용하면 ==AccessibleObject.setAccessible== 을 통해 private 생성자의 접근 제어 keyword에 대한 제어를 변경하면 private 생성자를 호출할 수 있다

```java
public static void main(String[] args) {  
    try {  
	    // getDeclaredConstructor는 getConstuctor와 달리 
	    // private 생성자 까지 호출 가능하다
        Constructor<Elvis> defaultConstructor 
        = Elvis.class.getDeclaredConstructor(); 
    
	    // 생성자 접근제어 허용
        defaultConstructor.setAccessible(true);

		// 객체 생성 가능
        Elvis elvis1 = defaultConstructor.newInstance();  
        Elvis elvis2 = defaultConstructor.newInstance();  
		// Elvis.INSTANCE와 elvis1, elvis2는 모두 다른 hashCode
		
    } catch (InvocationTargetException | NoSuchMethodException | InstantiationException | IllegalAccessException e) {  
        e.printStackTrace();  
    }  
}
```

이러한 공격을 방어하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면된다
```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private static boolean created;
	
	private Elvis() {
		if (created) {  
		    throw new UnsupportedOperationException("can't be created by constructor.");  
		}  
		created = true;
	}

	public void leaveTheBuilding(){
		//...
	}
}
```

####   역직렬화(Deserialize) 시 싱글턴을 보장하지 않는다
```java
public static void main(String[] args) {
	// 직렬화
    try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("elvis.obj"))) {  
        out.writeObject(Elvis.INSTANCE);  
    } catch (IOException e) {  
        e.printStackTrace();  
    }  

	// 역직렬화
    try (ObjectInput in = new ObjectInputStream(new FileInputStream("elvis.obj"))) {  
        Elvis elvis3 = (Elvis) in.readObject();  
        System.out.println(elvis3 == Elvis.INSTANCE);  
        // false 출력
    } catch (IOException | ClassNotFoundException e) {  
        e.printStackTrace();  
    }  
}
```
이러한 경우 싱글턴 클래스에 Serializable 인터페이스를 구현하고, readResolve() 메소드를 구현해야 한다
```java
public class Elvis implements Serializable {
	public static final Elvis INSTANCE = new Elvis();
	private static boolean created;
	
	private Elvis() {
		if (created) {  
		    throw new UnsupportedOperationException("can't be created by constructor.");  
		}  
		created = true;
	}

	public void leaveTheBuilding(){
		//...
	}

	// 문법적으로는 Override는 아니지만 Deserialize 할 때 이 메소드가 사용된다  
	private Object readResolve() {  
	    return INSTANCE;  
	}
}
```
위 두가지 문제점을 극복하고 테스트도 가능한 싱글턴 클래스를 생성하기 위해선 아래와 같이 클래스를 만들어 주어야한다
```java
public class Elvis implements IElvis, Serializable {  
  
    public static final Elvis INSTANCE = new Elvis();  
    private static boolean created;  
  
    private Elvis() {  
        if (created) {  
            throw new UnsupportedOperationException("can't be created by constructor.");  
        }  
  
        created = true;  
    }  
  
    public void leaveTheBuilding() {...}  
  
    // 문법적으로는 Override는 아니지만 Deserialize 할 때 이 메소드가 사용된다  
    private Object readResolve() {  
        return INSTANCE;  
    }    
}
```
첫번째와 두번째 방식에서 장점으로 꼽았던 간결하다는 장점이 무너짐과 동시에, 위와 같은 배경 지식이 없다면 이해하기 난해한 코드가 되어버렸다


### 싱글턴을 만드는 세번째 방법
원소가 하나인 열거(enum) 타입을 선언한다
```java
public enum Elvis {
	INSTANCE;
	public void leaveTheBuilding(){
		//...
	}
}
```
Enum 타입으로 만들어 주게되면 `더 간결`하고  `추가적인 노력 없이 직렬화` 할 수 있고, `리플렉션 공격에서도 방어가능`하다

> Enum 타입의 경우도 private 생성자가 있지만 리플렉션 API로도 접근할 수 없기 때문에 리플렉션 공격에 대해서 방어가 가능하다


단, 싱글턴이 enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다
(인터페이스를 구현하도록 선언할 수는 있다)

---
---

순수 java 소스로 완벽한 싱글턴을 보장하는 클래스를 만들기 위해서는 복잡한 소스와 깊이 있는 배경 지식이 필요하다
또한 위에서 언급하지 않은 Thread-safe에 대해서도 고민을 해야한다([Thread-safe한 싱글턴 패턴](https://github.com/jeff-seyong/Design-Pattern/tree/master/singleton-pattern))

하지만 우리 대부분(?)은 Spring을 사용하고 있고 Spring에서 Bean은 별도의 선언하지 않으면 기본적으로 싱글턴을 보장하는 것을 알고 있기 때문에 큰 고민하지 않고 쉽게 사용하고 있다


### 그렇다면 Spring에서는 어떻게 Bean들이 싱글턴임을 보장할까?
Spring에서는 Spring Container를 통해 싱글턴임을 보장한다
Spring Container에 Bean 저장소에다가 Bean 이름과 인스턴스를 매핑하여 저장해둔다
이후 의존성 주입이 필요한곳에 참조 하고자하는 인스턴스를 주입시켜준다

단, new 연산자를 통해서 인스턴스를 생성하는 경우에는 Spring에서 관리하고 있는 Bean이 아닌 클래스로부터 새로운 인스턴스를 생성하므로 주의해야 한다