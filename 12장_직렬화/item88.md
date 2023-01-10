# Item 88. readObject는 방어적으로 작성하라

```java
public class Period {
	private final Date start;
	private final Date end;

	// 방어적 복사
	public Period(Date start, Date end) {
		this.start = new Date(start.getTime());
		this.end = new Date(end.getTime());
		if (this.start.compareTo(this.end) > 0) {
			throw new IllegalArgumentException();
		}
	}

	// 방어적 복사
	public Date start() {
		return new Date(start.getTime());
	}

	// 방어적 복사
	public Date end() {
		return new Date(end.getTime());
	}

	public String toString(){
		return start + " - " + end;
	}
}
```

방어적 복사를하는 아이템 50에 나왔던 코드인데, 이코드를 직렬화하기로 한다면?    
=> 물리적표현과 논리적 표현이 부합하는 케이스이기때문에 Serializable 을 구현하면 문제 없을 수  있어보인다.

## readObject

> p468 readObject메서드가 실질적으로 또다른 public 생성자이기때문에 다른 생성자와 똑같이 대해야한다. (매개변수 유효성검사 등)   

=> readObject가 매개변수 유효성검사 하지않고, 방어적 복사를 하지않는다면 위의 클래스의 불변식을 깨뜨릴 수 있다.


#### 실질적으로 또다른 public 생성자이다?

ObjectInputStream 클래스에 존재하는 메서드   
   
