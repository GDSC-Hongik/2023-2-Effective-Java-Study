# 스트림은 주의해서 사용하라

```
💡 스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라
```

## 스트림 API의 도입

- 다량의 데이터 처리 작업(순차적, 병렬적)에 유용
- fluent API : 메서드 연쇄 지원
    - 단 하나의 표현식으로 완성 가능

### 스트림

- 데이터 원소의 유한 or 무한 **시퀀스**

### 스트림 파이프라인

- 이 원소들로 수행하는 **연산** 단계
- **소스 스트림**으로 시작 → (하나 이상의 **중간 연산**) → **종단 연산**으로 끝남
    - 중간 연산 : 스트림을 어떠한 방식으로 변환
    - 종간 연산 : 스트림에 최후의 연산 가함
- **지연 평가**됨 (lazy evaluation)
    - **종단 연산**이 호출될 때 평가 이루어짐
- 기본적으로 **순차적** 수행
    - 병렬로 실행하려면? parallel 메서드 호출

## 스트림을 언제 써야 하는가

```
💡 스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다
```

```java
public class Anagram {
	public static void main(String[] args) throws IOException {
		Path dictionary = Paths.get(args[0]);
		int minGroupSize = Integer.parseInt(args[1]);
	
		try (Stream<String> words = Files.lines(dictionary)) {
			words.collect(groupingBy(word -> alphabetize(word)))
					.values().stream()
					.filter(group -> group.size() >= minGroupSize)
					.forEach(g -> System.out.println(g.size() + ": " + g));
		}
	}
	// alphabetize 메서드 코드 생략
}
```

```
💡 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다
```

- 스트림을 적절히 활용하면 깔끔하고 명료해진다
    - **도우미 메서드**를 적절히 활용하자
- **char** 값들을 처리할 때는 스트림을 **삼가**는 편이 낫다
- 기존 코드는 스트림을 사용하도록 리팩터링하되, 새 코드가 더 나아 보일 때만 반영하자
- 스트림을 반환하는 메서드 이름은 원소의 정체를 알려주는 **복수 명사**로 쓰자

### 함수 객체로는 할 수 없지만 코드 블록으로는 할 수 있는 일들

```
💡 계산 로직에서 아래의 일들을 수행해야 한다면 스트림과 맞지 않는 것이다
```

- 범위 안의 지역변수를 읽고 수정 가능
- return 문을 사용해 메서드에서 빠져나갈 수 있음
- break, continue문 사용 가능
- 메서드 선언에 명시된 검사 예외를 던질 수 있음

### 스트림을 써야 하는 경우

- 원소들의 시퀀스를 일관되게 변환한다 (**transform**)
- 원소들의 시퀀스를 필터링한다 (**filter**)
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다 (**merge**)
- 원소들의 시퀀스를 컬렉션에 모은다 (**collect**)
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다 (**find**)

### 스트림과 반복 중 어느 쪽을 써야 할지 바로 알기 어려운 경우

- `flatMap` : 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음 다시 하나의 스트림으로 합침
    - 평탄화(flattening)

```java
private static List<Card> newDeck() {
	return Stream.of(Suit.values())
			.flatMap(suit -> Stream.of(Rank.values())
									.map(rank -> new Card(suit, rank)))
			.collect(toList());
}
```