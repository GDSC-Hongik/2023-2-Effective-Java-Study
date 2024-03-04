# 예외의 상세 메시지에 실패 관련 정보를 담으라

```
💡 예외는 실패와 관련한 정보를 얻을 수 있는 접근자 메서드를 적절히 제공하는 것이 좋다.
```

## 실패 원인에 대한 정보의 필요성

- 프로그래머가 얻을 수 있는 유일한 정보임
- 그 실패를 재현하기 어려우면 더 자세한 정보를 얻기 어렵거나 불가능
- 사후 분석을 위해 실패 순간의 상황을 포착하여 예외의 상세 메시지에 담자

## 실패 순간 포착하기

### 발생한 예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담자

```
💡 상세 메시지에 비밀번호나 암호 키 같은 정보까지 담아서는 안 된다.
```

- 원인이 모두 다르기 때문에 현상을 보면 무엇을 고쳐야 할지 분석할 때 좋음
- 가독성보다는 담긴 내용이 훨씬 중요
    - 주 소비층 : 문제를 해결해야 할 프로그래머, SRE 엔지니어

### 필요한 정보를 예외 생성자에서 모두 받아 상세 메시지까지 미리 생성하자

- 메시지 만드는 작업 중복 x

```java
/**
 * IndexOutOfBoundsException을 생성한다.
 *
 * @param lowerBound 인덱스의 최솟값
 * @param upperBound 인덱스의 최댓값 + 1
 * @param index 인덱스의 실젯값
 */
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
	// 실패를 포작하는 상세 메시지를 생성한다.
	super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));

	// 프로그램에서 이용할 수 있도록 실패 정보를 저장해둔다.
	this.lowerBound = lowerBound;
	this.upperBound = upperBound;
	this.index = index;
}
```