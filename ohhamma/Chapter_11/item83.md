# 지연 초기화는 신중히 사용하라

```
💡 대부분의 필드는 지연시키지 말고 곧바로 초기화해야 한다.
```

## 지연 초기화

```
💡 필요할 때까지는 하지 말라.
```

- 필드의 초기화 시점 → 그 값이 처음 필요할 때까지 늦추는 기법
- 주로 최적화 용도로 사용
- 클래스와 인스턴스 초기화 시 발생하는 순환 문제 해결
- 양날의 검
    - 클래스/인스턴스 생성 시의 초기화 비용 ↓
    - 지연 초기화하는 필드에 접근하는 비용 ↑
- 그럼에도 필요한 경우
    - 해당 필드를 사용하는 인스턴스의 비율 ↓
    - 해당 필드를 초기화하는 비용 ↑
- 멀티스레드 환경 → 까다롭다 !
    - 필드를 동기화하지 않으면 버그로 이어짐

## 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다

- 인스턴스 필드를 초기화하는 일반적인 방법

```java
private final FieldType field = computerFieldValue();
```

- 인스턴스 필드의 지연 초기화

```
💡 지연 초기화가 초기화 순환성을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용하자.
```

```java
private FieldType field;

private synchronized FieldType getField() {
	if (field == null)
		field = computerFieldValue();
	return field
}
```

## 지연 초기화 홀더 클래스 관용구

```
💡 성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스 관용구를 사용하자.
```

- 클래스는 클래스가 처음 쓰일 때 비로소 초기화됨

```java
private static class FieldHolder {
	static final FieldType field = computerFieldValue();
}

private static FieldType getField() { return FieldHolder.field; }
```

## 이중검사 관용구

```
💡 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하라.
```

- 초기화된 필드에 접근할 때의 동기화 비용 ↓
- 필드 값 동기화 없이 검사 → (초기화되지 않았다면) 동기화하여 검사
    - 필드가 초기화되지 않았을 때만 필드 초기화
    - `volatile`로 선언해야 함

```java
private volatile FieldType field;

private FieldType getField() {
	FieldType result = field;
	if (result != null) // 첫 번째 검사 (락x)
		return result;
		
	synchronized(this) {
		if (field == null) // 두 번째 검사 (락o)
			field = computeFieldValue();
		return field;
	}
}
```

### 변종 : 단일검사 관용구

- 반복해서 초기화해도 상관없는 인스턴스 필드를 지연 초기화하는 경우
- 이중검사에서 두 번째 검사 생략
- volatile 한정자를 없애도 되는 경우 (**짜릿한 단일검사**)
    - 모든 스레드가 필드 값을 다시 계산해도 상관없음
    - 필드 타입이 long과 double을 제외한 다른 기본 타입

```java
private volatile FieldType field;

private FieldType getField() {
	FieldType result = field;
	if (result == null)
		field = result = computeFieldValue();
	return result;
}
```