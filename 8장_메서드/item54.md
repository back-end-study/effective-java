## 아이템 54 null이 아닌, 빈 컬렉션이나 배열을 반환하라

```java
private final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 * 	단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmtpy() ? null
		: new ArrayList<>(cheesesInStock);
}
```

위와 같은 상황은 자주 발생하는 상황이다.

만약 위와같이 `null`을 리턴하게 되면, 해당 메서드 호출 하는 클라이언트는 아래와 같이 
`null`을 처리하는 코드를 추가로 작성해야한다.


```java

//  매우 불편

List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.M\\ozzarell
	...
}
```


### 그렇다고 해서 빈 컨테이너를 할당하면 비용이 많이 들지않나?

1. 성능 저하

성능분석 결과 빈 컨테이너를 할당하는것이 성능 저하의 주범이라고 확인되지 않는다.

3. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환 가능하다.

가능성은 적지만, 사용 패턴에 따라 빈 컬렉션을 할당 하는 것이 성능을 눈에 띄게 떨어뜨릴수도 있다. 이럴때 빈 불변 컬렉션을 반환 하는것이다.
왜냐하면 불변 객체는 불변이기때문에 안전하다

> ex) Collections.emptyList, Collections.emptySet, Collections.emptyMap)

배열을 사용할 때도 마찬가지이다. 절대 `null` 을 반환하지 말고 길이가 0인 배열을 반환한다.

```java
private static final Cheese[] EMTPY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMTPY_CHEESE_ARRAY);
}
```

위와 같이 길이 `0` 짜리 배열을 미리 선언해두고, 매번 그 배열을 반환하면 된다.

### 정리
> null이 아닌 빈 배열이나 컬렉션을 반환해라. null을 반환하는 API는 사용하기도 어렵고 오류 처리 코드도 늘어난다. 심지어 성능이 좋은것도 아니다