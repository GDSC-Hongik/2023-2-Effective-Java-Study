# Item 72. 표준 예외를 사용하자

예외도 다른 코드와 마찬가지로 재사용 하는 것이 좋다.  
따라서, 표준 예외를 쓰면 얻는 게 많다.

### 많이 쓰는 표준 예외
- IllegalArgumentException
- IllegalStateException
- NullPointerException
- IndexOutOfBoundsException
- ConcurrentModificationException
- UnsupportedOperationException

### 재사용하면 안 되는 것들
- Exception
- Throwable
- RuntimeException
- Error

이들은 추상 클래스라고 생각하자.