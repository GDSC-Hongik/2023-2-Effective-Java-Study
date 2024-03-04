# 추상화 수준에 맞는 예외를 던지라

```
💡 아래 계층의 예외를 예방하거나 스스로 처리할 수 없고,
   그 예외를 상위 계층에 그대로 노출하기 곤란하다면 예외 번역을 사용하라.
```

## 예외 번역

- 상위 계층에서 저수준 예외를 잡아 → 자신의 추상화 수준에 맞는 예외로 바꾸어 던짐

```java
try {
	...
} catch (LowerLevelException e) {
	throw new HigherLevelException(...);
}
```

## 예외 연쇄

- 문제의 근본 **원인**인 저수준 예외 → 고수준 예외에 실어 보냄
- 문제의 원인을 프로그램에서 접근할 수 있게 해줌

```java
try {
	...
} catch (LowerLevelException cause) {
	throw new HigherLevelException(cause)
}
```

- 최종적으로 `Throwable(Throwable)` 생성자까지 건네짐

```java
// 예외 연쇄용 생성자
class HigherLevelException extends Exception {
	HigherLevelException(Throwable cause) {
		super(cause);
	}
}
```

## 그러나 남용하는 건 곤란하다

- 아래 계층에서는 예외가 발생하지 않도록 해보자
    - 상위 계층 메서드의 매개변수 값을 아래 계층으로 보내기 전에 미리 검사해보자
- 또는, 상위 계층에서 에외를 조용히 처리하여 문제를 API 호출자에까지 전파하지 말자
    - 이때 발생한 예외는 `java.util.logging`을 활용하여 기록해두자