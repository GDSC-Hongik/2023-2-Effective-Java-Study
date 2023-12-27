# try-finally보다는 try-with-resources를 사용하라

- 자바 라이브러리에는 `close` 메서드를 호출해 직접 닫아줘야 하는 자원이 많음
    - 클라이언트가 놓치기 쉬워 예측할 수 없는 성능 문제로 이어짐

## try-finally

- 자원이 제대로 닫힘을 보장하는 수단
    - **BUT** 자원이 둘 이상이면 너무 지저분해진다!

## try-with-resources

- 해당 자원이 `AutoCloseable` 인터페이스를 구현해야함
    - 단순히 void를 반환하는 close 메서드 하나만 정의한 인터페이스
    - 짧고 읽기 수월, 문제 진단하기도 훨씬 좋다
- 숨겨진 예외들 → 스택 추적 내용이 숨겨졌다는 꼬리표를 달고 출력 (Throwable의 getSuppressed)
- `catch`절과 함께 사용 가능