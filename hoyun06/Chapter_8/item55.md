### Item55 : 옵셔널 반환은 신중히 하라

###### 자바 8 이전
자바 8 이전에는 메서드가 특정 조건에서 값을 반환할 수 없는 경우에 취할 수 있는 방법이 2개 있었다
- 예외 던지기
- null 반환하기

하지만 `예외 던지기`는 스택 트레이스 전체를 캡쳐하므로 비용이 꽤 들었고 null 반환도 이전 item에서 언급한 것과 같이
추가적인 처리 코드가 필요했다.

###### 자바 8 에서의 옵셔널
그래서 위 방법을 대체할 수 있는 방법으로 자바 8부터는 `옵셔널`이라는 수단이 생겼다.  
`Optional<T>` 는 T 타입의 인스턴스를 담을 수 있는데 만약 해당 인스턴스가 존재한다면 그대로 담고 존재하지 않는 경우에는(ex. null)
`빈 Optional`을 유지하게 된다. 그렇기 때문에 메서드에서 특정 조건에서 값을 반환할 수 없는 경우 반환 타입을 `Optional<T>` 로 설정하면
안전하게 결과를 반환할 수 있다. 

###### Optional 관련 메서드
- **Optional.empty()** : 빈 Optional 을 반환한다. 반환값이 null인 경우 빈 Optional을 반환하기 위해 사용한다
- **Optional.of(value)** : value 값이 확실히 존재할 때 해당 값을 Optional로 감싸기 위해 사용한다. 이 경우 value가 null이면 NPE가 발생한다.
- **Optional.ofNullable(value)** : value값이 존재할 수도 있고 존재하지 않을 수도 있는 경우 Optional로 감싸서 반환할 때 사용할 수 있다.
- **.orElse(value)** : 만약 Optional로 반환한 값이 비어있다면 기본값으로 사용할 값을 설정할 수 있는 메서드다.
- **.orElseThrow(Supplier)** : 만약 Optional로 반환한 값이 비어있다면 던질 예외를 설정할 수 있는 메서드다.
- **.get()** : Optional로 반환한 값이 존재함이 분명하다면 이 메서드를 이용해서 안에 담긴 값을 꺼낼 수 있다.
- **Optional.isPresent()** : Optional에 값이 담겨있는지 확인하는 용도로 사용할 수 있다.

###### Optional 사용 시 주의할 점
- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다
- 박싱된 기본 타입을 담는 옵셔널은 컨테이너화를 2번 시키는 것이므로 성능이 그리 좋지 않을 수 있다
  - 그 대안으로 int, long, double에 대해선 OptionalInt, OptionalLong, OptionalDouble을 제공하니 대신 사용할 수 있다 
- 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하지 않는 것이 좋다

## Optional의 orElse~
Optional은 여러 orElse~ 메서드들을 제공한다. 각각이 어떤 역할을 하는지 살펴보고 이를 사용할 때의 주의할 점을 알아보자.

###### orElse
orElse는 해당 Optional에 담긴 value가 null인지 아닌지에 따라 반환하는 값이 달라지는 메서드인데
메서드 세부 사항은 다음과 같다
```java
public T orElse(T other) {
    return value != null ? value : other;
}
```
이때 value는 Optional 안에 들어있는 값이다.  
`value`가 `null`이 아니라면 -> `value`값 그대로 반환  
`null`이라면 -> 매개변수로 받은 `other` 값을 그대로 반환  
여기서 매개변수로 받는 것이 `실제 T 타입`임을 기억하면 좋을 것 같다.

###### orElseGet
orElse와 하는 일은 비슷한 메서드인데 메서드 시그니처와 내부 구현이 약간 다르다.
메서드 세부 사항은 다음과 같다
```java
public T orElseGet(Supplier<? extends T> supplier) {
    return value != null ? value : supplier.get();
}
```
마찬가지로 value가 null인지 아닌지에 따라 반환값이 달라지는데 여기선 매개변수로 
`Supplier<? extends T>` 를 받는 사실을 기억하면 좋을 것 같다.  
**약간의 힌트를 주자면 해당 supplier는 value가 null인 경우에만 실행된다.**

###### orElseThrow
이름 그대로 이해하면 될 것 같다. 이 메서드는 2개의 메서드가 오버로딩 돼있는데 먼저 매개변수가 없는 메서드를 보자. 메서드 세부 사항은 다음과 같다.
```java
public T orElseThrow() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}
```
Optional에 담긴 값이 null이라면 exception을 던지는데 자체적으로 `NoSuchElementException`을 생성해 던진다. 

다음으로는 매개변수를 한 개 받는 메서드를 보자. 메서드 세부 사항은 다음과 같다.
```java
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
    if (value != null) {
        return value;
    } else {
        throw exceptionSupplier.get();
    }
}
```
하는 기능은 동일하나 차이점이라고 한다면 매개변수로 Supplier를 받는 것인데 이때 타입 매개변수는 `Throwable을 상속한 것`이라면 그 어떤 것도 가능하다(한정적 와일드 카드). 

###### orElse와 orElseGet 사용 시 주의할 점
아까 위에서 잠시 언급했지만 `orElse`는 매개변수로 실제 T 타입의 값을 받고 `orElseGet`은 T 타입을 반환하는 Supplier를 받는다.
그래서 실제로 Optional을 사용할 땐 이 차이점을 잘 인지하고 상황에 맞게 적합한 메서드를 사용하는 것이 좋다. 이 차이점을 잘 알지 못하고 orElse를 사용했던 상황을 NewFit 프로젝트 
예시를 보며 살펴보려고 한다.
