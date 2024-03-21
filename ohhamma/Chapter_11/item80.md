# 스레드보다는 실행자, 태스크, 스트림을 애용하라

## 실행자 서비스

- 인터페이스 기반의 유연한 **태스크 실행** 기능

```java
// 작업 큐
ExecutorService exec = Executors.newSingleThreadExecutor();

// 실행할 태스크 넘기기
exec.execute(runnable);

// 실행자 종료
exec.shutdown();
```

### 주요 기능

- 특정 태스크 완료까지 대기 (`get`)
- 태스크 모음 중 임의의 태스크 or 모든 태스크 완료까지 대기 (`invokeAny`/`invokeAll`)
- 실행자 서비스 종료까지 대기 (`awaitTermination`)
- 완료된 태스크 결과 차례로 받음 (`ExecutorCompletionService`)
- 태스크 주기적으로 or 특정 시간에 실행 (`ScheduledThreadPoolExecutor`)

### ThreadPoolExecutor

- 스레드 풀 동작을 결정하는 거의 모든 속성 설정 가능

### CachedThreadPool

- 작은 프로그이나 가벼운 서버에 적합
    - 요청받은 태스크들이 큐에 쌓이지 x → 즉시 스레드에 위임
- 무거운 프로덕션 서버에 적합한 클래스
    - `FixedThreadPool` : 스레드 개수 고정
    - `ThreadPoolExecutor` : 완전히 통제 가능

## 태스크

- **작업 단위**를 나타내는 핵심 추상 개념
- `Runnable` & `Callable`
    - Callable → 값 반환, 임의의 예외 던질 수 있음
- 실행자 서비스 : 태스크를 수행하는 일반적인 매커니즘
    - 작업 수행 담당

### 포크-조인 태스크

- 포크-조인 풀이라는 실행자 서비스가 실행
- 작은 하위 태스크로 분할 가능
- `ForkJoinPool`을 구성하는 스레드들이 처리
- 일을 먼저 끝낸 스레드 → 다른 스레드의 남은 태스크를 가져와 대신 처리
    - CPU 최대한 활용
    - 처리량↑ 지연시간↓