# Item 49. 매개변수가 유효한지 검사하라

### 매개변수 유효성 검사를 제대로 하지 않으면 발생하는 문제
1. 메서드가 수행되는 도중에 예외가 발생할 수 있음
2. 잘 실행은 되지만 잘못된 결과를 줌
3. 당장은 알 수 없지만 미래에 런타임 예외를 발생시킴

### assert를 사용하여 매개변수 유효성 검증
**일반 유효성 검사와 다른 점**
1. 실패 시 AssertionError를 던짐
2. 런타임에 아무 영향이 없음

### 메서드가 직접 사용 X, 나중을 위해 저장하는 경우 더 주의하자
- NPE 등 발생해도 추적이 어려움