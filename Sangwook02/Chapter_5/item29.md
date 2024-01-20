# Item 29. 이왕이면 제네릭 타입으로 만들라

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    // 이하 생략
}
```
제네릭 타입을 새로 만들어 사용할 수 있도록 연습해보자.  
위는 제네릭 타입을 사용하지 않은 Stack 클래스이다.

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    // 이하 생략
}
```
클래스 선언에 <E>를 추가하고, Object를 E로 바꿨다.  
한 가지 문제가 있는데, E는 실체화 불가 타입이기 때문에 배열을 만들 수 없다.  
=> `elements = new E[DEFAULT_INITIAL_CAPACITY];`에서 오류가 발생한다.

## 해결 방법
### Object 배열을 E 배열로 형변환
첫 번째 방법으로 Object 배열을 E 배열로 형변환하는 방법이 있다.  
컴파일러는 오류 대신 경고를 내보내지만, 타입 안전하지 않다.  

안전함을 증명했다면, `@SuppressWarnings("unchecked")` 어노테이션을 달아 경고를 숨길 수 있다.

### 필드를 E[ ] -> Object[ ] 변경
이렇게 하면 위의 형변환과는 다른 문제가 생긴다.  
pop()의 `E result = elements[--size];`에서 오류가 발생한다.  

`@SuppressWarning("unchecked")
E result = (E) elements[--size];`
로 해결할 수 있다.


## 제네릭 타입의 타입 매개변수
대부분의 타입을 사용할 수 있으나, 기본 타입은 사용 불가.