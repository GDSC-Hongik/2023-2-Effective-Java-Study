# 옵셔널 반환은 신중히 하라

```
💡 옵셔널 반환에는 성능 저하가 뒤따르니,
성능에 민감한 메서드는 null을 반환하거나 예외를 던지는 편이 나을 수 있다.
```

## 메서드가 값을 반환할 수 없을 때

```
⚠️ 두 방법 모두 허점이 있다.
```

### 1. 예외를 던진다

- 진짜 예외적인 상황에서만 사용해야 함
- 비용 문제 : 예외 생성 시 스택 추적 전체를 캡쳐

### 2. null을 반환한다

- 별도의 처리 코드 추가
    - 무시하는 경우 `NullPointerException` 발생 가능

## Optional<T>의 등장

- `null`이 아닌 `T` 타입 참조를 하나 담거나, 아무것도 담지 않을 수 있음
- 원소를 최대 1개 가질 수 있는 **불변** 컬렉션
- 예외를 던지는 메서드보다 유연, 편리
- null을 반환하는 메서드보다 오류 가능성 ↓

## Optional 반환

```
⚠️ 옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 말자.
```

- 스트림의 종단 연산 중 상당수가 옵셔널 반환

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
	if (c.isEmpty())
		return Optional.empty();

	E result = null;
	for (E e : c)
		if (result == null || e.compareTo(result) > 0)
			result = Objects.requireNonNull(e);

	return Optional.of(result);
}
```

### 선택 기준

```
💡 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 하는 경우에 옵셔널을 반환한다.
```

- 옵셔널은 검사 예외와 취지가 비슷하다

### 주의할 점

```
⚠️ 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 된다.
```

- 빈 `Optional<List<T>>`를 반환하기보다 빈 `List<T>`를 반환하자
- 성능이 중요한 상황 → 맞지 않을 수 있음
    - 적합한지 세심히 측정해보자

## 반환된 Optional 처리하기

### **기본값** 정해두기

```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

### 원하는 **예외** 던지기

- 실제 예외가 아닌 예외 팩터리 (생성비용x)

```java
Toy myToy = max(toys).orElseThrow(TemperTantrunException::new);
```

### 항상 값이 **채워져** 있다고 가정

- 잘못 판단 시 → `NoSuchElementException` 발생

```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

### isPresent

- 옵셔널이 채워져 있으면 `true`, 비어 있으면 `false` 반환

```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID: " + (parentProcess.isPresent() ?
	String.valueOf(parentProcess.get().pid()) : "N/A"));
```

- 스트림 활용하기

```java
streamOfOptionals
	.filter(Optional::isPresent)
	.map(Optional::get)
```

### stream

- `Optional` → `Stream`으로 변환해주는 어댑터
- 값이 있으면 원소로 담은 스트림으로, 없으면 빈 스트림으로 변환
- `flatMap`와 조합

```java
streamOfOptionals
	.flatMap(Optional::stream)
```

## 기본 타입 전용 옵셔널 클래스

```
⚠️ 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자.
```

- `OptionalInt`, `OptionalLong`, `OptionalDouble`

## Optional의 다른 쓰임

```
💡 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 게 적절한 상황은 거의 없다.
```

- 반환값 이외의 용도로 쓰는 경우는 매우 드묾