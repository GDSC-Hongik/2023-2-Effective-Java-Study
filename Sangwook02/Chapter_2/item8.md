# Item 8. finalizer와 cleaner 사용을 피하라

finalizer와 cleaner는 자바가 제공하는 객체 소멸자다.  
하지만, 이 둘 모두의 사용을 지양해야 한다.  
finalizer 나름의 쓰임새가 있지만 책은 '쓰지 말아야' 한다고 한다.  
cleaner는 상대적으로 덜 위험하지만 역시 사용하지 말라고 한다.

### finalizer와 cleaner는 제때 실행되어야 하는 작업을 할 수 없다.  
이 둘이 얼마나 작업을 수행할지는 GC에 달렸고 GC의 구현에 따라 다르기 때문에 예측이 불가능하다.  
자원 회수가 지연되다보면 OOME(OutOfMemortError)가 발생할 수도 있다고 한다.  

### 상태의 영구 수정 작업에 사용 불가
finalizer와 cleaner는 수행 시점이 불명확할 뿐만 아니라, 수행되는지 여부도 보장되지 않는다.

### 성능, 보안 문제
finalizer와 cleaner를 사용하면 try-with-resources 방식보다 50배 가까이 성능이 저하된다.  
finalizer는 보안 문제도 일으킬 수 있으니 사용하지 말아야 한다.

## 대안 : AutoCloseable
finalizer와 cleaner를 대체할 방법은 AutoCloseable을 구현하고 인스턴스의 사용이 끝나면 close 메서드를 호출하는 것이다.  
### 그렇다면 finalizer와 cleaner는 어디에 쓰는가
- 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할
- 네이티브 피어와 연결된 객체