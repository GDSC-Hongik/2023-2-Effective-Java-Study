# Enum Validation

스프링 MVC가 제공하는 여러가지 validation 기능이 있다.  
@NotNull, @Size, @Email 등으로 들어오는 값에 대한 검증을 간단하게 처리할 수 있다.  
자바에서 사용 가능한 숫자, 문자열, 날짜 등에 대한 처리는 위의 내용으로 충분하지만, 
Enum에 대한 검증으로는 부족하다.

## dto에서 Enum은 어떻게 받아야 하는가
dto에서 Enum은 그 자체로 받을 수도 있고, String으로 받을 수도 있다.  
만약 아래와 같이 Enum으로 받는다면, Enum에 정의된 값만 받을 수 있게 된다.
```java
public record Dto(Exmaple example) {}
```
하지만, 만약 Enum에 정의된 값이 아닌 다른 값이 들어온다면, 400 Bad Request를 반환하고
`HttpMessageNotReadableException`이 발생한다.

예외 처리가 깔끔하지 못하다.  
어떤 이유로 HttpMessageNotReadableException이 발생했는지 알 수 없기 때문이다.  
정교한 exception handling을 위해서는 EnumValidator를 사용하는 것이 좋다.

## Enum Validation
다른 validation annotation과 마찬가지로 Enum에 대한 validation annotation을 만들어 사용할 수 있다.
```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = EnumValidator.class)
public @interface ValidateEnum {
	// ...
}
```
@Constraint에는 어떤 Validator 클래스를 사용할 것인지 명시할 수 있다.  
`ConstraintValidator` 인터페이스를 구현하여 EnumValidator 클래스를 만들어 사용하면 된다.  

