# [Item58] 전통적인 for문 보다는 for-each 문을 사용하라

컬렉션을 순회할 때 전통적인 for 문은 다음과 같다.

```java
// 컬렉션 순회하기
for (iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    // e로 무언가를 한다.. 
}

// 배열 순회하기
for (int i = 0; i < a.length; i++) {
    // a[i]로 무언가를 한다..
}
```

우리에게 진짜 필요한 건 **원소**들인데, **반복자와 인덱스 변수는 코드를 지저분**하게 한다.

### Enhanced for statement(향상된 for 문)

java 1.5부터 도입된 **for-each**는 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어서

어떤 컨테이너를 다루는지는 신경쓰지 않아도 된다.

```java
for (Element e : elements) {
    // e로 무언가를 한다..
}
```

### for-each 문은 컬렉션을 중첩해서 사용해야 할 때 이점이 더욱 커진다

아래 코드에서 버그를 찾아보자

```java
public class Card {
    private final Suit suit;
    private final Rank rank;

    enum Suit { CLUB, DIAMOND, HEART, SPADE }
    enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }

    static Collection<Suit> suits = Arrays.asList(Suit.values());
    static Collection<Rank> ranks = Arrays.asList(Rank.values());

    Card(Suit suit, Rank rank ) {
        this.suit = suit;
        this.rank = rank;
    }

    public static void main(String[] args) {
        List<Card> deck = new ArrayList<>();
        
        for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
            for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
                deck.add(new Card(**i.next()**, j.next()));
    }
}
```

위 코드를 실행시켜 보면 `NoSuchElementException`이 발생한다.

그 이유는 중첩된 for문에서 숫자(Suit)하나당 한번씩만 호출되야하는데 카드(Rank) 하나당 한번씩 불리고 있기 때문이다

