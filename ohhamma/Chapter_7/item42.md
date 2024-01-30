# 익명 클래스보다는 람다를 사용하라

```
💡 익명 클래스는 함수형 인터페이스가 아닌 타입의 인스턴스를 만들 때만 사용하라
```

## 익명 클래스

- 과거에 함수 객체를 만드는 주요 수단

```java
Collections.sort(words, new Comparator<String>() {
	public int compare(String s1, String s2) {
		return Integer.compare(s1.length(), s2.length());
	}
};
```

- 코드가 너무 긺 → 자바에 적합 x

## 람다식

- aka 함수형 인터페이스
- **추상 메서드 하나**짜리 인터페이스

```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

- 컴파일러가 문맥을 살펴 **타입 추론**

```
💡 타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자
```

### 더 간결하게

- 비교자 생성 메서드 사용

```java
Collections.sort(words, comparingInt(String::length));
```

- List 인터페이스에 추가된 `sort` 메서드 이용

```java
words.sort(comparingInt(String::length));
```

### 그러나..

- 이름이 없고 문서화도 못 함
- 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 → 쓰지 말아야 한다
    - 세 줄 안에 끝내는게 좋다

## 람다로 대체할 수 없는 곳이 있다

```
💡 람다는 함수형 인터페이스에서만 쓰임
```

- 추상 클래스의 인스턴스를 만들 때 → 익명 클래스 사용해야함
- 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때 → 익명 클래스 사용
- 람다는 **자신을 참조할 수 x**
    - 람다에서의 `this` → 바깥 인스턴스

```
💡 람다를 직렬화하는 일은 극히 삼가야 한다 (익명 클래스의 인스턴스도 마찬가지)
   → private 정적 중첩 클래스의 인스턴스를 사용하자
```