
자바 라이브러리에서 close 메서드를 호출해 직접 닫아줘야 하는 자원 예시 : InputStream, OutputStream, java.sql.Connection 등
<br>자원 닫기는 예측할 수 없는 성능 문제로 이어져 finalizer을 사용하지만 썩 믿음직스럽지 못하다.


#### 코드 9-1. try-finally 더 이상 자원을 회수하는 최선의 방책이 아니다.

```
public static String firstLineOfFile(String path) throw IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

#### 코드 9-2. 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다!

```
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
			out.write(buf, 0, n);
		} finally {
			out.close();
		}
	} finally {
		in.close();
	}
}
```

- try-finally 구문을 하나만 쓰면 leak이 생길 수 있다.
- 예외는 try 블록과 finally 블록 모두에서 발생할 수 있음.
- 하지만 기기에서 물리적인 문제가 생긴다면 firstLineOfFile 메서드 안의 readLine 메서드가 예외를 던지고, 같은 이유로 close 메서드도 실패할 것이다. 
- 이런 상황에선 두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 버린다. 이 경우 디버깅이 굉장히 어렵다.

### 해결책 : try-with-resources
이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스(단순히 void를 반환하는 close 메소드 하나만 정의한 인터페이스)를 구현해야 한다. 
자바 라이브러리와 서드파티 라이브러리들의 수많은 클래스와 인터페이스가 이미 AutoCloseable을 구현하거나 확장해뒀다.    

#### 코드 9-3 try-with-resource 자원을 회수하는 최선책! (코드 9-1 재작성)

```
public static String firstLineOfFile(String path) throw IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))){
        return br.readLine();
    }
}
```

#### 코드 9-4 복수의 자원을 처리하는 try-with-resources 짧고 매혹적이다! (코드 9-2에 적용한 모습)

```
static void copy(String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0)
		out.write(buf, 0, n);
	}
}
```

#### firstLineOfFile 메서드 
- readLine과 (코드에서 보이지 않는) fclose 호출 양쪽에서 예외가 발생하면, close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외가 기록된다. 
- 이처럼 실전에선 프로그래머에게 보여줄 예외 하나만 보존되고 여러개의 다른 예외가 숨겨질 수도 있다.
- 이렇게 숨겨진 예외들은 스택 추적 내역에서 '숨겨졌다(suppressed)'라는 꼬리표를 달고 출력된다.
- 자바 7에서 Throwable에 추가된 getSuppressed 메서드를 이용하면 프로그램 코드에서 가져올 수도 있다.


#### 코드 9-5 try-with-resouces를 catch절과 함께 쓰는 모습
```
public static String firstLineOfFile(String path) throw IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (Exception e) {
        return defaultVal;
    }
}
```
파일을 열거나 데이터를 읽지 못했을 때 예외를 던지는 대신 기본값을 반환하도록 한 예시.

### 핵심 정리
- 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자.
- 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다.
- try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resouces로는 쉽게 자원을 회수할 수 있다.
- 만들어지는 예외 정보도 훨씬 유용하다.

### 왜 try-with-resources를 써야하나요?

p48. 자바 퍼즐러 예외 처리 코드의 실수
```
public class Copy {
    private static final int BUFFER_SIZE = 8 * 1024;

    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
            try {
            	out.close();
             } catch(IOException e){
             	//TODO 이렇게 하면 되는 거 아닌지?
             }
             
             try {
            	in.close();
             } catch(IOException e){
             	//TODO 안전한지?
             }
    }
```

IOException이 아닌 RuntimeException이 발생하면 out.close()에서 끝남. 

p49. try-with-resources 바이트코드
```
public class TopLine{
	public TopLine(){
    }
    
    static String firstLineOfFile(String path) throws IOException{
    BufferedReader br = new BufferedReader(new FileReader(path));
    
    String var2;
    try{
    	var2 = br.readLine();
    } catch (Throwable var5) {
    	try{
        br.close();
        } catch(Throwable var4) {
        	var5.addSuppressed(var4);
            //close할 때 발생하는 예외를 계속 추가해줌
        }
        
        throw var5;
    }
   
   	br.close();
    return var2;
    }
    
    public static void main(String[] args) throws IOException{
    	String path = args[0];
        System.out.println(firstLineOfFile(path));
    }
}
```

- resource 반납을 위해 finally 구문을 쓰지 않고 중첩된 try-catch 문을 쓰고 있다.
- 첫 번째 에러를 그냥 던져주고, 추가적으로 발생하는 에러를 addSuppresed로 추가해줌.
