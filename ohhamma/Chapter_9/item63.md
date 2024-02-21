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