### Item64 : 객체는 인터페이스를 사용해 참조하라

###### 인터페이스를 사용하면 더 좋은 점
인터페이스를 사용하여 객체를 참조하면 해당 인터페이스의 구현체라면 어느 것이든지 참조할 수 있다는 장점이 있다. 그래서
기존에 사용하던 구현체보다 성능이 더 뛰어난 구현체를 개발하여 사용하고자 할 때 구현체를 생성하여 반환하는 부분의 코드만 약간 수정하면
기존 코드는 정상적으로 동작한다. 다만 기존의 구현체가 제공하는 특별한 기능이 있었을 경우 새로운 구현체 또한 이 기능을 포함하여 제공해야 한다.

###### 인터페이스를 사용하여 참조하지 않아도 되는 경우
- 적합한 인터페이스가 없는 경우
  - 특정 객체를 가리킬 수 있는 적합한 인터페이스가 없는 경우 당연히 구체 타입으로 참조를 해야 한다. 그렇지만 이 경우에도 최대한 덜 구체적인(ex. 추상 클래스) 클래스로 참조할 수 있는지
  확인하고 있다면 그것을 사용하는 것이 좋다. 즉, 최대한 덜 구체적인 타입으로 참조하는 것이 바람직하다는 것이다
- 특정 구현체가 제공하는 기능을 사용해야 하는 경우
  - 인터페이스에서 제공하는 스펙에 더해서 구현체 클래스가 추가적으로 제공하는 기능을 반드시 사용해야 하는 경우에는 당연히
  구체 클래스를 이용하여 참조해야 한다