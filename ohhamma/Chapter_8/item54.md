# null이 아닌, 빈 컬렉션이나 배열을 반환하라

```
💡 null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다. 그렇다고 성능이 좋은 것도 아니다.
```

## null을 반환하는 경우

- 방어코드 추가작성 필요

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contiains(Cheese.STILTON))
	System.out.println("좋았어, 바로 그거야.");
```

## 빈 컨테이너를 반환하는 경우

### 추가적인 비용이 드는가?

- 성능 저하의 주범이라 확인되지 않는 한 성능 차이는 신경 쓸 필요 x
- 빈 컬렉션, 배열 → 새로 할당하지 않고도 반환 가능
    - 매번 똑같은 빈 **불변** 컬렉션을 반환하면 됨

```java
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? Collections.emptyList() 
		: new ArrayList<>(cheesesInStock);
}
```

## 배열을 반환하는 경우

- 정확한 길이(0)의 배열을 반환하기만 하면 됨
- 성능 저하 우려 시 배열을 미리 선언 → 매번 그 배열 반환

```
⚠️ 배열을 미리 할당하면 성능이 나빠진다.
```

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
	return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```