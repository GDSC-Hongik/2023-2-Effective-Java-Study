# 스트림에는 부작용 없는 함수를 사용하라

## 스트림 패러다임의 핵심

- 계산을 일련의 **변환**으로 재구성
- 각 변환 단계 → 이전 단계의 결과를 받아 처리하는 **순수함수**여야 함
    - 순수 함수 : 오직 입력만이 결과에 영향을 주는 함수
- 스트림 연산에 건네는 함수 객체는 모두 부작용이 없어야 함

```
💡 forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자
```

```java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
	freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

## Collector 인터페이스

### 스트림의 원소를 컬렉션으로 모으기

```java
List<String> topTen = freq.keySet().stream()
		.sorted(comparing(freq::get).reversed())
		.limit(10)
		.collect(toList());
```

- **toList()**
- **toSet()**
- **toCollection(collectionFactory)**

```
💡 Collectors의 멤버를 정적 임포트하여 쓰면 스트림 파이프라인 가독성이 좋아진다
```

### toMap

```java
toMap(keyMapper, valueMapper)
```

- 가장 간단한 맵 수집기 (인수 **2**개)
- 스트림의 각 원소가 **고유한 키**에 매핑되어 있을 때 적합

---

```java
Map<Artist, Album> topHits = albums.collect(
	toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```

- 더 복잡한 형태 : 인수 **3**개 받음 (`BinaryOperator<U>`)
- 병합(merge) 함수까지 제공 가능
- 어떤 키와 그 키에 연관된 연소들 중 하나를 골라 **연관 짓는** 맵을 만들 때 유용

```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

- 충돌이 나면 마지막 값을 취하는 수집기 만들 때 유용 (last-wrtite-wins)

---

- **4**번째 인수 : 맵 팩터리
- 원하는 특정 맵 구현체 지정 가능 (EnumMap, TreeMap)

```
💡 toConcurrentMap : 병렬 실행된 후 결과 (ConcurrentHashMap 인스턴스 생성)
```

### groupingBy

```java
words.collect(groupingBy(word -> alphabetize(word)))
```

- 입력으로 **분류 함수**를 받음
- 출력으로 원소들을 **카테고리별**로 모아 놓은 맵을 담은 수집기 반환
    - 카테고리 → 해당 원소의 맵 키
- **다운스트림 수집기** : 해당 카테고리의 모든 원소를 담은 스트림으로부터 값 생성
    - `toSet()`, `toCollection(collectionFactory)`
    - `counting()` : 해당 카테고리에 속하는 원소의 개수와 매핑

```java
Map<String, Long> freq = words
				.collect(groupingBy(String::toLowerCase, counting()));
```

- **맵 팩터리** : 맵과 그 안에 담긴 컬렉션의 타입 모두 지정 가능
    - ex) 값이 TreeSet인 TreeMap 반환하는 수집기 생성 가능

```
💡 groupingByConcurrent : 동시 수행 버전 (ConcurrentHashMap 인스턴스 생성)
```

### partitioningBy

- 분류 함수에 `predicate`를 받음
- 키가 `Boolean`인 맵 반환

### 다운스트림 수집기 전용 메서드

```
💡 collect(counting()) 형태로 사용할 일은 전혀 없다
```

- **summing**, **averaging**, **summarizing**
    - 각각 int, long, double 스트림용으로 하나씩 존재
- 다중정의된 **reducing** 메서드들
- **filtering**, **mapping**, **flatMapping**, **collecting**, **AndThen**

### ‘수집’과는 관련 없는 메서드

- `minBy`, `maxBy`
    - 스트림에서 값이 가장 작은/큰 원소를 찾아 반환
- `joining`
    - `CharSequence` 인스턴스의 스트림에만 적용 가능
    - 원소들을 단순히 연결 or 구분문자를 받아 join