# 반환 타입으로는 스트림보다 컬렉션이 낫다

```
💡 컬렉션에 담아 반환
- 반환 전부터 이미 원소들을 컬렉션에 담아 관리하는 경우
- 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적은 경우
    - 아니라면 전용 컬렉션 구현을 고민하라
```

```
💡 스트림 or Iterable 반환
- 컬렉션을 반환하는 게 불가능한 경우
```

## 원소 시퀀스 반환하기

- 자바 7까지는 컬렉션 인터페이스를 사용했음
- 자바 8에 스트림이 도입되면서 복잡해짐

### 스트림과 반복

- 스트림은 반복을 지원하지 x
    - Stream이 Iterable을 확장하지 않기 때문
- 우회로가 존재하나 실전에 쓰기에는 너무 난잡, 직관성↓

## **어댑터 메서드 사용하기**

- `Stream<E>` → `Iterable<E>`로 중개해주는 어댑터

```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
	return stream::iterator;
}
```

- `Iterable<E>` → `Stream<E>`로 중개해주는 어댑터

```java
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
	return StreamSupport.stream(iterable.spliterator(), false);
}
```

### Collection 인터페이스는 반복과 스트림을 동시에 지원한다

- `Iterable`의 하위 타입 & `stream` 메서드 제공

```
💡 원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collections나 그 하위 타입을 쓰는 게 일반적으로 최선이다.
```

- **BUT** 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안 된다
    - 반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 → 전용 컬렉션을 구현해보자
    - ex) 멱집합 → `AbstractList` 이용
- AbstractCollection으로 Collection을 구현하기 어려운 경우 → 스트림이나 Iterable 반환도 OK

## 구현하기 쉬운 쪽 택하기

- 입력 리스트의 모든 부분리스트를 스트림으로 반환

```java
public class SubLists {
	public static <E> Stream<List<E>> of(List<E> list) {
		return IntStream.range(0, list.size())
				.mapToObj(start -> IntStream.rangeClosed(start + 1, list.size())
											.mapToObj(end -> list.subList(start, end)))
				.flatMap(x -> x);
	}
}
```