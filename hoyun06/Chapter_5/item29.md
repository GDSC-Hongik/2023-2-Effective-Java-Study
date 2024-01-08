### Item29 : 이왕이면 제네릭 타입으로 만들라

item 7번에서 Stack 예제 코드를 작성한 적이 있다.
```java
public class Stack {
    private Object[] elements; // 스택에 들어갈 원소를 담을 배열
    private int size = 0;     // 스택의 크기
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
        Object top = elements[--size];
        elements[size] = null; 
        return top;
    }
}
```
이번 item에선 이 Stack 클래스를 제네릭 클래스로 수정하려고 한다.  
제네릭 클래스로 만들기 위해선 클래스 선언부에 타입 매개변수를 추가하고 클래스 내부 타입을 적절히 변경하면 된다.
```java
public class Stack<E> {
    private E[] elements; // 스택에 들어갈 원소를 담을 배열
    private int size = 0;     // 스택의 크기
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
        E top = elements[--size];
        elements[size] = null; 
        return top;
    }
}
```
이렇게 변경을 완료하고 컴파일하면 오류가 발생한다.  
생성자 안에 `new E[DEFAULT_INITIAL_CAPACITY]` 부분에서 문제가 발생한다. 이전 item에서 다뤘던 것처럼 E는 실체화 불가 타입이므로
E 타입 배열을 선언하는 것은 불가능하다. 그렇다면 E 타입 배열을 생성하는 방법이 없을까?  
책에선 배열 생성 제약을 우회할 수 있는 2가지 방법을 알려준다.

###### 첫 번째 방법: 타입 캐스팅 사용하기
생성자 내부 코드를 다음과 같이 수정하자.
```java
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```
이렇게 하면 정상적으로 배열 생성이 가능해진다. 이렇게 하면 컴파일은 정상적으로 되지만 컴파일러는 이에 대해 비검사 형변환 경고를 띄운다.
생성된 Object 배열이 타입 안전한지 확인할 수 없다는 것이다. 그래서 이걸 우리가 직접 해줘야 한다. 다음 사항들을 고려해 볼 때 위 코드는 타입 안전하다고 생각할 수 있다.
1. elements 배열은 private이므로 외부로 공유될 일이 없으며 다른 메서드로 넘겨지는 일도 없다.
2. 배열에 원소 추가는 push 메서드에 의해서만 일어나는데 그때 들어가는 원소 타입이 E 타입인 것은 분명하다.

따라서, 위 코드가 타입 안전함을 우리 스스로가 보장할 수 있으므로 이전에 소개했던 @SuppressWarnings("unchecked")을 이용하여 비검사 경고를 숨길 수 있다.

###### 두 번째 방법: Object 배열 사용하기
첫 번째 방법에선 elements 배열이 E 타입이었다. 하지만 이번엔 Object 배열을 사용하려고 한다. elements 배열의 타입을 Object로 변경하면 컴파일러는 오류를 발생시킨다.  
```java
E top = elements[--size]; // Object 타입의 원소를 E로 캐스팅할 수 없다!
```
다행히도 이 문제는 캐스팅을 통해 해결할 수 있다. E 타입으로 캐스팅을 해주면 컴파일러는 이번엔 경고를 띄워준다.  
이번에도 역시나 비검사 형변환 경고를 띄우는데 이 경우에도 타입 안전성은 자명하므로 어노테이션을 이용하여 경고를 숨길 수 있다.

###### 이전 item과의 모순?
이번 item에선 제네릭 클래스에서의 배열 사용에 관한 내용을 다뤘다. 사실 이전 item에선 배열보단 제네릭 컬렉션 사용을 권장했지만 그 방법이 항상 옳은 것은 아니다.  
리스트라는 자료구조는 자바에 의해 기본 제공되는 타입이 아니다. ArrayList와 같은 리스트 컬렉션 클래스도 내부적으론 배열로 구현되어 있다.  
다만 배열과는 다르게 제네릭 컬렉션은 타입 매개변수를 정함에 있어 제약이 없다.