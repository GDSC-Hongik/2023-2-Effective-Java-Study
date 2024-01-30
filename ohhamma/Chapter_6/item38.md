# 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

```
💡 열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다
```

## 타입 안전 열거 패턴은 확장할 수 있으나 열거 타입은 그럴 수 없다

- 타입 안전 열거 패턴 : 열거된 값에 새로운 값을 추가 → 다른 목적으로 사용 가능
- 열거 타입을 확장하는 것은 좋은 생각이 아님
    - 기반 & 확장 타입 순회 어려움
    - 설계 & 구현 복잡해짐

## 연산 코드

- 확장할 수 있는 열거 타입이 어울리는 쓰임
    - 기본 연산 외에 추가적으로 확장 연산을 열어주는 경우
- 각 원소 → 특정 기계가 수행하는 연산

## 인터페이스를 이용해 확장 가능 열거 타입 흉내내기

- 열거 타입을 확장하지 않고 확장된 인터페이스를 연산 타입으로 사용 가능
    - 인터페이스 : `Operation`
    - 열거 타입 : `BasicOperation`, `ExtendedOperation`
- 자바 라이브러이 예시
    - 인터페이스 : `CopyOption`, `OpenOption`
    - 열거 타입 : `java.nio.file.LinkOption`

### 확장된 열거 타입의 모든 원소 테스트하기

1. `class` 리터럴 넘기기

```java
public static void main(String[] args) {
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]);
	test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(
		Class<T> opEnumType, double x, double y) {
	for (Operation op : opEnumType.getEnumConstants())
		System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

1. 한정적 `와일드카드` 타입 넘기기

```java
public static void main(String[] args) {
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]);
	test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet,
		double x, double y) {
	for (Operation op : opSet)
		System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

- 여러 구현 타입 연산 조합해 호출 가능
- **BUT** 특정 연산에서 EnumSet, EnumMap 사용 x

### 문제점 : 열거 타입끼리 구현 상속 불가

- 아무 상태에도 의존하지 않는 경우 → 디폴트 구현을 인터페이스에 추가
- **BUT** 공유하는 기능이 많아지는 경우 → 도우미 클래스, 정적 도우미 메서드로 분리 필요