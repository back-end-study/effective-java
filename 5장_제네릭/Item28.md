# Item28. 배열보다는 리스트를 사용하라


### 1. 배열은 공변이고 제네릭은 불공변이다.
  - Object가 Long의 부모타입이라 Object[]도 Long[] 는 호환된다
  - `List<Object>`는`List<Long>`와 호환되지않는다.

```java
// 런타임 시점에 확인됨(컴파일이 된다는문제)
Object[] array = new Long[1];
array[0] = "타입이 달라 넣을수 없음"; // 런타임 ArrayStoreException 예외
        
// 컴파일 시점에 확인됨
List<Object> list = new ArrayList<Long>(); // 컴파일 에러
list.add("타입이 달라 넣을수 없음");        
```

### 2.  배열은 런타임에도 원소의 타입을 인지하지만 제네릭은 런타임에는 타입정보가 소거(erasure)된다.

### erasure란?
제네릭 도입전 코드와의 **호환성을 유지하기위해** 컴파일러가 필요한곳에 **형변환을 넣어주는 역할을한후, 제네릭타입을 제거한다.**   
즉, 컴파일된 파일(*.class)에는 제네릭 타입에대한 정보가 없다.
```java
// 컴파일 전
List<ClassA> list = new ArrayList<ClassA>();
list.add(new ClassA());
ClassA a = list.get(0);

// 컴파일 후 
List list = new ArrayList();
list.add(new ClassA());
ClassA a = (ClassA)list.get(0);
```

### 3. 제네릭 배열은 허용되지않는다.

#### 제네릭 배열이 허용된다면?

```java

List<String>[] stringLists = new List<String>[1];  //제네릭 배열(컴파일 오류)
List<Integer> intList = List.of(42); // 제네릭

// 제네릭 배열이 가능하다고 했을때,  List<String>[] 타입을 집어넣어, List[] 로 인식하게된다
Object[] objects = stringLists; 

// List[]배열의 첫번째공간에 List<Integer>를 집어넣음  
objects[0] = intList; 

//  stringLists 에는 Integer값이 들어있어 런타임에 ClassCastException 발생        
String s = stringLists[0].get(0);
```

이를 방지하기위해선 제네릭 배열의 컴파일을 막아야한다. 


## 제네릭 사용하자 
#### 제네릭 사용 x

```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        this.choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)]; // 런타임오류 발생
    }

    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();

        Chooser chooser = new Chooser(list);
        Object choose = chooser.choose();
    }
}
```
제네릭 미사용시 컴파일 타입체크가 안되기때문에 런타임 오류를 만날 가능성이 높아진다.

#### 제네릭 사용
- 타입안정성 체크가 가능하다 (런타임 오류를 만날일이 없다)
- 경고나 컴파일오류를 만나면 배열을 리스트로 대체해라
```java
public class Chooser<T> {
    private final List<T> choiceArray;

    public Chooser(Collection<T> choices) {
        this.choiceArray = new ArrayList<>(choices);
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray.get(rnd.nextInt(choiceArray.size()));
    }

    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();

        Chooser chooser = new Chooser(list);
        Object choose = chooser.choose();
    }
}
```


### 정리

- 배열대신 리스트 사용해라   
=> 제네릭을 자연스럽게 같이 사용하게되어 타입안정성을 높힌다.   
=> 불필요한 타입캐스팅을 줄인다.   
