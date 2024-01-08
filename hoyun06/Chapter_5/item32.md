### Item32 : 제네릭과 가변인수를 함께 쓸 때는 신중하라

###### 제네릭과 가변인수(varargs)
item 28에서 제네릭은 실체화 불가 타입이라 배열 생성이 불가능하다고 했다.  
하지만 제네릭을 가변인수로 넘기는 것이 가능은 하고, T... arr 과 같이 타입 매개변수를
가변인수로 받는 것도 가능은 하다. 물론 이에 대해 컴파일러는 경고를 발생시킨다(heap pollution).
참 모순이다.  
개발자가 직접 제네릭 배열을 생성하는 것은 허용하지 않으면서 제네릭 varargs 매개변수를 받는 메서드를
선언할 수 있도록 하는 것이 말이다. 책에선 이를 허용한 이유가 '실무에서 유용하므로'이다.  
실제로 자바 라이브러리에도 Arrays.asList(T... a), Collections.addAll(Collection<? super T> c, T... elements) 등
제네릭 varargs 매개변수를 받는 메서드를 선언해뒀다. 당연히 이 메서드들은 타입 안전성이 보장된다.

###### @SafeVarargs
자바 7부터는 `@SafeVarargs` 어노테이션이 추가됐다. 이는 제네릭 가변인수를 받는 메서드 작성자로 하여금 해당 메서드 사용 시
클라이언트 쪽에서 발생하는 경고를 숨길 수 있도록 해줬다. 이 어노테이션이 붙은 코드를 사용하면 컴파일러는 해당 코드는 타입 안전성이 
보장됨을 인지하고 경고를 발생시키지 않는다.  
당연히 이 어노테이션은 타입 안전성이 확실히 보장되는 상황에서만 사용해야 한다.

> 그렇다면 해당 메서드가 안전한지는 어떻게 알 수 있을까?

우선 제네릭 가변인수를 넘기면 해당 varargs를 담는 제네릭 배열이 생김을 인지해야 한다. 그리고
1. 해당 메서드가 이 제네릭 배열에 아무런 값도 저장하지 않는다.
2. 이 제네릭 배열의 참조가 외부로 노출되지 않는다.(신뢰할 수 없는 코드에 노출되지 않는다)

위 2가지 조건을 고려했을 때 비로소 메서드가 안전함을 보장할 수 있다.  
단, 2번에 관해선 예외가 존재하긴 한다.
1. @SafeVarargs가 붙은 메서드라면 제네릭 배열을 넘겨도 괜찮다.
2. 이 제네릭 배열의 내부 원소의 메서드를 호출하는 것은 괜찮다(제네릭 배열 자체를 넘기는 것이 아니라면).

사용 예시
```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
```

###### 다른 방법
@SafeVarargs를 사용하는 것 말고 코드 자체를 수정하면 당연히 컴파일러 경고는 사라진다.  
제네릭 varargs를 받던 메서드의 매개변수를 List 혹은 제네릭 컬렉션으로 수정하면 된다.
```java
static <T> List<T> flatten(List<List<? extends T>> lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
```

## 핵심 정리
    제네릭 varargs를 매개변수로 받는 메서드는 유용하긴 하다. 하지만 해당 메서드가 타입 안전함을 확실히 보장하고
    @SafeVarargs를 붙여 경고도 제거해주자.