# 표준 예외를 사용하라

```
💡 예외도 재사용하자. 자바 라이브러리는 대부분 API에서 쓰기에 충분한 수의 예외를 제공한다.
```

## 표준 예외를 사용하면 좋은 점

- 내 API가 다른 사람이 익히고 사용하기 쉬워짐
- 내 API를 사용한 프로그램도 읽기 쉬워짐
- 예외 클래스 수↓ → 메모리 사용량↓ 클래스 적재 시간↓

## 많이 재사용되는 표준 예외들

```
💡 Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말자.
```

### IllegalArgumentException

- 인수로 부적절하거나 허용되지 않는 값을 넘길 때
- null 허용하지 않는 경우 → `NullPointerException`
- 시퀀스의 허용 범위를 넘어서는 경우 → `IndexOutOfBoundsException`

### IllegalStateException

- 객체가 메서드를 수행하기에 적합하지 않을 때

### ConcurrentModificationException

- 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려 할 때

### UnsupportedOperationException

- 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때

## 재사용할 예외를 선택하기 어려운 경우

- 인수 값이 무엇이엇든 어차피 실패했을 거라면 → `IllegalStateException`
- 그렇지 않으면 → `IllegalArgumentException`