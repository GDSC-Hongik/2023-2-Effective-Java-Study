# Item 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

매개변수화 타입은 불공변
- List<String>은 List<Object>의 하위 타입이 아님
- List<String>은 List<Object>가 하는 일을 제대로 수행 불가  
=> 하위 타입이 될 수 없음.  

## 불공변보다 유연한 타입이 필요할 때도 있다
```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```
위의 Stack 클래스에 pushAll 메서드를 추가해보자.
```java
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}
```
- 컴파일은 됨
- Stack<Number>로 선언하고 pushAll 메서드에 List<Integer>를 넘긴다면?
  - 매개변수화 타입은 불공변이므로 컴파일 에러 발생
  - 어떻게 해결?
    - 입력 매개변수를 E의 Iterable이 아니라 E의 하위 타입의 Iterable로 바꾸면 됨.
    - Iterable<? extends E>가 바로 E의 하위 타입의 Iterable

```java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```

이번에는 popAll 메서드를 추가해보자.
```java
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```
- pushAll과 달리 E의 상위 타입을 dst에 담을 수 있어야 함.
- 따라서, Collection<? super E>를 사용하는 것이 맞음.
```java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```
유연성을 극대화한 두 메서드의 예시를 살펴봤다.

## PECS
- producer-extends, consumer-super라는 뜻이다.
- 생산자라면 extends, 소비자라면 super를 사용하라는 뜻이다.
- 여기서 생산자는 객체를 가져올 곳, 소비자는 객체를 넣을 곳이라고 이해하면 쉽게 이해할 수 있을 것 같다.

