# 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

## 자원 의존성

- 사용하는 자원에 따라 동작이 달라지는 클래스 → 정적 유틸리티 클래스, 싱글턴 방식 적합x
- 오류에 취약
- 멀티스레드 환경에 적합x

## 의존 객체 주입

- 인스턴스 생성 시 **생성자**에 필요한 자원을 넘겨주는 방식
- **불변** 보장 → 여러 클라이언트가 의존 객체들을 안심하고 공유
- 생성자 / 정적 팩터리 / 빌더 → 모두 적용 가능
- `유연성` `재사용성` `테스트 용이성`

```java
public class Lotto {
    private final List<Integer> numbers;

    public Lotto(List<Integer> numbers) {
        validateSize(numbers);
        validateDuplicate(numbers);
        validateRange(numbers);
        this.numbers = numbers;
    }
		// ...
}
```

```java
public class Visit {
    private Date date;
    private Order order;

    private Visit(final Date date, final Order order) {
        this.date = date;
        this.order = order;
    }

    public static Visit of(final Date date, final Order order) {
        return new Visit(date, order);
    }
}
```

## 생성자에 자원 팩터리 넘겨주는 방식

- **팩터리 메서드 패턴**
- 팩터리 : 호출 시 특정 타입의 인스턴스 반복 생성해주는 객체
- `Supplier<T>` : 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리

```java
Mosaic create(Supplier<? extends Tile> tileFactory> { ... }
```

## 의존성이 많아지는 경우

- 코드 가독성↓
- 의존 객체 주입 프레임워크 : Dagger, Guice, Spring → 해소 가능