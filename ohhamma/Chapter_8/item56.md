# 공개된 API 요소에는 항상 문서화 주석을 작성하라

```
💡 정말 잘 쓰인 문서인지를 확인하는 유일한 방법은
자바독 유틸리티가 생성한 웹페이지를 읽어보는 길뿐인다.
```

## Javadoc

- 코드 변경 시 API 수정해줌
- **문서화 주석** : 소스코드 파일에서 기술된 설명을 추림 → API 문서로 변환
- 문서화 주석 안의 `HTML` 요소들이 최종 HTML 문서에 반영됨

## 문서화 주석 작성 규칙

```
💡 API를 올바로 문서화하려면
공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.
```

### 메서드용 문서화 주석 - 해당 메서드와 클라이언트 사이의 규약

- **what** : 어떻게 동작하는지가 아니라 무엇을 하는지를 기술하자
- **전제조건** : 메서드를 호출하기 위한 전제조건 모두 나열하자
    - `@param` 태그를 이용해 그 조건에 영향받는 매개변수에 기술
- **사후조건** : 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건 모두 나열하자
- **부작용** : 사후조건으로 명확히 나타나지 않지만 시스템의 상태에 어떠한 변화를 가져오는 것
    - ex) 백그라운드 스레드를 시작시키는 메서드

---

- 모든 매개변수에 `@param` 태그 달기
    - 명사구 / 산술 표현식
- 반환 타입이 `void`가 아니라면 `@return` 태그 달기
    - 명사구 / 산술 표현식
- 발생 가능성이 있는 모든 예외에 `@throws` 태그 달기
    - if로 시작해 해당 예외를 던지는 조건 설명

```java
/**
 * Returns the element at the specified position in this list.
 * <p>This method is <i>not</i> guaranteed to run in constant
 * time. In some implementations it may run in time proportional
 * to the element position.
 *
 * @param index index of element to return; must be
 *        non-negative and less than the size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException if the index is out of range
 *         ({@code index < 0 || index >= this.size()})
 */
E get(int index);
```

- `{@code}`
    - 태그로 감싼 내용을 코드용 폰트로 렌더링
    - 태그로 감싼 내용에 포함된 HTML 요소나 다른 자바독 태그 무시
    - 여러줄로 된 코드 예시 → `<pre>` 태그로 감싸자
- `this` : 호출된 메서드가 자리하는 객체

### 상속용으로 설계된 클래스 - 자기사용 패턴

- `@implSpec` : 해당 메서드와 하위 클래스 사이의 계약 설명
- 하위 클래스들이 메서드를 상속하거나 `super`로 호출했을 때의 동작방식 기술

```java
/** Returns true if this collections is empty.
 *
 * @implSpec
 * This implementation returns {@code this.size() == 0}.
 *
 * @return true if this collection is empty
 */
public boolean isEmpty() { ... }
```

### HTML 메타문자 포함하기

- `{@literal}` : HTML 마크업이나 자바독 태그 무시
- 코드 폰트로 렌더링하지는 x

```java
/* A geometric series coverages if {@literal |r| < 1}. */
```

### 각 문서화 주석의 첫 번째 문장

```
💡 한 클래스(혹은 인터페이스) 안에서 요약 설명이 똑같은 멤버(혹은 생성자)가 둘 이상이면 안 된다.
```

- 해당 요소의 요약 설명
    - 메서드와 생성자 → 동작을 설명하는 (주어가 없는) **동사구**
    - 클래스, 인터페이스, 필드 → 대상을 설명하는 **명사절**
- 다중정의 메서드 → 특히 조심하자
- 마침표(`.`)에 주의하자
    - 의도치 않은 마침표를 포함한 텍스트 → `{@literal}`로 감싸주자

```java
/**
 * A suspect, such as Colonel Mustard or {@literal Mrs.Peacock}.
 */
public class Suspect { ... }
```

### 제네릭 타입, 제네릭 메서드 - 모든 타입 매개변수에 주석

```java
/**
 * An object that maps keys to values. A map cannot contain
 * duplicate keys; each key can map to at most one value.
 *
 * (Remainder omitted)
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface Map<K, V> { ... }
```

### 열거 타입 - 상수들에도 주석

- 열거 타입 자체와 그 열거 타입의 public 메서드에도 적용

```java
/**
 * An instrument section of a symphony orchestra.
 */
public enum OrchestraSection {
	/** Woodwinds, such as flute, clarinet, and oboe. */
	WOODWIND,

	/** Brass instruments, such as french horn and trumpet. */
	BRASS,

	/** Percussion instruments, such as timpani and cymbals. */
	PERCUSSION,

	/** Stringed instruments, such as violin and cello. */
	STRING;
}
```

### 애너테이션 타입 - 멤버들에도 모두 주석

- 필드 설명 → 명사구
- 애너테이션 타입 설명 → 이 애너테이션을 단다는 것이 어떤 의미인지를 설명하는 동사구

```java
/**
 * Indicates that the annotated method is a test method that
 * must throw the designeated exception to pass.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
	/**
	 * The exception that the annotated test method must throw
	 * in order to pass. (The test is permitted to throw any
	 * subtype of the type described by this class object).
	 */
	Class<? extends Throwable> value();
}
```

### 패키지 설명

- `[package-info.java](http://package-info.java)` 파일에 작성
- 패키지 선언 관련 애너테이션을 추가로 포함 가능
- 모듈 시스템 → `[module-info.java](http://module-info.java)` 파일에 작성

### 스레드 안전성과 직렬화 가능성

```
💡 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.
```

- 직렬화 가능 클래스 → 직렬화 형태도 API 설명에 기술

## HTML 문서 검색 기능의 도입

- 키워드를 입력하면 → 관련 페이지들이 드롭다운 메뉴로 나타남
- 원한다면 `{@index}` 사용 → 중요한 용어 추가로 색인화

```java
/* This method compiles with the {@index IEEE 754} standard. */
```

## 메서드 주석 상속

- 문서화 주석이 없는 API 요소 발견 → 가장 가까운 문서화 주석 찾아줌
    - 상위 클래스보다 인터페이스를 먼저 찾음

### 문서화 주석 일부 상속

- `{@inheritDoc}`
- 인터페이스의 문서화 주석 → 클래스에서 재사용 가능