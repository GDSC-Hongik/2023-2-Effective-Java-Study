### Item7 : 다 쓴 객체 참조를 해제하라

###### 자바에 대한 오해
**자바는 C 혹은 C++와는 다르다.**  
자바에서는 사용하지 않는 객체를 개발자가 직접 처리하지 않아도 된다. GC가 알아서 해주니 말이다.  
하지만 그렇다고 해서 개발자가 아예 메모리 관리를 해주지 않아도 된다는 것은 큰 오해다.

###### 메모리 누수 예시
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
        return elements[--size]; // pop하는 객체에 대해서 추가적인 처리를 하지 않는 예시.
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
위 코드를 보면 Stack이라는 클래스를 정의하여 간단한 형태의 스택 자료구조를 구현한 모습이다.  
여기서 pop 메서드는 스택이 비어있지 않다면 가장 위에 있는 원소를 반환한다.
>위 코드는 정상적으로 동작하지만 사실은 메모리 누수의 위험성을 내포하고 있다.
 
메모리 누수는 바로 `pop 메서드` 내부에서 일어난다.  

pop 메서드를 살펴보면..
```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    return elements[--size];
}
```
- 이와 같이 스택의 맨 위에 있는 원소를 반환하지만 해당 객체에 대한 추가적인 처리를 하지 않는다.  
- 여기서 추가적인 처리란 더이상 사용되지 않는 객체에 대한 처리를 뜻한다.  

이 경우엔 스택에서 꺼내진 객체들은 GC에 의해 수거되지 않는다.  
'pop 메서드를 통해 반환된 객체를 사용하니까 당연한 거 아닌가?'라고 생각할 수도 있다.   
하지만 위 경우에는 반환된 객체가 더이상 사용되지 않더라도 GC에 의해서 수거되지 않는다.
>객체가 pop되더라도 elements 배열에 해당 객체의 참조값이 남아있기 때문이다.

###### 어차피 배열 안에 있는 객체는 몇 개 없지 않나요..
그래봐야 배열 안에 있는 객체들에 대해서만 메모리를 차지하니 별 일 아니라고 생각할 수 있다.  
하지만 그 객체가 내부적으로 참조하는 객체들과 또 그 객체들이 내부적으로 참조하는 객체들도 모두 GC의 수거 대상에서 제외되므로 메모리 누수가 꽤나 커지게 된다.  
즉, 단 하나의 객체만으로도 연쇄적으로 메모리 누수를 일으킬 수 있는 셈이다.  
그러므로 위 pop 메서드를 다음과 같이 구현하자
```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object top = elements[--size];
    elements[size] = null; // pop되는 객체의 참조를 null로 설정한다.
    return top;
}
```
이처럼 사용되지 않는 객체의 참조를 null로 바꾸면..
1. 메모리 낭비를 막을 수 있다.
2. 혹시나 스택에서 pop된 객체를 다시 접근하려고 하면 바로 NPE가 터지면서 오류를 발생시킨다.

###### 그럼 개발자가 항상 미사용 객체의 참조를 null로 바꿔줘야 하나요?
그렇지 않다. 개발할 때마다 미사용 객체를 null로 바꾸는 것은 별로 효율적이지도 않고 코드만 지저분하게 만드는 경우가 많다.   
미사용 객체의 참조를 null로 바꾸는 대신 해당 미사용 객체를 스코프 밖으로 밀어내면 자연스레 해당 객체는 GC의 수거 대상이 된다.  
코드를 메서드 단위로 잘 분리해서 클린 코드를 작성한다면 이는 자연스레 달성할 수 있다.

###### 그럼 위 Stack 클래스에서는 왜 명시적으로 null로 바꾸나요?
>위 Stack 클래스는 자체적으로 메모리 관리를 하기 때문이다.

- Stack 클래스는 필드에 있는 elements 배열을 통해 자신만의 메모리 풀을 유지한다.
- 그렇기 때문에 pop 메서드에 의해 특정 객체가 스택에서 제거돼도 해당 객체는 여전히
elements 배열에서 참조된다.
- 그렇기 때문에 GC는 이 객체가 더이상 사용되지 않는 미사용 객체라는 것을 알 수가 없다.

**정리하자면, 자신만의 메모리 공간을 유지하는 경우 명시적으로 null 참조를 해주는 것이 바람직하다.**

###### 이외에도 메모리 누수를 일으킬 수 있는 경우
1. 객체 참조를 캐시(cache)에 저장하고 놔두는 경우  
    - Map 자료형 안에 객체를 저장해놓고 이후에 그 객체가 더이상 사용되지 않아서 그 존재에 대해
   잊게 되는 경우 메모리 누수가 발생한다.
    - 이 경우엔 해결책으로 WeakHashMap을 사용할 수 있다. WeakHashMap은 map에 저장되어 있는 
   엔트리들의 key 값이 제거되면 해당 value 값도 같이 제거해준다.
   - 참고로 일반 HashMap은 위 기능이 없다. 특정 엔트리의 key 값이 제거돼도 value 값은 여전히 유지된다.
    ```java
    Map<Key, String> map = new HashMap<>();
    
    Key key = new Key();
    map.put(key, "1");
    
    key = null;
    System.gc();
    ```
   HashMap은 value "1" 의 key인 `key` 참조 객체에 null이 들어가도 "1" 인스턴스는 여전히 유지된다.  
    
    반면...
     ```java
    Map<Key, String> map = new WeakHashMap<>();
    
    Key key = new Key();
    map.put(key, "1");
    
    key = null;
    System.gc();
    ```
   WeakHashMap은 key 값에 null이 들어가면 GC에 의해서 value 값인 "1"도 수거 대상이 된다.
2. 리스너(listener) 혹은 콜백(callback)
    - api 제공 시에 클라이언트가 콜백을 등록만 할 수 있고 해지할 수 있는 수단을 제공하지 않으면
   콜백이 쌓이기만 하고 수거가 되지 않아서 메모리 누수가 발생한다.
    - 위에서 언급한 WeakHashMap의 key 값으로 콜백을 등록하면 메모리 누수를 막을 수 있다.