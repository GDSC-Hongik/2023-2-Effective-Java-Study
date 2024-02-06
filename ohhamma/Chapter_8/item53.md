# 가변인수는 신중히 사용하라

## 가변인수 메서드

- 명시한 타입의 인수를 **0개 이상** 받을 수 있다
    - 인수 개수가 정해지지 않았을 때 아주 유용
- 인수의 개수와 길이가 같은 배열을 만듦 → 인수들을 배열에 저장, 가변인수 메서드에 건네줌
    - 인수 개수는 런타임에 **배열 길이**로 알 수 있음

### 인수가 1개 이상이어야 하는 경우

- 첫 번째로는 평범한 매개변수를 받고, 가변인수를 두 번째로 받는다

```java
static int min(int firstArg, int... remainingArgs) {
	int min = firstArg;
	for (int arg : remainingArgs)
		if (arg < min)
			min = arg;
	return min;
}
```

## 성능에 걸림돌이 되는 가변인수

- 호출될 때마다 배열을 새로 할당하고 초기화

### 해결법

- ex) 메서드 호출의 95%가 인수를 3개 이하로 사용
    - 마지막 다중정의 메서드가 인수 4개 이상인 5%의 호출 담당
- ex) `EnumSet`의 정적 팩터리 → 열거 타입 집합 생성 비용 ↓

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```