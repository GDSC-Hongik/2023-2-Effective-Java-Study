# Item 81. wait와 notify보다는 동시성 유틸리티를 애용하라

wait와 notify대신 고수준 동시성 유틸리티를 사용하면 까다로운 작업을 간단히 처리할 수 있다.  
### 고수준 유틸리티
1. 실행자 프레임워크
2. 동시성 컬렉션
3. 동기화 장치

---
동시성 컬렉션에서 동시성을 무력화하지 못하므로 여러 메서드를 묶어 호출하는 것이 불가능  
=> 이를 위해 상태 의존적 수정 메서드들이 추가됨

