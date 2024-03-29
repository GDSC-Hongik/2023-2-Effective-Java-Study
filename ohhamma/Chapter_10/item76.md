# 가능한 한 실패 원자적으로 만들라

```
💡 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 한다.
```

## **실패 원자적으로 만드는 방법 : 불변 객체**

- 태생적으로 실패 원자적
- 상태가 생성 시점에 고정되어 변하지 x

### 매개변수의 유효성 검사

- 잠재적 예외의 가능성 대부분을 거르는 방법

```java
public Object pop() {
	if (size == 0) {
		throw new EmptyStackException();
	}
	...
}
```

### 실패 가능성이 있는 코드 > 객체 상태를 바꾸는 코드

- ex) `TreeMap`에 원소를 추가하는 경우

### 객체의 복사본에서 작업을 수행한 후, 성공적으로 완료되면 원본과 교체

- 데이터를 임시 자료구조에 저장해 작업하는 것이 더 빠를 때 good
- ex) 원소들을 배열로 옮겨 담은 뒤에 정렬

### 작업 도중 발생하는 실패를 가로채는 복구 코드 작성

- 작업 전 상태로 되돌리기
- ex) 디스크 기반의 내구성을 보장해야 하는 자료구조

## 그러나 항상 달성 가능한 건 아니다

- 두 스레드가 동기화 없이 같은 객체를 동시에 수정하는 경우
- 복구할 수 없는 `AssertionError`인 경우