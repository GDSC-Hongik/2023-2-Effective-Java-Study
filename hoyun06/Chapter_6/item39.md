### Item39 : 명명 패턴보다 애너테이션을 사용하라

###### 명명 패턴
주로 프레임워크에서 사용되는 방식. 프레임워크 내에서 명확히 구분되어야 하는 프로그램 요소에는 정해진 이름을 붙이도록 하는 방식.

ex) JUnit 3 까지는 테스트 메서드를 작성할 때 무조건 이름을 test로 시작하게끔 했다.  
하지만 이런 명명 패턴은 문제가 있다.
- 오타가 나더라도 개발자가 인식하기 어렵다
  - tset 라고 작성하면 해당 메서드는 테스트되지 않으므로 통과가 된 것인지 아예 실행이 되지 않은 것인지 파악하기 힘들다.
- 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다
- 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다

이를 해결하기 위해 JUnit 4 부터는 애너테이션을 도입했다.

###### 애너테이션
저자가 이번 item에서 애너테이션을 설명하기 위해 작성한 커스텀 애너테이션을 보자.
```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```
보면 애너테이션을 정의하는데 또 다른 애너테이션이 사용되었는데 이를 `메타 애너테이션`이라고 한다.

`@Retention(RetentionPolicy.RUNTIME)`
- @Test 애너테이션이 runtime에도 유효해야 함을 나타낸다.

`@Target(ElementType.METHOD)`
- @Test 애너테이션을 사용 가능한 범위를 나타낸다. 여기서는 메서드에만 적용 가능하다.

위 애너테이션은 매개변수를 받지 않는 정적 메서드 용 애너테이션을 선언하고자 한 것이다.  
이번엔 특정 예외를 던져야만 테스트가 통과하도록 지원하기 위해 매개변수를 받는 애너테이션을 보자.
```java
import java.lang.annotation.*;

/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();  // 매개변수
}
```
위 코드를 보면 한 개의 매개변수가 있는데 타입이 `Class<? extends Throwable>`다. 즉, Throwable을 상속하는 클래스 객체다. 다시 말해, 
그 어떤 오류 혹은 에러 타입을 모두 수용할 수 있다. 이 애너테이션의 실제 활용 코드를 보자.

```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // 성공해야 한다.
        int i = 0;
        i = i / i;
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {  // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }  // 실패해야 한다. (예외가 발생하지 않음)
}
```
```java
public static void main(String[] args){
    if (m.isAnnotationPresent(ExceptionTest.class)) {
        tests++;
        try {
            m.invoke(null);
            System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
        } catch (InvocationTargetException wrappedEx) {
            Throwable exc = wrappedEx.getCause();
            Class<? extends Throwable> excType =
            m.getAnnotation(ExceptionTest.class).value();
            if (excType.isInstance(exc)) {
                passed++;
            } else {
                System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc);
            }  
        } catch (Exception exc) {
            System.out.println("잘못 사용한 @ExceptionTest: " + m);
        }
    }
}
```
Sample2 클래스의 메서드를 모두 순회한다. 특정 메서드가 ExceptionTest 애너테이션을 달고 있다면 해당 메서드에서 던지는 예외가 애너테이션의 매개변수 타입인지를 확인하고
테스트 성공 여부를 판단하는 코드다.

매개변수를 배열의 형태로도 받을 수 있다.
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();   // Class 객체의 배열
}

// 실제 사용 예시
@ExceptionTest({ IndexOutOfBoundsException.class, NullPointerException.class })
```

여러 개의 매개변수를 받는 것을 애너테이션으로도 처리할 수 있다. 여러 번 적용하고 싶은 해당 애너테이션에 `@Repeatable` 애너테이션을 적용하면 된다. 단, 이 경우에는 명심해야 할 점이 2가지 있다.
1. `@Repeatable` 애너테이션이 달린 애너테이션의 `컨테이너 애너테이션`을 정의하고 `@Repeatable` 애너테이션의 매개변수로 해당 컨테이너 애너테이션의 클래스를 넘긴다.
2. 해당 `컨테이너 애너테이션`은 `@Repeatable` 애너테이션이 달린 애너테이션의 배열을 반환하는 value() 메서드를 정의해야 한다.

말로만 설명하면 굉장히 어렵다. 코드를 살펴 보자.
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();     
}
```
아래가 컨테이너 애너테이션이다. 내부에 원본 애너테이션의 배열을 반환하는 value 메서드가 있음을 알 수 있다.

