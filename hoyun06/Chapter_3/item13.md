### Item13 : clone 재정의는 주의해서 진행하라

###### Cloneable의 목적 및 문제점
- 의도: 복제해도 되는 클래스임을 명시하는 `믹스인 인터페이스`(mixin interface)
  - 믹스인: 대상 타입의 주된 기능에 선택적 기능을 '혼합(mixed-in)'한다고 해서 믹스인
  - Cloneable을 구현하는 클래스는 해당 클래스 주요 기능은 당연하고 '복제 기능' 이라는 선택적 기능도 제공
- 문제점
  - clone 메서드는 Object에 존재. 그마저도 protected 
  - 따라서, Cloneable을 구현한다고 선언하는 것만으로는 외부에서 clone 메서드 호출 불가


###### Cloneable이 하는 일
- 어떤 클래스가 Cloneable을 구현하고 clone 메서드를 호출하면 대상 클래스의 모든 필드를 복사한
복사본을 반환한다.
- 만약 해당 클래스가 Cloneable을 구현하지 않았다면 CloneNotSupportedException 던진다.

###### clone 메서드 일반 규약
- `x.clone() != x` 식은 참이어야 한다.(내용은 같아도 주소는 다르다)
- `x.clone().getClass() == x.clone().getClass()` 식도 참이어야 한다.(클래스 타입은 일치해야 한다.)
- `x.clone().equals()`는 참이어야 하지만, 필수는 아니다.

###### 잘 구현된 clone 메서드 예시
clone 메서드를 사용하기 위해선... 
1. 대상 클래스에 implements Cloneable 추가
2. clone 메서드 오버라이딩
3. super.clone() 호출

만약 복제하려고 하는 해당 클래스의 모든 필드가 `기본 타입` 혹은 `불변 클래스에 대한 참조`라면 `super.clone()`호출 만으로도 완벽한 복사본 생성이 가능하다.  

잘 된 예시
```java
@Override
public PhoneNumber clone() {
  try {
    return (PhoneNumber) super.clone();
  } catch (CloneNotSupportedException e) {
    throw new AssertionError(); // 일어날 수 없는 일이다.
}
```
자바는 기본적으로 `공변 반환 타입`을 지원하므로 이와 같이 PhoneNumber 클래스로 캐스팅하여 반환해도 된다.
- 공변 반환 타입: 메서드를 오버라이딩할 때 서브 클래스의 타입으로 캐스팅하여 반환하는 것. 이러면 클라이언트 코드에서 
직접 캐스팅을 하지 않고도 바로 사용할 수 있다.

###### clone 재정의 시 발생할 수 있는 문제
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
위와 같이 가변 객체의 참조를 가지는 필드가 있는 경우  
super.clone()을 호출하면 문제가 생긴다.

> size 변수는 올바르게 복사가 되지만   
> 복사된 Stack 객체와 원본 Stack 객체는 elements 배열을 공유한다!

super.clone()을 호출하면 `얕은 복사`가 진행되어 원본 stack의 elements 배열에 담긴 참조값들을 그대로 복사해서
복사본의 elements에서 똑같이 참조한다.  
이 경우 두 객체 중 어느 하나를 수정하면 양쪽 객체 모두에게 반영된다.

그럼 위 Stack 클래스의 올바른 clone 메서드는 어떻게 해야 할까?
```java
public class Stack {
    @Override
    public Stack clone() {
        try {
            Stack clone = (Stack) super.clone();
            clone.elements = elements.clone(); // Array가 제공하는 clone 사용
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    } 
}
```
먼저 super.clone으로 필드 복사를 마친 뒤에 배열의 clone을 재귀적으로 호출하여 배열의 `깊은 복사`를 하면 된다.

다만, 이 방식은 elements 배열이 final로 선언되어 있는 경우 사용이 불가능하다.
그리고 아래와 같은 경우 배열의 clone만으로는 해결되지 않는 문제가 있다.
```java
public class HashTable implements Cloneable {
    private Entry[] buckets;
  
    private static class Entry {
		final Object key;
		Object value;
		Entry next;
	}
  
    public Entry(Object key, Object value, Entry next) {
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```
위 HashTable 클래스에서 이전과 같은 clone 메서드를 재정의한다고 하면 물론 복제본은 자신만의 buckets를 가지긴 한다.  
하지만 buckets 내부에 들어있는 연결 리스트에 대해서는 여전히 동시 참조를 하므로 다른 방법이 필요하다.
따라서 아래와 같이 구현하자
```java
public class HashTable implements Cloneable{
    private Entry[] buckets;
    
    private static class Entry {
    final Object key;
    Object value;
    Entry next;
    
    private static class Entry {
        Object key, value;
        Entry next;
    
        Entry deepCopy() {
          return new Entry(key, value, 
                  next == null ? null : next.deepCopy());
        }
    }
    
    @Override 
    public HashTable clone() {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];
		
        for (int i = 0; i < buckets.length ; i++) {
            if (buckets[i] != null) {
              result.buckets[i] = buckets[i].deepCopy();
            }
        }
        return result;
    }
}
```
Entry 클래스에 deepCopy라는 메서드를 정의하여 자신만의 연결 리스트를 가리키도록 설정해주자.


###### 요약
Cloneable을 구현할 때는..
1. clone 메서드를 재정의하자
2. 리턴 타입을 자신의 클래스로 선언하고 public으로 선언하자 
3. super.clone을 먼저 호출하고 추가 작업이 필요한 필드는 작업을 해준다(ex. 깊은 복사)
   - 추가 작업이란 대체로 가변 객체들에 대한 깊은 복사를 뜻한다.
   - 모든 필드가 기본 타입 혹은 불변 객체라면 불필요하다.
   - 하지만 기본 타입이더라도 id값이나 serialNumber 같은 경우는 clone 시 값 수정이 필요할 수도 있다.

###### Cloneable... 너무 귀찮은데 다른 방법은 없나요?
있다.  
`복사 생성자 / 복사 팩터리` 라는 것이 있다. 복사 팩터리는 복사 생성자의 변형이므로 복사 생성자를 보면
```java
public class Car {
	public Car() {}
	
	public Car(Car car) {
		// 복사
    }
}
```
위와 같이 자신을 매개변수로 받는 생성자를 뜻한다.  
이 방식은 Cloneable을 구현하는 것보다 훨씬 나은 점들이 있다.

1. 생성자 방식을 사용한다.
2. final을 정상적으로 사용할 수 있다.
3. 형변환이 불필요하다
4. 해당 타입이 구현한 인터페이스를 매개변수로 받을 수도 있다.

## 핵심 정리
    객체 복사 시에는 웬만해선 복사 생성자 / 복사 팩터리를 사용하자.  
    하지만 배열의 경우 clone을 사용해도 된다.