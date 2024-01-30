# Item 41. 정의하려는 갓이 타입이라면 마커 인터페이스를 사용하라

마커 인터페이스: 아무 메서드 없이 특정 속성을 가짐을 '표시'하는 인터페이스  
ex) Serializable

마커 annotation이 나왔지만 interface를 완전히 대체 불가

### interface가 더 좋은 점
- interface는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 사용 가능
  - annotation과 달리 컴파일 타임에 타입을 검사 가능
- 적용 대상을 더 정밀히 지정 가능