## 핵심 정리
    명명 패턴이 꼭 나쁜 것만은 아니지만 그 작업을 애너테이션으로 할 수 있다면 애너테이션을 활용하자

### 어노테이션
소스 코드에 메타데이터를 표시할 수 있는 방법. 정보를 표시한다는 점에 있어 주석과 비슷한 개념이지만 주석보다 조금 더 구조의 틀이 잡혀있다.
어노테이션의 종류는 3가지로 크게 나뉜다.
1. 자바 제공 표준 어노테이션
2. 메타 어노테이션
3. 사용자 정의 어노테이션

###### 자바 제공 표준 어노테이션
말 그대로 자바 진영에서 제공하는 어노테이션이다. 우리가 이전에 배웠던 것들을 조금 살펴보자.
- `@Override`
  - 이번 주차 item에도 나온 어노테이션. 메서드 재정의하겠다고 표시한다.
- `@SuppressWarnings`
  - 제네릭 관련 item에서 나온 어노테이션. 컴파일러에 의해 생성되는 경고를 무시하겠다고 표시한다.
- `@SafeVarargs`
  - 마찬가지로 제네릭 item 가변인수 부분에서 나온 어노테이션. 타입 안전성을 보장한다고 표시한다.
- `@FunctionalInterface`
  - 이번 주차 item에 나온 어노테이션. 함수형 인터페이스임을 표시한다.

###### 메타 어노테이션
'메타'라는 단어로부터 알 수 있듯이 어노테이션을 정의하는 용도로 사용하는 어노테이션. 몇 가지만 살펴보자.
- `@Target`
  - 이번 item에서 나온 어노테이션. 특정 어노테이션이 적용될 수 있는 범위를 표시한다.
- `@Retention`
  - 이번 item에서 나온 어노테이션. 특정 어노테이션의 생명 주기를 표시한다.
- `@Repeatable`
  - 이번 item에서 나온 어노테이션. 특정 어노테이션을 반복 사용할 수 있음을 표시한다.

###### 어노테이션 구조 파악하기(속성)
어노테이션은 속성을 가질 수 있는데 그 구조는 다음과 같다.
```java
public @interface CustomAnnotation {
  
    String message() default "default message value";
}
```
풀어서 설명하면  
**[속성의 타입] [속성의 이름] default [디폴트 값];** 형식이다. 처음 어노테이션을 봤을 땐 속성이 메서드 방식으로 표현되어 있어서
이해하기 어려웠는데 그냥 메서드 하나하나가 속성이라고 이해하면 편할 것 같다.

```java
@Target({METHOD, FIELD}) 
@Retention(RUNTIME)
public @interface ManyToOne {

    Class targetEntity() default void.class;

    CascadeType[] cascade() default {};
  
    FetchType fetch() default FetchType.EAGER;

    boolean optional() default true;
}
```

###### @Retention
위에서 @Retention은 어노테이션들의 생명 주기를 표시한다고 했다. 이는 Retention 어노테이션의 속성과 연관이 있다.
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    RetentionPolicy value();
}
```
```java
public enum RetentionPolicy {
    SOURCE,
    CLASS,
    RUNTIME
}
```
Retention은 value라는 속성을 가지는데 이때 이 value의 타입은 RetentionPolicy enum이다. 각 상수의 특징은 다음과 같다.
- SOURCE : 소스 상에서만 해당 어노테이션 정보를 유지한다. 즉, .class 파일에선 해당 어노테이션은 제거된다.
- CLASS : .class 파일까지 해당 어노테이션이 유지는 되나 런타임에는 해당 어노테이션의 정보에 접근할 수 없다.
- RUNTIME : .class 파일까지 해당 어노테이션이 유지되며 런타임에도 리플렉션을 이용해서 해당 어노테이션의 정보에 접근할 수 있다.

이번 item에서 저자가 작성한 @ExceptionTest 어노테이션의 value 속성을 런타임에도 리플렉션을 통해 접근할 수 있었던 이유가 바로
해당 @ExceptionTest 어노테이션의 Retention이 RUNTIME 이었기 때문.