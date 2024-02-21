# 문자열 연결은 느리니 주의하라

```
💡 많은 문자열을 연결할 때는 문자열 연결 연산자(+)를 피하자.
```

### 문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례한다.

- 성능을 포기하고 싶지 않다면 `String` 대신 `StringBuilder`를 사용하자

```java
public String statement2() {
	StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
	for (int i = 0; i < numItems(); i++)
		b.append(lineForItem(i));
	return b.toString();
}
```

- statement2의 수행 시간 → **선형**으로 증가 !

## 사용할 수 있는 방법들

### StringBuilder / StringBuffer

- `StringBuffer` : 멀티스레드로 동시에 `StringBuilder`에 여러 스레드에서 접근해야 하는 경우

### Stream

```java
Arrays.stream(arr).collect(Collectors.joining());
```

- `joining`도 내부적으로 `StringBuilder` 사용

### StringJoiner

```java
new StringJoiner(",", "[", "]");
```

- `delimiter`, `prefix`, `suffix`를 붙여서 join 가능

### String.join

```java
String.join("", arr);
```

- 내부적으로 `StringJoiner` 사용

## 참고자료

[자바에서 문자열 합칠 때 '+' 연산을 쓰지 마세요! (StringBuilder, StringJoiner, String.join, StringBuffer)](https://nahwasa.com/entry/자바에서-String에-대한-연산을-쓰지-마세요-StringBuilder-StringBuffer)