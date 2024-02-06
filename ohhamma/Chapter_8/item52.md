# 다중정의는 신중히 사용하라

```
💡 재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택된다
```

## 다중정의가 혼동을 일으키는 상황을 피해야 한다

- 헷갈릴 수 있는 코드는 작성하지 않는 게 좋다
- API 사용자들에게 고통을 주지 말자

### 매개변수 수가 같은 다중정의는 만들지 말자

- 가변인수(varargs)를 사용하는 메서드 → 다중정의를 아예 하지 말아야 한다

### 다중정의하는 대신 메서드 이름을 다르게 지어주자

- ObjectOutputStream 클래스
    - `writeBoolean(boolean)` / `readBoolean()`
    - `writeInt(int)` / `readInt()`
    - `writeLong(long)` / `readLong()`

### 생성자는 정적 팩터리라는 대안을 활용하자

- 애초에 생성자는 재정의할 수 없음 → 다중정의와 혼용될 걱정 x

## 매개변수 중 하나 이상이 “근복적으로 다르다”면 헷갈릴 일이 없다

- 두 타입의 값을 서로 어느 쪽으로든 형변환할 수 없다
- 그러면 매개변수들의 **런타임 타입**만으로 결정됨

### 오토박싱의 도입

- `set.remove(i)` vs `list.remove(i)`
    - remove(**Object**) vs remove(**int** index)
- `List` 인터페이스가 취약해짐

### 람다와 메서드 참조의 도입

```
💡 암시적 타입 람다식이나 부정확한 메서드 참조 같은 인수 표현식은
   목표 타입이 선택되기 전에는 그 의미가 정해지지 않기 때문에 적용성 테스트 때 무시된다.
```

- `System.out::println` : 부정확한 메서드 참조
- 다중정의 해소 알고리즘이 우리의 기대처럼 동작하지 않음

### 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안 된다

- 서로 근본적으로 다르지 않기 때문

### 관련 없는 클래스들끼리도 근본적으로 다르다

- “관련 없다” : 상위/하위 관계가 아닌 두 클래스
    - String과 Throwable

## 이미 만들어진 클래스가 끼어드는 경우

- `String` : contentEquals(StringBuffer) vs contentEquals(CharSequence)
- 결국엔 어떤 다중정의 메서드가 불리는지 몰라도 기능이 똑같다면 신경 쓸 게 없다
- 상대적으로 더 특수한 다중정의 메서드 → 덜 특수한(더 일반적인) 다중정의 메서드로 일을 넘겨버리자

```java
public boolean contentEquals(StringBuffer sb) {
	return contentEquals((CharSequence) sb);
}
```