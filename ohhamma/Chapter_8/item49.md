# 매개변수가 유효한지 검사하라

## 오류는 가능한 한 빨리 발생한 곳에서 잡아야 한다

- 메서드 몸체가 실행되기 전에 매개변수를 확인하자
- **생성자** 매개변수의 유효성 검사
    - 클래스 불변식을 어기는 객체가 만들어지지 않게 하는 데 필요

### 매개변수 검사를 제대로 하지 못하면 생기는 문제들

- 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다
- 메서드가 잘 수행되지만 잘못된 결과를 반환할 수 있다
    - 미래의 알 수 없는 시점에 메서드와 관련 없는 오류를 낼 수 있다
- **실패 원자성**을 어기는 결과를 낳을 수 있다

### 예외적인 상황들

- 유효성 검사 비용이 지나치게 높거나 실용적이지 않는 경우
- 암묵적 유효성 검사에 너무 의존했다가는 실패 원자성을 해칠 수 있음

## 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다

- public과 protected 메서드
- `@throws` 자바독 태그 사용

```java
/**
 * (현재 값 mod m) 값을 반환한다. 이 메서드는
 * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
 *
 * @param m 계수(양수여야 한다.)
 * @return 현재 값 mod m
 * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
 */
public BigInteger mod(BigInteger m) {
	if (m.signum() <= 0)
		throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
	... // 계산 수행
}
```

- `java.util.Objects.requireNonNull` : 자바의 null 검사 메서드

## 단언문(assert)을 사용해 매개변수 유효성을 검증할 수 있다

- public이 아닌 메서드
- 자신이 단언한 조건이 무조건 **참**이라고 선언

```java
private static void sort(long a[], int offset, int length) {
	assert a != null;
	assert offset >= 0 && offset <= a.length;
	assert length >= 0 && length <= a.length - offset;
	... // 계산 수행
}
```

### 일반 유효성 검사와 다른 점

- 실패하면 `AssertionError`를 던짐
- 런타임에 아무런 효과도, 아무런 성능 저하도 없다