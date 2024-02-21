# 다른 타입이 적절하다면 문자열 사용을 피하라

```
💡 문자열은 잘못 사용하면 번거롭고, 덜 유연하고, 느리고, 오류 가능성도 크다.
```

## 문자열이 적합하지 않은 사례들

### 다른 값 타입을 대신하는 경우

- 입력받을 데이터가 진짜 문자열일 때만 문자열로 받자
- 기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고, 없다면 새로 하나 작성하자

### 열거 타입을 대신하는 경우

- 상수를 열거할 때는 문자열보다 열거 타입이 낫다

### 혼합 타입을 대신하는 경우

```java
String compoundKey  = className + "#" + i.next();
```

- String이 제공하는 기능에만 의존해야 함
- 차라리 전용 클래스를 새로 만드는 편이 낫다
    - private 정적 멤버 클래스

### 권한을 표현해야 하는 경우

```java
public class ThreadLocal {
	private ThreadLocal() { }  // 객체 생성 불가

	// 현 스레드의 값을 키로 구분해 저장한다.
	public static void set(String key, Object value);

	// (키가 가리키는) 현 스레드의 값을 반환한다.
	public static Object get(String key);
}
```

- ex) 스레드 지역변수 기능을 설계하는 경우
- 문제점 : 스레드 구분용 문자열 키가 전역 이름공간에서 공유됨
    - 의도치 않게 두 클라이언트가 같은 변수를 공유할 가능성
    - 보안 취약

```java
public class ThreadLocal {
	private ThreadLocal() { }  // 객체 생성 불가

	public static class Key {  // 권한
		Key() { }
	}

	// 위조 불가능한 고유 키를 생성한다.
	public static Key getKey() {
		return new Key();
	}

	public static void set(Key key, Object value);
	public static Object get(Key key);
}
```

- 해결법 : 문자열 대신 위조할 수 없는 키 사용 → 권한(capacity)

```java
public final class ThreadLocal<T> {
	public ThreadLocal();
	public void set(T value);
	public T get();
}
```

- 리팩토링 : `Key` → `ThreadLocal`로 변경, 매개변수화하여 타입안정성 확보
- `Key` 자체가 스레드 지역변수가 됨