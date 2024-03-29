### Item35 : ordinal 메서드 대신 인스턴스 필드를 사용하라

열거 타입은 해당 상수의 순서를 알려주는 ordinal() 메서드를 제공한다.  
하지만 이 메서드는 한계가 명확하다.
- 상수들의 순서가 바뀐다면 한 상수의 ordinal 값이 호출할 때마다 변경된다
- 순서를 기준으로 한 값을 반환하므로 혹시나 여러 상수가 같은 값을 필요로 한다고 해도 지원 불가능하다
- 만약 8개의 상수가 있는 상태에서 12를 나타내는 상수가 필요할 경우 사용되지도 않는 더미 상수를 3개나 추가해야 한다(9, 10, 11)

그러므로 ordinal 메서드를 사용하지 말자.  
그대신 인스턴스 필드를 사용하여 상수에 값을 매칭하자.