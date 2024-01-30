# 람다보다는 메서드 참조를 사용하라

```
💡 메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하라
```

## 메서드 참조

- 함수 객체를 람다보다도 **더 간결하게** 만드는 방법

```java
// lambda
map.merge(key, 1, (count, incr) -> count + incr);

// method reference
map.merge(key, 1, Integer::sum);
```

- 매개변수 수↑ → 메서드 참조로 제거할 수 있는 코드양↑
- 람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없다

## 때론 람다가 메서드 참조보다 간결할 때가 있다

- 메서드와 람다가 같은 클래스에 있는 경우

```java
// method reference
service.execute(GoshThisClassNameIsHumongous::action);

// lambda
service.execute(() -> action());
```

## 메서드 참조의 유형

### 정적 메서드를 가리키는 메서드 참조

```java
// method reference
Integer::parseInt

// lambda
str -> Integer.parseInt(str)
```

### 수신 객체를 특정하는 한정적 인스턴스 메서드 참조

```java
// method reference
Instant.now()::isAfter

// lambda
Instant then = Instant.now(); t -> then.isAfter(t)
```

- 근본적으로 **정적 참조**와 유사
    - 함수 객체가 받는 인수 == 참조되는 메서드가 받는 인수

### 수신 객체를 특정하지 않는 비한정적 인스턴스 메서드 참조

```java
// method reference
String::toLowerCase

// lambda
str -> str.toLowerCase()
```

- 함수 객체를 **적용하는 시점**에 수신 객체를 알려줌
- 수신 객체 전달용 매개변수가 매개변수 목록의 첫 번째로 추가됨
    - 뒤로는 참조되는 메서드 선언에 정의된 매개변수들이 뒤따름
- 주로 스트림 파이프라인에서의 **매핑**과 **필터** 함수에 쓰임

### 클래스 생성자를 가리키는 메서드 참조

```java
// method reference
TreeMap<K,V>::new

// labmda
() -> new TreeMap<K,V>()
```

### 배열 생성자를 가리키는 메서드 참조

```java
// method reference
int[]::new

// lambda
len -> new int[len]
```