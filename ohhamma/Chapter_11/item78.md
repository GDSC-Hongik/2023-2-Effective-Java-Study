# 공유 중인 가변 데이터는 동기화해 사용하라

```
💡 여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화해야 한다.
```

## 동기화의 기능

```
💡 동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.
```

- **배타적** 실행 : 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게
- 스레드 간 **통신** : 한 스레드가 만든 변화를 다른 스레드에서 확인

## 다른 스레드를 멈추는 방법

```
💡 Thread.stop은 사용하지 말자!
```

- 쓰기 메서드와 읽기 메서드 **모두 동기화**
- `volatile` : 항상 가장 최근에 기록된 값을 읽게 됨을 보장
    - 동기화의 두 효과 중 **통신** 쪽만 지원

```java
public class StopThread {
	private static boolean stopRequested;
	
	private static synchronized void requestStop() {
		stopRequested = true;
	}
	
	private static synchronized boolean stopRequested() {
		return stopRequested;
	}
	
	public static void main(String[] args) throws InterruptedException {
		Thread backgroundThread = new Thread(() -> {
			int i = 0;
			while (!stopRequested())
				i++;
		});
		backgroundThread.start();
		
		TimeUnit.SECONDS.sleep(1);
		requestStop();
	}
}
```

## 안전 문제와 해결법

- 프로그램이 잘못된 결과 계산
- `synchronized` 한정자 붙여서 해결
- `volatile` 제거

## 락 프리 프로그래밍

- `java.util.concurrent.atomic` 패키지
- 원자성(배타적 실행)까지 지원

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generatesSerialNumber() {
	return nextSerialNum.getAndIncrement();
}
```

## 가장 좋은 해결법

- 불변 데이터만 공유하거나 아무것도 공유하지 말자
- **가변** 데이터는 **단일 스레드**에서만 쓰도록 하자

## 사실상 불변 & 안전 발행

- 사실상 불변 : 한 스레드가 데이터를 수정한 후 다른 스레드에 공유할 때 → **객체에서 공유하는 부분만 동기화**
    - 그 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 필요 x
- 안전 발행 : 다른 스레드에 이런 객체를 건네는 행위

### 객체를 안전하게 발행하는 방법

- 객체 → 정적 필드, volatile 필드, final 필드, 보통의 락을 통해 접근하는 필드에 저장
- 동시성 컬렉션에 저장