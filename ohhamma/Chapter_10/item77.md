# 예외를 무시하지 말라

```
💡 검사와 비검사 예외에 동일하게 적용된다.
```
## API 설계자의 목소리를 흘려버리지 말자

- `catch` 블록을 **비워두지 말자**

```java
// 아주 의심스러운 코드다 !
try {
	...
} catch (SomeException) {
}
```

## 예외를 무시해야 하는 경우

- `catch` 블록 안에 그렇게 결정한 **이유를 주석으로 남기자**
- 변수 이름 → `ignored`

```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4;
try {
	numcColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
	// 기본값을 사용한다. (색상 수를 최소화하면 좋지만, 필수는 아니다)
}
```

- ex) `FileInputStream`을 닫을 때
    - 복구 필요 x : 입력 전용 스트림이므로
    - 남은 작업 중단할 필요 x : 필요한 정보는 이미 다 읽었으므로