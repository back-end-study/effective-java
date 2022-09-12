# item06 불필요한 객체 생성을 피하라
### 똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을때가 많다. 재사용은 빠르고 세련되다. 특히 불변 객체(아이템 17)는 언제든 재사용할 수 있다.

잘 못된 객체 생성 예
```java
// 따라 하지 말것!! - 실행될 때마다 String 인스턴스를 생성
String s = new String("bikini");
// 개선된 버전 - 실행될 때마다 하나의 String 인스턴스를 사용
String s = "bikini";
```
> [String Literals jls-3.10.5](https://docs.oracle.com/javase/specs/jls/se17/html/jls-3.html#jls-3.10.5)

### 생성자 대신 정적 팩터리 메서드(item01)를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다.
ex) Boolean(String) 생성자 대신 Boolean.valueOf(String) 팩터리 메서드 사용, 자바 9에서 deprecated API로 지정되었다. [deprecated list](https://docs.oracle.com/en/java/javase/17/docs/api/deprecated-list.html)
```java
// public static final Boolean TRUE = new Boolean(true);  
// public static final Boolean FALSE = new Boolean(false);

// Boolean.valueOf(String)
public static Boolean valueOf(String s) {  
    return parseBoolean(s) ? TRUE : FALSE;  
}

// Boolean.valueOf(boolean)
public static Boolean valueOf(boolean b) {  
    return (b ? TRUE : FALSE);  
}


// Long.valueOf(long)
public static Long valueOf(long l) {  
    final int offset = 128;  
    if (l >= -128 && l <= 127) { // will cache  
        return LongCache.cache[(int)l + offset];  
    }  
    return new Long(l);  
}
```

### 생성 비용이 아주 비싼 객체가 반복해서 필요하다면 캐싱하여 재사용하길 권한다.
```java
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]}L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
String.matches는 정규식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않다. String.matches내부에서 사용하는 Pattern 인스턴스는 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상이 된다. Pattern은 입력받은 정규표현식에 해당하는 [유한 상태 머신](https://ko.wikipedia.org/wiki/%EC%9C%A0%ED%95%9C_%EC%83%81%ED%83%9C_%EA%B8%B0%EA%B3%84)을 만들기 때문에 인스턴스 생성 비용이 높다.
성능 개선을 위해 필요한 정규표현식을 표현하는 Pattern 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱해두고 재사용한다.
```java
public class RomanNumerals {
    private static final Pattern ROMAN = Patterns.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]}L?X{0,3})(I[XV]|V?I{0,3})$");
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```
개선된 방식의 클래스가 초기화된 후 이 메서드를 한 번도 호출하지 않는다면 ROMAN 필드는 쓸데없이 초기화된 꼴이다. 메서드가 처음 호출될 때 필드를 초기화 하는 지연 초기화(item83)로 불필요한 초기화를 없앨 수 있지만, 권하지는 않는다. 지연 초기화는 코드를 복잡하게 만드는데, 성능은 크게 개선되지 않을때가 많기 때문이다.(item67)

### 가변 객체라도 새로운 인스턴스가 필요하지 않을 수 있다.
[어탭터](https://refactoring.guru/design-patterns/adapter)는 실제 작업은 뒷단 객체에 위임하고, 자신은 제2의 인터페이스 역할을 해주는 객체다. 어댑터는 뒷단 객체만 관리하면 된다. 즉, 뒷단 객체 외에는 관리할 상태가 없으므로 뒷단 객체 하나당 어댑터 하나씩만 만들어지면 충분하다.
Map 인터페이스의 keySet 메서드는 Map 객체 안의 키 전부를 담은 Set 뷰(어댑터)를 반환 한다. 반환된 Set 인스턴스가 가변이더라도 반환된 인스턴스들은 기능적으로 모두 똑같다. (?) 즉, 반환한 객체 중 하나를 수정하면 다른 모든 객체가 따라서 바뀐다. 모두가 똑같은 Map 인스턴스를 대변하기 때문이다.
[HashMap#keySet()](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/HashMap.java#L914)
```java
public Set<K> keySet() {
    Set<K> ks = keySet;
    if (ks == null) {
        ks = new KeySet();
        keySet = ks;
    }
    return ks;
}

final class KeySet extends AbstractSet<K> {
...
    public final void clear() { HashMap.this.clear(); }
...
}
```
```java
public static void main(String[] args) {
    Map<String, Integer> map = new HashMap<>();
    map.put("A", 0);
    map.put("B", 1);
    map.put("C", 2);

    Set<String> keySet = map.keySet();
    System.out.println(map);  // {A=0, B=1, C=2}
//        keySet.add("D"); // java.lang.UnsupportedOperationException java.base/java.util.AbstractCollection.add(AbstractCollection.java:267);  
    keySet.clear();
    System.out.println(map);  // {}
}
```

### 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.
오토박싱은 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다. 오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다. 의미상 별다를 것 없지만 성능에서는 그렇지 않다(item 61)
```java
private static long sum() {
    Long sum = 0L;
    for(long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i; // sum = Long.valueOf(sum.longValue() + i);
    return sum;
}
```
sum 변수를 long이 아닌 Long으로 선언해서 불필요한 Long 인스턴스가 약 231개나 만들어진다.
**박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자**

### "객체 생성은 비싸니 피해야한다"로 오해하면 안 된다.
요즘 JVM에서는 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 일이 크게 부담되지 않는다. **프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일이다.**
아주 무거운 객체가 아닌 다음에야 단순히 객체 생성을 피하고자 여러분만의 객체 풀을 만들지는 말자. 객체 풀을 만드는 게 나은 예가 있지만 일반적으로 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량을 늘리고 성능을 떨어뜨린다. 요즘 JVM 가비지 컬렉터는 상당히 잘 최적화되어서 가벼운 객체를 다룰 때는 직접 만든 객체 풀보다 훨씬 빠르다.

대조되는 개념 '새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라'(item50)
방어적 복사가 필요한 상황에서 객체를 재사용했을 때의 피해가, 필요 없는 객체를 반복 생성했을 때의 피해보다 훨씬 크다는 사실을 기억하자. 방어적 복사에 실패하면 언제 터져 나올지 모르는 버그와 보안 구멍으로 이어지지만, 불필요한 객체 생성은 그저 코드 형태와 성능에만 영향을 준다.


