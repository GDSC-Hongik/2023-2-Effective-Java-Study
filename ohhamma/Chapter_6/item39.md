# 명명 패턴보다 애너테이션을 사용하라

```
💡 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다
```

## 명명 패턴

- 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에 적용
- JUnit의 테스트 메서드 이름 : `test`로 시작

### 단점

- **오타**를 허용하지 x
- 올바른 프로그램 요소에서만 사용될거라 **보장 x**
    - 개발자가 의도한 프로그램 수행되지 x
- 프로그램 요소를 매개변수로 전달할 방법 x
    - 기대되는 예외 타입을 테스트 매개변수로 전달해야 하는 경우

## 애너테이션

- JUnit 버전4부터 전면 도입
- 대상 코드의 의미는 그대로 둠

### 마커 애너테이션 타입

- 아무 매개변수 없이 단순히 대상에 marking

```java
import java.lang.annotaion.*;

/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 * 예외 발생 시 해당 테스트를 실패로 처리
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

- 메타애너테이션 : 애너테이션 선언에 다는 애너테이션
    - `@Retention` → 런타임에도 유지되어야함
    - `@Target` → 메서드 선언에만 사용
- 해당 에너테이션에 관심 있는 도구에게 특별할 처리를 할 기회를 줌
    - `m.isAnnotationPresent(Test.class)`

### 매개변수 하나를 받는 애너테이션 타입

- 특정 에외를 던져야만 성공하는 테스트 지원

```java
import java.lang.annotaion.*;

/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 에너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
	Class<? extends Throwable> value();
}
```

- `Throwable`을 확장한 클래스의 Class 객체 → 모든 예외 & 오류 타입 수용
    - @ExceptionTest(ArithmeticException.class)
- 애너테이션 매개변수 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인
    - `excType.isIntance(exc)`

### 배열 매개변수를 받는 애너테이션 타입

- 예외를 여러 개 명시하고 그중 하나가 발생하면 성공

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
	Class<? extends Throwable>**[]** value();
}
```

### 반복 가능한 애너테이션 타입

- `@Repeatable` 메타애너테이션
- 하나의 프로그램 요소에 여러 번 달 수 있음
    - `컨테이너 애너테이션` 타입 → 여러 개 달았을 때와 하나만 달았을 때를 구분
    - `getAnnotationsByType` vs `isAnnotationPresent`

```java
// 반복 가능한 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
	Class<? extends Throwable> value();
}

// 컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
	ExceptionTest[] value();
}
```

- @Repeatable을 단 애너테이션을 반환하는 **컨테이너 애너테이션** 정의,
    - @Repeatable에 컨테이너 애너테이션의 **class 객체**를 매개변수로 전달해야함
- 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 **value 메서드** 정의해야함
- 컨테이너 애너테이션 타입에 적절한 보존 정책(**@Retention**), 적용 대상(**@Target**) 명시해야함

```
💡 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다
```