# Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

의존 객체 주입이란 클래스가 사용하는 외부의 객체를 직접 생성하지 않고, 생성자를 통해 주입받는 방식을 말한다. 이를 통해 클래스는 자신이 사용할 객체를 선택할 수 있게 된다. 또한, 테스트를 위해 가짜 객체를 주입할 수 있게 된다.

가령 아래와 같은 코드가 있다고 하자.

```java
public class Car {
    private final String name;
    private int position = 0;

    public Car(String name) {
        validateName(name);
        this.name = name;
    }

    public void move() {
        if (isMovable()) {
            position++;
        }
    }

    private boolean isMovable() {
        return RandomUtils.nextInt(0, 9) >= 4;
    }
}
```

<!-- 테스트에는 FIRST 원칙이 있다. 그중에서 R, Repeatable은 테스트는 반복 가능해야 한다는 원칙이다. 하지만 `publishLotto()`의 경우 랜덤에 의존하기 때문에, 매번 다른 숫자를 발급한다. 따라서 우리는 로또를 생성할 때 인자로 넘어온 숫자가 겹치지 않는지, 즉 `validateNumbersDistinct()` 가 제대로 동작하는지 테스트할 수 없다. -->

테스트에는 FIRST 원칙이 있다. 그중에서 R, Repeatable은 테스트는 반복 가능해야 한다는 원칙이다. 하지만 `isMovable()`의 경우 `RandomMoveStrategy`에 의존하기 때문에, 매번 다른 값을 리턴한다. 따라서 우리는 `Car`가 제대로 동작하는지 테스트할 수 없다.

테스트를 위해서는 랜덤성을 격리해야 한다. 이때 다양한 방식을 채택할 수 있다. 책에서 언급하는 의존객체 주입이 대표적인 예시이다.

```java
public class Car {
    private final String name;
    private final MoveStrategy moveStrategy;
    private int position = 0;

    public Car(String name) {
        validateName(name);
        this.name = name;
        this.moveStrategy = new RandomMoveStrategy();
    }

    public void move() {
        if (isMovable()) {
            position++;
        }
    }

    private boolean isMovable() {
        return moveStrategy.determineMovable();
    }
}
```

그리고 테스트 시에는 목 이동전략을 주입하여, `isMovable()` 가 제대로 동작하는지 테스트할 수 있다.

```java
class FixedMoveStrategy implements MoveStrategy {
    private List<Boolean> movables;
    private int index = 0;

    public FixedMoveStrategy(List<Boolean> movables) {
        this.movables = movables;
    }

    @Override
    public boolean determineMovable() {
        if (index >= movables.size()) {
            return false;
        }

        return movables.get(index++);
    }
}
```