### Item46 : 스트림에서는 부작용 없는 함수를 사용하라

스트림 파이프라인에서 사용되는 함수는 `순수 함수`여야 한다. `순수 함수`란 함수의 출력값이 오직 입력에 의해서만 영향을 받는 함수를 뜻한다.
즉, 다른 가변 상태를 참조하지 않아야 하며 외부의 상태를 변경해선 안된다.  
이렇게 하기 위해선 스트림 파이프라인에 넘기는 함수 객체는 부작용 없는 함수를 사용해야 한다.
```java
// 잘못된 스트림 api 사용
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```
```java
// 잘 사용한 스트림 api
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```
위 잘못된 예시에서는 종단 연산인 forEach에서 연산의 결과를 그저 반환하는 것이 아닌 외부 상태를 변경하는 코드이므로 잘못된 예시라고 볼 수 있다.

###### 수집기
위 잘 사용한 예시 코드를 보면 collect 메서드를 이용해서 스트림 내부 원소들을 하나의 객체로 취합해서 반환했다. 이처럼 여러 원소들을 하나로 '수집'해서 반환해 준다고 하여 스트림에는 
수집기라고 칭하는 개념이 있다. 자바 10까지 해서 Collectors 에는 43개의 메서드가 있다고 하는데 개략적으로 알아보자.

###### toMap
toMap은 스트림을 map으로 취합하는 메서드다.
- toMap(keyMapper, valueMapper)
  - 스트림 원소를 key로 매핑하는 함수와 value로 매핑하는 함수를 각각 매개변수로 받는다.
  - 이 메서드는 스트림의 각 원소가 유일한 key에 대응될 수 있는 경우에 사용할 수 있다.
- toMap(keyMapper, valueMapper, 병합 함수)
  - 이 방식은 스트림 원소들 중에 같은 key값을 공유하는 원소들을 하나의 값으로 합치는 경우에 사용할 수 있다.
  - 책에서는 Album이 담긴 stream을 순회하며 key 값은 해당 앨범의 작곡가로 매핑하고 value 값은 해당 작곡가의 앨범 중 가장 많이 팔린 
  앨범과 매핑하는 용도로 이 메서드를 활용했다.
- toMap(keyMapper, valueMapper, 병합 함수, mapFactory)
  - 여기선 네 번째 매개변수만 추가됐는데 클라이언트가 원하는 map 구현체를 명시할 수 있다.

###### groupingBy
groupingBy는 입력으로 분류 함수를 받고 출력으로는 스트림의 원소들을 해당 분류 함수를 이용하여 각 카테고리로 분류한 map을 반환한다.
- 분류 함수만 매개변수로 받는 groupingBy
  - 가장 기본적인 형태로 반환되는 map의 value값은 List의 형태를 띈다.
- 분류 함수와 다운스트림 수집기를 매개변수로 받는 groupingBy
  - 다운스트림 수집기는 분류 함수에 의해 각 카테고리별로 나뉜 스트림으로부터 값을 만들어내는 역할을 맡고 있다.
  - 예를 들어 groupingBy의 다운스트림 수집기로 toSet() 메서드를 넘기면 원래는 list로 반환되던 map의 value값이 set으로 반환된다.
- 분류 함수와 다운스트림 수집기와 맵 팩터리를 매개변수로 받는 groupingBy
  - 다운스트림 수집기를 받는 groupingBy에 더해 최종 결과본 map의 구현체도 직접 명시할 수 있는 메서드다.
  - 참고로 이 메서드의 경우 맵 팩터리 매개변수가 다운스트림 수집기보다 앞에 와서 `점층적 인수 목록 패턴`을 준수하지 않는다고 한다.

###### partitioningBy
groupingBy와 유사하나 다운스트림 수집기 자리에 predicate을 넘겨서 key가 Boolean인 map을 반환한다.

###### minBy, maxBy
이 메서드들은 Collectors에 정의되어 있으나 수집과는 큰 관련은 없다. 위 두 메서드는 인수로 비교자를 받고 해당 비교자를 이용하여 스트림 내부에서 
최소/최대인 원소를 반환한다.

###### joining
이 메서드는 CharSequence 인스턴스의 스트림에만 적용 가능하다.
- 매개변수 없는 joining
  - 원소들을 concatnate하는 수집기를 반환한다
- delimeter를 매개변수로 받는 joining
  - 원소들을 연결하되 연결 부분에 delimeter에서 명시한 문자를 구분자로 넣어준다
- delimeter와 prefix, suffix를 받는 joining
  - delimeter로 연결된 것에 prefix와 suffix를 붙여주는 수집기를 반환한다
  - 만약 prefix, delimeter, suffix에 ('[', ',', ']') 이렇게 넘기면 최종적으로 "[a, b, c]"와 같이 문자열을 반환한다.

## 핵심 정리
    스트림 파이프라인의 operation에 사용되는 메서드들은 부작용 없는 함수여야 한다.
    스트림은 수집기가 굉장히 중요한데 특히 forEach는 값을 확인할 때만 사용하자.