# Item 7. 다 쓴 객체 참조를 해제하라

C++에서처럼 매번 소멸자를 부를 필요없이 GC가 참조되지 않은 인스턴스를 알아서 해제해주니 메모리 관리에 대한 편의성이 좋아졌다.  
하지만, 여전히 신경쓰지 않으면 부분이 있다.  
자기 메모리를 직접 관리하는 클래스가 바로 그 예시이다.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
	
    ,,,(생략)

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size]; 
    }
}
```
이 스택의 pop() 메서드를 보면, 스택이 커졌다가 줄어들 때
스택에서 제외된 객체들을 GC가 회수하지 않는다.  
의도치 않게 다 쓴 참조를 여전히 가지고 있으므로 그 객체 뿐만 아니라 그 객체가 참조하는 모든 객체를 회수하지 못한다.  
다 쓴 참조는 null 처리해주면 이 문제를 해결할 수 있다.  
위의 pop메서드를 아래와 같이 바꾸면 스택에서 꺼내지는 순간 참조가 해제된다.
```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] = null;
    return result; 
}
```
물론 무조건 null 처리 하는 것이 좋은 방법은 아니다.  
가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이므로 null 처리는
자기 메모리를 직접 관리하는 클래스에 한해서만 해주면 된다.
***
이외에도 캐시, 리스너 등이 메모리 누수의 주범이다.  
메모리 누수는 겉으로 잘 드러나지 않아 발견하기 쉽지 않다.  
코드를 짤 때부터 메모리 누수에 주의해서 짜는 것이 좋을 것 같다.