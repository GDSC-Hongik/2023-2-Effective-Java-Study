# Item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

인터페이스는 흔히 계약서라고 불리곤 한다. 실제로 어떻게 그 계약을 수행할 것인지(구현)는 계약자(구현체)에게 달려있고 무엇을 할 수 있는지만 클라이언트(호출자)에게 알려주는 것이다. 책에서는 이것을 '타입을 정의하는 용도' 라고 부른다. 뭔가 직관적이지는 않지만 그러려니 하자. 아무튼, 이렇게 인터페이스를 사용하지 않는다면 그건 인터페이스를 잘못 사용하고 있는 것이다. 

상수 인터페이스가 대표적인 안티 패턴 사례이다.

```java
public interface PhysicalConstants {
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

위의 아보가드로 수에 해당하는 상수는 당연히 내부 구현에 사용될 것이다. 따라서 이러한 상수 인터페이스는 인터페이스가 아니라 구현에 속한 것이므로 올바르지 않. 뿐만 아니라 구현체가 필요하지 않은 상수까지 포함되도록 강제하게 된다. 

상수의 경우 상수 전용 클래스를 만들어서 사용하는 것이 좋다.

```java
public class PhysicalConstants {
    private PhysicalConstants() {} // 인스턴스화 방지

    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

상수 인터페이스에서처럼 자체적으로 인스턴스화가 불가능하면서도 동일하게 기능하도록 만들 수 있다. 만약 상수들 간의 관련성이 높은 경우 열거 타입까지 고려해볼 수 있다. 