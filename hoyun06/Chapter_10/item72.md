### Item72 : 표준 예외를 사용하라

우리가 코드를 재활용하는 것처럼 예외 또한 자바에서 제공하는 표준 예외를 재사용하는 것이 좋다.

###### 표준 예외 재사용 시 얻을 수 있는 이점
- 많은 프로그래머에게 이미 익숙한 규약을 준수하므로 해당 api를 읽고 쓰기 쉬워진다
- 사용하는 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스 로딩 시간도 줄어든다

###### 자주 쓰이는 표준 예외
- IllegalArgumentException
  - 메서드 호출 시 부적절한 인수를 넘기는 경우에 발생한다
- IllegalStateException
  - 특정 객체가 호출된 메서드를 수행하기 적합하지 않은 상태일 때 발생한다
  - ex) 해당 객체가 제대로 초기화되지 않은 경우. 객체가 깨진 경우.
- NullPointerException
  - 특정 객체 참조가 null인 경우에 발생한다
- IndexOutOfBoundsException
  - 특정 시퀀스에서 허용하는 범위를 넘어갈 때 발생한다
- ConcurrentModificationException
  - 단일 스레드에서만 사용하려고 설계한 객체를 여러 스레드에서 동시적으로 수정하려고 할 경우에 발생한다
- UnsupportedOperationException
  - 클라이언트가 요청한 동작을 해당 객체가 지원하지 않는 경우에 발생한다

###### 예외 사용 시 유의할 점
`Exception`, `RuntimeException`, `Throwable`, `Error` 를 직접적으로 사용하지는 말자.  
위 대상들을 추상 클래스라고 여기고 적절히 상속/구현하여 사용하자. 위 대상들은 대부분의 예외 클래스들의
상위 타입이므로 포괄적인 '예외 상황' 그 자체를 나타내서 의미가 불분명하고 테스트하기도 어렵다.

다만 나만의 예외를 확장하는 경우 해당 예외 클래스도 '직렬화'가 가능해야 함을 잊지 말자.