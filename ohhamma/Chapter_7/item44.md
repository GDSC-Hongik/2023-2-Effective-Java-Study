# 표준 함수형 인터페이스를 사용하라

```
💡 필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라
```

## 기본 인터페이스 (총 6개)

### UnaryOperator<T>

```java
T apply(T t)
```

- 반환값과 인수의 타입 동일

### BinaryOperator<T>

```java
T apply(T t1, T t2)
```

- 반환값과 인수의 타입 동일

### Predicate<T>

```java
boolean test(T t)
```

- 인수 하나를 받아 boolean 반환

### Function<T, R>

```java
R apply(T t)
```

- 인수와 반환 타입이 다름

### Supplier<T>

```java
T get()
```

- 인수를 받지 않고 값을 반환

### Consumer<T>

```java
void accept(T t)
```

- 인수를 하나 받고 반환값 없음

## 변형 인터페이스

```
💡 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자
```
### 기본 타입을 반환하는 변형 (각 3개씩)

- 기본 인터페이스에 **int**, **long**, **double**용으로 각 3개씩 변형이 생김
    - ex) `IntPredicate` : int를 받는 Predicate
- **Function**의 변형만 (반환타입) 매개변수화
    - ex) `LongFunction<int[]>` : long 인수를 받아 int[] 반환

### Function 인터페이스의 추가 변형 (총 9개)

- 입력과 결과 타입이 모두 기본 타입 → `SrcToResult`
    - ex) `LongToIntFunction`
- 입력이 객체 참조이고 결과가 기본 타입 → `ToResult`
    - ex) `ToLongFunction<int[]>`

### 인수를 2개씩 받는 변형 (총 9개)

- `BiPredicate<T, U>`
- `BiFunction<T, U, R>`
- `BiConsumer<T, U>`

→ 기본 타입을 반환하는 변형 3개씩

### BooleanSupplier 인터페이스

- `boolean`을 반환하도록 한 Supplier의 변형

## 그렇다면 코드를 직접 작성해야 할 때는 언제인가?

- 표준 인터페이스 중 필요한 용도에 맞는 게 없는 경우
- 구조적으로 똑같은 표준 함수형 인터페이스가 있더라도 직접 작성해야만 하는 경우
    - `Comparator<T>`

### Comparator 특성

```
💡 이 중 하나 이상을 만족하면 전용 함수형 인터페이스 구현을 고민해보자
```

- **자주** 쓰이며, 이름 자체가 용도를 명확히 설명해줌
- 반드시 따라야 하는 **규약**이 있음
- 유용한 **디폴트 메서드**를 제공할 수 있음

## @FunctionalInterface

```
💡 직접 만든 함수형 인터페이스에는 항상 @FunctionalInterface 애너테이션을 사용하라
```

- 해당 인터페이스가 **람다용**으로 설계된 것임을 알려줌
- 해당 인터페이스가 **추상 메서드를 오직 하나만** 가지고 있어야 컴파일되게 해줌
- 유지보수 과정에서 실수로 메서드를 **추가하지 못하게** 막아줌

## 함수형 인터페이스를 API에서 사용할 때의 주의점

```
💡 다중정의는 주의해서 사용하라
```

- 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안 된다
    - ex) ExecutorService의 sumbit 메서드 : Callable<T>, Runnable 다중정의
    - 올바른 메서드를 알려주기 위해 형변환 필요