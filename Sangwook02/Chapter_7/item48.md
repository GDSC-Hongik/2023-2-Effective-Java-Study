# Item 48. 스트림 병렬화는 주의해서 적용하라

자바로 동시성 프로그램을 작성하는 것이 쉬워지고 있지만, 올바르게 사용하도록 노력해야 함.  

### 병렬화로 성능 개선을 할 수 없는 경우
- limit을 중간 연산으로 사용하는 경우
- 데이터 소스가 Stream.iterate인 경우

### 병렬화로 성능 개선을 할 수 있는 경우
- ArrayList, HashMap, HashSet, ConcurrentHashMap, 배열 등 
  - 원소들을 순차적으로 처리할 때 참조 지역성이 뛰어남
    - 참조 지역성: 이웃한 원소들의 참조들이 메모리에 연속해서 저장됨
    - 참조 지역성이 떨어지면 병렬화로 성능 개선을 기대하기 어려움
      - 메인 메모리에서 캐시 메모리로 올 때까지 기다리기만 하기 때문
- 종단 연산이 축소형이거나 조건에 맞으면 바로 반환하는 연산인 경우
  - ex) count, min, max, allMatch, anyMatch, noneMatch 
  - 가변 축소를 수행하는 Stream의 collect 메서드는 부적절
