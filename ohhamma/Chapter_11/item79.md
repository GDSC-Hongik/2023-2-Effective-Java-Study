# 과도한 동기화는 피하라

```
💡 동기화 영역 안에서의 작업은 최소한으로 줄이자.
```

## 동기화된 영역 안에서 제어를 클라이언트에 양도하면 안 된다

- **외계인** 메서드 : 재정의할 수 있는 메서드, 클라이언트가 넘겨준 함수 객체
    - 통제 불가능
    - 예외 발생, 교착상태, 데이터 훼손 가능
- 재진입이 가능한 자바 언어의 락 → 교착상태 x
    - **BUT** 락이 보호하는 데이터와 관련없는 다른 작업 진행 → 위험하다
    - 교착상태가 될 상황 → **데이터 훼손**으로 변모 가능

## 해결 방법

```
💡 동기화 영역에서는 가능한 한 일을 적게 하자.
```

### 외계인 메서드 호출을 동기화 블록 바깥으로 옮기자

- **열린 호출** : 동기화 영역 **바깥**에서 호출되는 외계인 메서드
    - 실패 방지 효과
    - 동시성 효율 개선

```java
private void notifyElementAdded(E element) {
	List<SetObserver<E>> snapshot = null;
	synchronized(observers) {
		snapshot = new ArrayList<>(observers);
	}
	for (SetObserver<E> observer : snapshot)
		observer.added(this, element);
}
```

### 자바의 CopyOnWriteArrayList를 사용하자

- ArrayList를 구현한 클래스
- 내부를 변경하는 작업 → 항상 깨끗한 **복사본**을 만들어 수행
    - 순회 시 락이 필요 없어 매우 빠르다 !
- **관찰자** 리스트 용도로 최적
    - 수정할 일이 드물고 순회만 빈번히 일어나는 경우

```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
	observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
	return observers.remove(observer);
}

private void notifyElementAdded(E element) {
	for (SetObserver<E> observer : observers) {
		observer.added(this, element);
	}
}
```

## 가변 클래스 작성 시 따라야 할 사항

```
💡 과도한 동기화가 초래하는 진짜 비용 : 경쟁하느라 낭비하는 시간 (지연시간)
```

- 동기화를 하지 x : **외부**에서 알아서 동기화하게 하자
    - `java.util`
- 동기화를 **내부**에서 수행 : 스레드 안전한 클래스로 만들자
    - 외부에서 락을 거는 것보다 동시성을 개선할 수 있을 때에만
    - `java.util.concurrent`
    - 다양한 기법 : 락 분할, 락 스트라이핑, 비차단 동시성 제어 등
- 선택하기 어렵다면 → 동기화하지 말고, 문서에 스레드 안전하지 않다고 명기

### 여러 자바 예시들

- `StringBuffer` → `StringBuilder` (동기화하지 않은 버전)
- `java.util.Random` → `java.util.concurrent.ThreadLocalRandom` (동기화하지 않은 버전)