![https://user-images.githubusercontent.com/68587990/204485593-014a2a95-18d4-4902-99e9-209242f1eeb6.png](https://user-images.githubusercontent.com/68587990/204485593-014a2a95-18d4-4902-99e9-209242f1eeb6.png)

같은 증상으로 아래 예시를 보자

```java
public class DiceRolls {
    enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }

    public static void main(String[] args) {
        // Same bug, different symptom!
        Collection<Face> faces = EnumSet.allOf(Face.class);

        for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
            for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
                System.out.println(i.next() + " " + j.next());
    }
}

// 출력
ONE ONE
TWO TWO
THREE THREE
FOUR FOUR
FIVE FIVE
SIX SIX
```

이 예시에서는 예외를 던지진 않았지만 36개 조합을 출력하려고 했으나, 단 6쌍만 출력하고 끝나버린다

이 문제를 해결하기 위해서는 바깥 for문과 안쪽 for문 사이에 i.next()의 리턴값을 저장하는 별도의 변수를 두어 사용해도 되지만 그닥 좋아보이지 않는다

그러나 for-each문을 사용하면 아주 간단히 해결할 수 있다

```java
for (Face f1 : faces)
    for (Face f2 : faces)
        System.out.println(f1 + " " + f2);
```

훨씬 간결해진다

하지만 for-each문을 항상 사용할 수 있는 것은 아니다

### for-each 문을 사용할 수 없는 3가지 상황

- 파괴적인 필터링(destructive filtering)

컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 `remove` 메서드를 호출해야한다

```java
public class DestructiveFiltering {
    public static void main(String[] args) {
        List<Person> people = new ArrayList<>();

        people.add(new Person("홍길동", 10));
        people.add(new Person("김길동", 20));
        people.add(new Person("최길동", 30));
        people.add(new Person("박길동", 40));
        people.add(new Person("이길동", 50));

        for (Person person : people) {
            if(person.age == 20) {
                people.remove(person);
            }
        }
    }

    @Getter
    @Setter
    private static class Person {
        private String name;
        private int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
    }
}
```

하지만 해당 소스를 실행시켜 보면

![https://user-images.githubusercontent.com/68587990/204488411-4cbb3e9f-3feb-49ea-932c-2648c0707b84.png](https://user-images.githubusercontent.com/68587990/204488411-4cbb3e9f-3feb-49ea-932c-2648c0707b84.png)

`ConcurrentModificationException` 가 발생한다

자바 8부터는 Collection에 default 메서드로 removeIf 가 등장하였는데, 이 메서드를 사용하면 안전하게 컬렉션에서 요소들을 제거할 수 있다

```java
people.removeIf(person -> person.age == 20);
```

Intellij에서도 removeIf를 사용하라고 권장하고 있다

![https://user-images.githubusercontent.com/68587990/204489012-01f3d14c-885a-4eca-a840-a1262ee7ec61.png](https://user-images.githubusercontent.com/68587990/204489012-01f3d14c-885a-4eca-a840-a1262ee7ec61.png)


- 변형(transforming)

리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다

```java
public class Transforming {
    public static void main(String[] args) {
        List<Person> people = new ArrayList<>();
        people.add(new Person("홍길동", 10));
        people.add(new Person("김길동", 20));
        people.add(new Person("최길동", 30));
        people.add(new Person("박길동", 40));
        people.add(new Person("이길동", 50));

        for(Person person : people) {
            if (person.age > 40) {
                person = new Person("수정된 이길동", 50);
            }
        }
        System.out.println("people = " + people);

        for (int i = 0; i < people.size(); i++) {
            if (people.get(i).age > 40) {
                people.set(i, new Person("수정된 이길동", 50));
            }
        }
        System.out.println("people = " + people);
    }

    @Data
    private static class Person {
        private String name;
        private int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
    }
}
// 출력
people = [
Transforming.Person(name=홍길동, age=10), 
Transforming.Person(name=김길동, age=20), 
Transforming.Person(name=최길동, age=30), 
Transforming.Person(name=박길동, age=40), 
Transforming.Person(name=이길동, age=50)
]
people = [
Transforming.Person(name=홍길동, age=10), 
Transforming.Person(name=김길동, age=20), 
Transforming.Person(name=최길동, age=30), 
Transforming.Person(name=박길동, age=40), 
**Transforming.Person(name=수정된 이길동, age=50)**
]
```

`age > 40` 인 경우에 값을 변경하였지만 변경되지 않는다

- 병렬 반복(parallel iteration)

여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 엄격하고 명시적으로 제어해야한다

```java
public static void main(String[] args) {
    List<Integer> dishList = Arrays.asList(1, 2, 3, 4, 5);
    List<Integer> foodList = Arrays.asList(100, 200, 300, 400, 500);
    // 접시 하나에 음식은 하나씩만!

    for (Integer dish : dishList) {
        for (Integer food : foodList) {
            System.out.println("dish : " + dish + " food : " + food);
        }
    }

    System.out.println("-------------------------------------------------------------------");

    for (Iterator<Integer> d = dishList.iterator(), f = foodList.iterator(); d.hasNext() && f.hasNext(); ) {
        System.out.println("dish : " + d.next() + " food : " + f.next());
    }
    // 혹은 index 변수 i 를 두고 접근하는 방법
}

// 출력
dish : 1 food : 100
dish : 1 food : 200
dish : 1 food : 300
dish : 1 food : 400
dish : 1 food : 500
dish : 2 food : 100
     (생략)
dish : 4 food : 500
dish : 5 food : 100
dish : 5 food : 200
dish : 5 food : 300
dish : 5 food : 400
dish : 5 food : 500
-------------------------------------------------------------------
dish : 1 food : 100
dish : 2 food : 200
dish : 3 food : 300
dish : 4 food : 400
dish : 5 food : 500
```

스택오버플로우에 같은 질문이 있었다 [**링크**](https://stackoverflow.com/questions/33208013/parallel-iteration-on-multiple-collections)

### 핵심 정리

for-each 문은 전통적인 for문에 비해 **명료**하고, **유연**하고, **버그를 예방**해주며 **성능 저하도 없다**

**가능한 모든곳에서 for-each문을 사용하자**