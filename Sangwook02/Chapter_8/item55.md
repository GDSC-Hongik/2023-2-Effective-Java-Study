# Item 55. 옵셔널 반환은 신중히 하라

### [자바 8 이전] 메서드가 값을 반환할 수 없는 상황
1. 예외 던지기
   - 스택 추적 전체를 캡쳐하므로 비용이 크다.
2. null 반환하기
   - NPE 발생 가능성

### [자바 8 이후] Optional<T> 클래스
- null이 아닌 T 타입 참조를 하나 담거나
- 아무것도 담지 않을 수 있음
- 최대 하나의 원소를 가지는 불변 컬렉션

##### 사용시 주의사항
1. Optional.of()에 null을 넣으면 NPE => Optional.ofNullable() 사용
2. 메서드가 Optional을 반환할 때, 클라이언트는 값을 받지 못한 경우에 대비해야.
   - 기본 값을 설정
   - 상황에 맞는 예외
   - 옵셔널에 항상 값이 채워져 있다고 확신한다면 바로 꺼내도 무방


### Performance
- 성능 저하가 있을 수 있음
- 박싱된 타입을 담는 옵셔널 -> 두 겹이므로 성능 저하