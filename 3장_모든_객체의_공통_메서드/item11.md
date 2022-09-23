# Item11. equals를 재정의하려거든 hashCode도 재정의하라

## Object 의 HashCode 규약

- 애플리케이션이 유지되는 동안 equals비교에 사용되는 정보(핵심필드)가 유지된다면, hashCode는 항상 같은값을 반환해야한다

- **equals() 로 두 객체를 같다고 판단했다면 두 객체의 hashCode는 똑같은 값을 반환해야한다**

- equals()가 두 객체를 다르다고 판단했더라도, hashCode를 각각 다른값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다


## 왜 hashCode도 재정의 해야되는지

- equals() 를 재정의 해서 물리적으로 다른 두 객체를  논리적으로 같다고 할수있다.   
이경우 equals() 로 두객체를 같다고 판단하는데 hashCode는 다른값을 가지게되어 **규약을 어기게된다**


```java
public class PhoneNumber {
    private final int areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        PhoneNumber that = (PhoneNumber) o;
        return areaCode == that.areaCode && prefix == that.prefix && lineNum == that.lineNum;
    }

    public static void main(String[] args) {
        Map<PhoneNumber, String> m = new HashMap<>();
        PhoneNumber phoneNumber = new PhoneNumber(707, 867, 5309);
        PhoneNumber phoneNumber2 = new PhoneNumber(707, 867, 5309);
        System.out.println(phoneNumber2.equals(phoneNumber)); // true

        m.put(phoneNumber, "제니");
        System.out.println(m.get(phoneNumber2)); // null
    }
```


## hashCode 재정의하는 최악의 방법
```java
@Override
public int hashCode() {
   return 42;
}
```
#### 특정 int값을 반환하는 방법

- 불가능한것은 아니지만 성능에 악영향을 준다

- 42라는 똑같은 값만 내주어 해시버킷하나에 노드들이 쌓이게 되어 연결리스트처럼 동작한다
=> 해시코드가 42인 객체들의 노드가 버킷에 쌓이게 되고 연결리스트로 버킷을 관리하게되어 조회시 속도가 O(1) -> O(N)으로 느려진다 

> 자바8부터는 연결리스트의 노드가 8개 이상이되면 그때부터 레드블랙트리로 관리합니다 O(LogN) 


## hashCode 작성요령

> 좋은 해시 함수는 같지 않은 인스턴스에 대해 같지 않은 해시 코드를 생성해야 한다    
이상적으로 해시 함수는 모든 int 값에 균일하지 않은 인스턴스의 합당한 컬렉션을 배포해야 한다. 

- 객체의 핵심필드 하나의 해시코드 값을 구해서 result라는 변수에 초기화시킨다

- 객체의 필드 타입에 따라 Type.hashCode(필드명)으로 수행한다 (Type은 Wrapper class)

- 참조타입인경우 그 참조타입의 hashCode()를 호출한다

- 배열인경우 핵심원소 각각을 필드처럼 다룬다 

위의 과정에서 계산한 해시코드로 result를 갱신한다
> result = 31 * result + c(핵심필드로 계산한 해시코드)

#### 곱숫자가 31인이유

- 홀수이면서 소수이며 빠르게 계산할수있다
  - 31N = 32N - N 으로  32N- N은  (N<<5) - N과 같아 빠르게 계산할수있다. 
  
  
## hashCode 지연초기화  
```java
    private int hashCode;
    
    @Override
    public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            result = Integer.hashCode(areaCode);
            result = 31 * result + Integer.hashCode(prefix);
            result = 31 * result + Integer.hashCode(lineNum);
            hashCode = result;
        }
        return result;
    }
```
- 해시코드를 항상 계산하는것도 비용이기때문에 캐싱방식을 고려할수있다. 

- 위의 코드에선 스레드 안정성까지 고려해야한다.
