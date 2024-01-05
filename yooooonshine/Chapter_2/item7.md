# 다 쓴 객체 참조를 해제하라
---
c,c++은 메모리를 직접 관리해야 한다.  즉 동적할당(malloc)을 하였다면 반드시 삭제(delete)를 해줘야 한다.
이에 비해 자바는 delete가 따로 없고 가비지 컬렉터가 자동으로 회수해 간다.

다만 자바의 가비지 컬렉터가 있다고 해서 메모리 관리를 안 해도 된다는 것은 아니다.
```java
private Object[] elements;
private int size = 0;

public Object pop() {
	if (size == 0) {
		throw new EmptyStackException();
	}
	return elements[--size];
}
```
위의 경우 pop()을 하더라도 size가 줄어들 뿐 여전히 element는 줄어들기 전 size 인덱스에 대한 참조를 가지고 있다.
즉 다시 쓰지 않는다 하더라도 여전히 참조를 가지고 있기에 메모리 누수가 발생한다.

이를 해결하기 위한 방법은
```java
private Object[] elements;
private int size = 0;

public Object pop() {
	if (size == 0) {
		throw new EmptyStackException();
	}
	elements[size] = null;
	return elements[--size];
}
```
위와 같이 null을 통해 참조를 없애주는 것이다.
가비지 컬렉터는 아무도 참조하지 않는 객체에 대해 회수해가기 때문이다.

즉 가비지컬렉터는 객체 참조 하나를 살려두면 이 객체 뿐 아니라, 이 객체가 참조하는 모든 객체를 수거해가지 못한다.

다만 null 처리하는 일은 프로그램을 지저분하게 하므로 예외적으로 처리해야 한다.
다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다.
즉 변수를 선언한 블록 밖으로 나와, 변수 유효범위를 벗어나, 참조를 해제하는 것이다.

## null 처리는 언제 해야 하는 걸까?
이는 stack 클래스와 같이 자기 메모리를 직접 관리하는 경우 null 처리를 통해 쓰지 않음을 가비지 컬렉터에 알려야 한다.
즉 자기 메모리를 클래스가 직접 관리한다면 가비지 컬렉터 입장에서는 어떤 객체가 사용되며 어떤 객체가 사용 안 될지 알 수 없다.



그렇다면 메모리 누수를 일으키는 것들은 무엇이 있을까?
## 1. 스택
위에서도 봤듯이 스택에 들어간 정보는 가비지 컬렉터가 어떤 것이 사용안될 객체인지 인지하지 못하므로, 메모리 누수를 일으키기 쉽다.
따라서 null을 이용하자!
## 2. 캐시
객체참조를 캐시에 넣어두고, 까먹으면 그 객체를 다 사용한 후에도 캐시에 계속 존재하게 된다.

이를 해결하기 위한 방법은
1. WeakHashMap을 활용하여 자동으로 처리
2. ScheduledThread PoolExcutor같은 백그라운드 스레드를 활용하거나 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행

여기서 weakHashMap이란?
```java
public class WeakHashMapTest {

	public static void main(String[] args) {
	
	WeakHashMap<Integer, String> map = new WeakHashMap<>();
	
	Integer key1 = 1000;
	
	Integer key2 = 2000;
	
	map.put(key1, "test a");
	
	map.put(key2, "test b");
	
	key1 = null;
	
	System.gc(); //강제 Garbage Collection
	
	map.entrySet().stream().forEach(el -> System.out.println(el));

	}

}
```
여기서 map안에 있는 key가 가리키는 객체가 null이 되면 자동으로 key도 지워주는 map이다.
## 3. 리스너, 콜백
클라이언트가 리스너나 콜백을 등록만 하고 해지하지 않는다면 콜백은 계속 쌓이게 되어 메모리 누수를 일으킨다. 
즉 콜백이나, 리스너 함수는 이벤트등에 대응하기 위해서 사전에 등록되어 있는 데, 등록만 되고 해지되지 않는다면 메모리 누수가 생긴다는 것이다.
이를 해결하기 위한 방법으로 콜백을 약한 참조(weak reference)로 저장하여 가비지 컬렉터가 수거해가도록 한다.

참고로 여기서
1. 리스너란
javaScript에서 존재하는 이벤트 리스너와 같이 특정 이벤트가 발생했을 때 작동하는 것이다.
2. 콜백이란
콜백이란 콜백함수를 등록해놓으면 이벤트가 발생했을 때 콜백함수가 호출된다.