# Item 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라

## 타입 안전성의 문제
```java
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList;
    String s = stringLists[0].get(0); // ClassCastException
}
```
- 배열은 공변이므로 List<String>[]은 Object[]의 하위 타입
  - objects[0] = intList;에서 컴파일 에러가 발생하지 않음
  - 런타임에 마지막 줄에서 ClassCastException이 발생

## @SafeVarargs
- 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치
- 안전하다는 확신이 있을 때만 사용해야 함
  - 가변인수 메서드를 호출할 때 제네릭 배열이 생성된다는 것을 기억하자.

### varargs 매개변수 배열에 아무것도 저장하지 않으면 안전한가?
```java
static <T> T[] toArray(T... args) {
    return args;
}
```
- 반환 타입은 메서드에 인수를 넘기는 컴파일 타임에 결정됨
- 이때는 컴파일러에게 충분한 정보가 주어지지 않으므로 타입을 잘못 판단 가능.
=> 그대로 반환하면 힙 오염을 메서드를 호출한 쪽으로 전이 가능

### 제네릭 varargs 매개변수 배열에 다른 메서드가 접근해도 안전한 경우
1. @SafeVarargs로 제대로 처리된 varargs 메서드에 넘기는 경우
2. 이 배열의 내용의 일부 함수를 호출만 하는 일반 메서드에 넘기는 경우