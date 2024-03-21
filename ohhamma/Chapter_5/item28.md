# 배열보다는 리스트를 사용하라

```
💡 배열과 제네릭에는 매우 다른 타입 규칙이 적용된다.
   둘을 섞어 쓰다가 컴파일 오류를 만나면, 가장 먼저 배열을 리스트로 대체해보자.
```

## 배열 vs 제네릭 타입

- **공변** vs 불공변
    - 배열 : 공변 → 런타임에 타입 호환 오류 발견
    - 제네릭 : 불공변 → 컴파일타임에 타입 호환 오류 발견
- **실체화** 여부
    - 배열 : 실체화 (런타임에 원소 타입 인지)
    - 제네릭 : 원소 타입을 컴파일타임에만 검사

## 배열의 한계

### 제네릭 배열 생성 오류

- 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용 x
- 타입 안전하기 않기 때문

```java
List<String>[] stringLists = new List<String>[1];
List<Integer> intList = List.of(42);
Object[] objects = stringLists;
objects[0] = intList;
String s = stringLists[0].get(0);    // 런타임에 ClassCastException 발생!
```

### 실체화 불가 타입

- `E`, `List<E>`, `List<String>`
- 런타임에 컴파일타임보다 타입 정보를 적게 가지는 타입
- 매개변수화 타입 중 실체화 가능 타입 → 비한정적 와일드카드 타입
    - List<?>, Map<?,?>
- 배열 + 비한정적 와일드카드 타입 → **BUT** 유용하게 쓰이지 x

### 제네릭에서 배열로 반환 불가

- 제네릭 컬렉션 → 자신의 원소타입을 담은 **배열 반환 불가능**
- 제네릭 타입 + 가변인수 메서드 → 경고 메시지
    - 가변인수 매개변수 배열의 원소가 실체화 불가 타입인 경우

## 해결법

- 배열로 형변환 시 제네릭 배열 생성 오류 or 비검사 형변환 경고 뜨는 경우
- `E[]` → `List<E>` 사용하면 해결
- 성능 나빠질 수 있으나, 타입 안정성 & 상호운용성 ↑

```java
public class Chooser<T> {
	private final List<T> choiceList;

	public Chooser(Collections<T> choices) {
		choiceList = new ArrayList<>(choices);
	}

	public T choose() {
		Random rnd = ThreadLocalRandom.current();
		return choiceList.get(rnd.nextInt(choiceList.size()));
	}
}
```