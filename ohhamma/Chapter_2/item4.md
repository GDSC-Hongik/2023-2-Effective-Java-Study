# 인스턴스화를 막으려거든 private 생성자를 사용하라

## 정적 메서드와 정적 필드만을 담은 클래스

- 기본 타입 값, **배열** 관련 메서드 모음 : `java.lang.Math`, `java.lang.Arrays`
- 특정 인터페이스를 구현하는 객체를 생성하는 **정적 팩터리** 모음 : `java.util.Collections`
- **final** 클래스 관련 메서드 모음
- **유틸리티** 클래스 → 인스턴스로 사용x

```java
public class ConvertUtil {
    private ConvertUtil() {
    }

    public static int convertToDay(final String input) {
        try {
            return Integer.parseInt(input);
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException(ExceptionMessage.INVALID_DATE.getMessage());
        }
    }

    public static int convertToMenuNumber(final String input) {
        try {
            return Integer.parseInt(input);
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException(ExceptionMessage.INVALID_ORDER.getMessage());
        }
    }
}
```

## 기본 생성자 생성 방지

- 컴파일러가 자동으로 기본 생성자 만듦
- 추상 클래스 → 인스턴스화 막을수x
    - 하위 클래스 생성 & 인스턴스화 가능
- `private` 생성자 → 클래스의 인스턴스화 방지 가능
- `AssertionError` : 클래스 내에서 실수로라도 생성자 호출하지 않도록
- 상속 불가능하게 하는 효과 → 하위 클래스가 상위 클래스 생성자에 접근 불가능해짐

```java
public class UtilityClass {
		// 인스턴스화 방지용
		private UtilityClass() {
				throw new AssertionsError();
		}
}
```