# Item 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

public 클래스의 public 필드는 사실상 c의 구조체나 다를 바가 없다. 따라서 이 필드를 private으로 만들고 getter를 통해서 접근하게 해야 한다. 이렇게 하면 필드의 구성이나 반환 방식이 '구현'이 되므로 변경이 발생하더라도 유연하게 교체할 수 있다. 단순히 불변이냐 아니냐를 떠나서 유지보수성과 확장성에 대한 이야기라고 보면 된다.

즉 요약하자면 가변 필드를 직접 노출하면 -> thread-safe 하지 않으므로 위험. 불변 필드를 직접 노출하면 -> thread-safe 한 것과는 관련성이 없지만 '구현'을 숨긴다는 캡슐화 관점에서는 좋지 않으므로 유지보수성 / 확장성 측면에서 나쁨. 이라는 얘기다.