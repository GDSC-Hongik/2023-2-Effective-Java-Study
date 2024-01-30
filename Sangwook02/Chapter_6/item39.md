# Item 39. 명명 패턴보다 애너테이션을 사용하라

## 명명 패턴의 단점
- 오타가 나면 안 됨
- 올바른 프로그램 요소에만 사용되리라는 보장이 없음
- 프로그램 요소를 매개변수로 전달할 수 없음  
=> 명명 패턴 대신 annotation을 사용함으로써 개선 가능

## @Repeatable 사용
### 주의할 점
- 컨테이너 애너테이션을 하나 더 정의하고 @Repeatable에 class 객체를 전달해야 함
- 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 함
- 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)을 명시해야 함
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```