![](https://velog.velcdn.com/images/rodlsdyd/post/2b6372a2-9801-4e5b-bb3e-4aff50ccafe8/image.png)

readObject0 메서드가 직렬화된 객체를 읽어들여서 Object타입으로 반환    
=> 실질적 public 생성자

직렬화된 객체를 바이트 스트림 형태로 읽어들여서 일반객체로 복원하는데 사용되기때문에 바이트 스트림을 받는 생성자라 할수있다고 한것이다(p468)

### 문제점 
> 불변식을 깨뜨릴 의도로 **임의 생성한 바이트스트림을 readObject에 건네면 문제가생긴다. **   
=> 정상적인 생성자로 만들어 낼수없는 객체를 생성해 낼수있기때문


```java
// 임의의 바이트 스트림을 건넨 예시
public class BogusPeriod {

	private static final byte[] serializedForm = {
		0x50, 0x65
		// Period를 직렬화한 바이트스트림과 다른 임의의 바이트스트림
	};

	public static void main(String[] args) {
		Period p = (Period) deserialize();
		System.out.println(p);
	}

	static Object deserialize() {
		try {
			return new ObjectInputStream(new ByteArrayInputStream(serializedForm)).readObject(); // 당연히 정상 Period 인스턴스와 다른 결과가 나온다.
		} catch (Exception e) {
			throw new IllegalArgumentException(e);
		}
	}
}
```
readObject는 바이트스트림을 가지고 Object타입으로 변환해주는 메서드이기때문에 Period를 직렬화한 바이트스트림이랑 다른 바이트스트림을 넣으면 당연히 다른결과가 나온다.


- Serializable을 implements한 클래스에서는 readObject()를 사용할수있다 ->          
Serializable을 implements해서 불변식이 깨질수있다.


### 해결방법

- readObject 메서드가 defaultReadObject를 호출한다음, 역직렬화된 객체가 유효한지 검사해야한다

```java
public class Period implements Serializable {
	private final Date start;
	private final Date end;


	...
    
    ...
    
    ...
    
    ...

	private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
		s.defaultReadObject();

		if (this.start.compareTo(this.end) > 0) {
			throw new InvalidObjectException("유효성 검사 실패");
		}
	}
}
```

   
**직렬화 가능한 클래스 내부에 readObject를 커스텀하게 정의해줘야한다.**   
   
- 이 메서드를 정의하게되면 ObjectInputStream의 readObject()대신 이 메서드의 방식대로 역직렬화 할수있다.

- defaultReadObject()의 역직렬화 방식은 readObject()와 비슷하다 

- 커스텀 하게 정의한 readObject()의 의미는 **ObjectInputStream 의 역직렬화 방식을 진행한후, 객체가 유효한지 추가로 검사할수있는 방법**   



> Serializable을 implements 한 클래스 내에서 readObject를 선언했다   
=> 기본적으로 역직렬화 할때 제공되는 방법인 ObjectInputStream 내의 readObject()를 사용하는방법대신 해당 메서드의 방법으로 객체를 역직렬화 하겠다는 의미 

### 두번째문제점 

> 가짜 바이트스트림을 이용해 Period인스턴스를 생성하는건 막을수있지만   
정상 Period인스턴스에서 시작된 바이트스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어낼수있다.

```java
public class MutablePeriod {
	public final Period period;
	public final Date start;
	public final Date end;

	public MutablePeriod() {
		try {
			ByteArrayOutputStream bos = new ByteArrayOutputStream();
			ObjectOutputStream out = new ObjectOutputStream(bos);

			// 유효한 Period 인스턴스를 직렬화
			out.writeObject(new Period(new Date(), new Date()));

			/**
			 * 악의적인 참조(내부 Date 필드로의 참조를 추가)
			 */
			byte[] ref = {0x71, 0, 0x7e, 0, 5}; // 가짜 참조를 생성하는 코드
			bos.write(ref); // ref를 Period 객체 내부의 start필드로 참조를 추가한다.
			ref[4] = 4;
			bos.write(ref); // start필드의 참조를 end 필드로 추가한다.

			ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
			this.period = (Period)in.readObject();
			this.start = (Date)in.readObject();
			this.end = (Date)in.readObject();

		} catch (Exception e) {
			e.printStackTrace();
			throw new RuntimeException();
		}
	}

	public static void main(String[] args) {
		MutablePeriod mp = new MutablePeriod();
		Period p = mp.period;
		Date pEnd = mp.end;

		pEnd.setYear(78);
		System.out.println(p);

		pEnd.setYear(69);
		System.out.println(p);
	}
}
```

![](https://velog.velcdn.com/images/rodlsdyd/post/26a03232-2e1d-4ca2-b2cd-787dc87de188/image.png)

추가한 참조를 통해 Period 객체의 start, end필드가 변경된 모습 

=> 클라이언트가 소유해서는 안되는 객체참조를 갖는 필드는 모두 반드시 방어적 복사를 해야한다   
=> readObject에서는 불변 클래스 안의 모든 private 가변요소를 방어적 복사해야한다.

### 방어적복사 + 유효성검사 수행하는 readObject

```java
	private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
		s.defaultReadObject();

		// 방어적 복사
		start = new Date(start.getTime());
		end = new Date(end.getTime());
		
        // 유효성 검사
		if (this.start.compareTo(this.end) > 0) {
			throw new InvalidObjectException("유효성 검사 실패");
		}
	}
```

final 필드를 제거해야만 사용할수있다.

![](https://velog.velcdn.com/images/rodlsdyd/post/0a9e602f-72c9-434c-9ecb-e40d1b8da9c8/image.png)

정상적인 2023년 값이 출력된다 

### 기본 readObject를 써도 좋은지 판단기준

- transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가 ? 
=> no라면, 커스텀 readObject 메서드로 모든유효성 검사 + 방어적 복사 수행
=> 혹은 직렬화 프록시 패턴(Item90)


#### 주의점

final이 아니고 직렬화 가능한 클래스라면 readObject메서드는 재정의 가능 메서드를 호출해선 안된다. 호출할경우 프로그램 오작동으로 이어진다.

=> 상위클래스의 readObject()가 호출되기전에 즉, 상위클래스의 필드들이 완전히 역직렬화 되기전에 하위클래스의 readObject()가 호출되는경우 


## 결론

- readObject메서드는 public 생성자를 작성하는 자세로 임해야한다(유효성검사, 참조를 갖는필드 방어적복사)

- 검사가 필요하다면 readObject()를 직렬화가능한 클래스에 정의하자

- 직,간접적으로 재정의 할수있는 메서드는 호출하지말자
