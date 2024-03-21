# wait와 notify보다는 동시성 유틸리티를 애용하라

```
💡 코드를 새로 작성한다면 wait와 notify를 쓸 이유가 거의(어쩌면 전혀) 없다.
```

## 고수준 유틸리티

- wait와 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자
- 세 범주 : 실행자 프레임워크, 동시성 컬렉션, 동기화 장치

## 실행자 프레임워크

- 유연한 태스크 실행 기능

## 동시성 컬렉션

- 표준 컬렉션 인터페이스 + 동시성
- 각자의 내부에서 동기화 수행 → **동시성 무력화 불가능**
- 외부에서 추가 락 사용 → 오히려 속도 ↓
- 동기화한 컬렉션을 낡은 유산으로 만들어버림
    - `Collections.synchronizedMap`보다는 `ConcurrentHashMap`을 사용하자

### 상태 의존적 수정 메서드

- 여러 기본 동작 → 하나의 원자적 동작으로 몪음
- ex) `Map`의 `putIfAbsent(key, value)`
    - 매핑된 값 없으면 → 새 값 집어넣음
    - 기존 값 있으면 → 그 값 반환
    - 기존 값 없으면 → null 반환

```java
private static final ConcurrentMap<String, String> map = new ConcurrentHashMap<>();

public static String intern(String s) {
	String result = map.get(s);
	if (result == null) {
		result = map.putIfAbsent(s, s);
		if (result == null)
			result = s;
	}
	return result;
}
```

### 작업이 성공적으로 완료될 때까지 차단

- ex) `Queue`를 확장한 `BlockingQueue`의 `take`
    - 큐의 첫 원소 꺼냄
    - 큐가 비었으면 → 새로운 원소 추가까지 대기
    - 작업 큐로 쓰기에 적합 (생산자-소비자 큐)
- 대부분의 실행자 서비스 구현체 → `BlockingQueue` 사용

## 동기화 장치

- 스레드가 다른 스레드 대기 가능
- 스레드 간 작업 조율 가능
- `CountDownLatch`, `Semaphore`
- 가장 강력한 동기화 장치 `Phaser`

### CountDownLatch

- 일회성 장벽
- 하나 이상의 스레드 → 또다른 하나 이상의 스레드 작업 종료까지 대기
- 생성 시 int 값 받음 : `countDown` 호출 횟수
- 어떤 동작들을 동시에 시작 → 모두 완료까지의 시간 계산
- **스레드 기아 교착상태** : 실행자가 동시성 수준만큼의 스레드를 생성하지 못하는 경우

```
💡 시간 간격을 잴 때는 항상 System.nanoTime을 사용하자.
```

```java
public static long time(Executor executor, int concurrency, Runnable action)
	throws InterruptedException {
	CountDownLatch ready = new CountDownLatch(concurrency);
	CountDownLatch start = new CountDownLatch(1);
	CountDownLatch done = new CountDownLatch(concurrency);
	
	for (int i = 0; i < concurrency; i++) {
		executor.execute(() -> {
			// 준비 완료 타이머에 통지
			ready.countDown();
			try {
				// 모든 스레드 준비 완료까지 대기
				start.await();
				action.run();
			} catch (InterruptedException e) {
				Thread.currentThread().interrupt();
			} finally {
				// 작업 종료 타이머에 통지
				done.countDown();
			}
		});
	}
	
	ready.await();
	long startNanos = System.nanoTime();
	start.countDown();
	done.await();
	return System.nanoTime() - startNanos;
}
```

## 레거시 코드를 다뤄야 하는 경우

- `wait` : 어떤 조건이 충족되기를 기다리게 할 때 사용
    - 객체를 잠근 동기화 영역 안에서 호출

```
💡 wait 메서드를 사용할 때는 반드시 wait loop 관용구를 사용하라.
   반복문 밖에서는 절대로 호출하지 말자.
```

```java
synchronized(obj) {
	while (<조건이 충족되지 않았다>)
		obj.wait(); // (락을 놓고, 때어나면 다시 잡는다.)
	
	... // 조건이 충족됐을 때의 동작을 수행한다.
}
```

### 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황

- 스레드가 notify 호출, 대기 중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻음
- 조건이 만족되지 않았는데 다른 스레드가 실수로/악의적으로 notify 호출
- 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll 호출
- 허위 각성 : 대기 중인 스레드가 notify 없이도 깨어나는 경우

### notify vs notifyAll

```
💡 일반적으로 notify보다는 notifyAll을 사용하라.
```

- 깨어나야 하는 모든 스레드가 깨어남을 보장 → 항상 정확한 결과
- 관련 없는 스레드가 실수로/악의적으로 wait를 호출하는 공격으로부터 보호
- 응답 불가 상태에 빠지지 않도록 하자