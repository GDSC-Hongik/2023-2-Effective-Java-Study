### Item79 : 과도한 동기화는 피하라

이번 item에서는 과도한 동기화에 의한 문제점을 다룬다.

###### 과도한 동기화가 야기하는 문제
- 성능 저하
- 교착 상태 야기
- 예상치 못한 동작 야기

###### 동기화 메서드 혹은 동기화 블럭 내에서는 절대로 제어권을 클라이언트에게 넘기면 안된다
이게 무슨 뜻이냐면 동기화 블럭 내에선 재정의 가능 메서드나 클라이언트로부터 전달받은 외부 객체와 같은
외부 세계의 객체 혹은 메서드를 사용해선 안된다는 뜻이다.  
왜냐하면 해당 재정의 가능 메서드가 어떤 방식으로 재정의되어 사용될지 모르고 마찬가지로 클라이언트로부터 넘겨받은 객체가 어떤 행동을 할 지 예측할 수가 없기 때문이다.

###### 동기화 블럭 내에서 외부 메서드 호출하는 예시
```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers
            = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }
	
    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for (SetObserver<E> observer : observers) {
                observer.added(this, element); // 문제 발생
            }
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);
        return result;
    }
}
```
위 코드는 관찰자 패턴을 적용한 클래스로서 위 클래스를 사용하는 클라이언트는 Set에 원소가 추가될 때마다 
Observer가 보내는 알림을 받을 수 있다. 
```java
@FunctionalInterface
public interface SetObserver<E> {
    void added(ObservableSet<E> s, E element);
}
```
```java
public class Main {
    public static void main(String[] args) {
        ObservableSet<Integer> s =
                new ObservableSet<>(new HashSet<>());
    
        s.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> s, Integer element) {
                System.out.println(element);
                if (element == 23)
                    s.removeObserver(this); 
            }
        });
    
        for (int i = 0; i < 100; i++) {
            s.add(i);
        }
    }
}
```
위 main 메서드에서는 새로운 관찰자를 추가하는데 이 관찰자는 23 이라는 숫자가 Set에 추가되면 관찰자 리스트에서 자기 자신을
제거하는 로직을 실행한다.  
그래서 for 문이 처음에 돌기 시작하면 0 ~ 22까지는 정상적으로 콘솔에 출력된다. 그리고 i = 23 인 순간 `s.add(i)`가 호출되면
```java
@Override
public boolean add(E element) {
    boolean added = super.add(element);
    if (added)
        notifyElementAdded(element);
    return added;
}
```
위 메서드가 실행되며 Set에 23이라는 원소를 넣고 `notifyElementAdded(element)` 를 또 호출한다.
```java
private void notifyElementAdded(E element) {
    synchronized (observers) {
        for (SetObserver<E> observer : observers) {
            observer.added(this, element); // 문제 발생
        }
    }
}
```
이 메서드에선 우선 동기화 블럭을 통해 `관찰자 리스트`에 대한 `락`을 획득한다. 그리곤 관찰자 리스트를 순회하며 각 관찰자의 `added` 메서드를 실행한다.
그렇게 관찰자 리스트를 순회하다가 
```java
new SetObserver<Integer>() {
    @Override
    public void added(ObservableSet<Integer> s, Integer element) {
        System.out.println(element);
        if (element == 23)
            s.removeObserver(this); 
    }
}
```
main 메서드에서 직접 추가한 위 관찰자의 `added` 메서드가 호출된다. 이때 콘솔에 '23'을 출력하고 `s.removeObserver(this)` 를 통해 관찰자 제거 메서드를 호출하게 된다.  
```java
public boolean removeObserver(SetObserver<E> observer) {
    synchronized(observers) {
        return observers.remove(observer);
    }
}
```
그에 따라 위 메서드가 호출되는데 이는 관찰자 리스트에서 매개변수로 받은 관찰자를 찾아 제거하는 역할을 수행한다.

현재 상황은 `notifyElementAdded` 메서드에서 관찰자를 순회하며 각 관찰자의 `added` 메서드를 실행 중인 상황이다.
그런데 그와 동시에 `removeObserver` 메서드에서 해당 관찰자 리스트의 원소를 제거하는 동작을 수행하게 되는 것이다.
그래서 결국 'ConcurrentModificationException'이 발생하게 된다.

이 문제의 원인은 동기화 블럭을 가진 `notifyElementAdded` 메서드 내부에서 외부 메서드인 `added` 메서드를 호출한 것이다.  

그럼 다음과 같은 생각을 할 수도 있다.
> `notifyElementAdded` 메서드에서 이미 관찰자 리스트에 대한 락을 획득했는데 어떻게 `removeObserver` 메서드에서 관찰자 리스트 내부의 원소를 제거한 걸까?

이는 자바 언어의 `락`과 관련이 있다.  
자바 언어에서의 `락`은 재진입을 허용한다. 즉, 특정 자원에 대한 `락`을 획득한 스레드는 해당 자원에 아무런 제재없이 재차 접근이 가능하다.

위 코드의 경우 `notifyElementAdded` 메서드에서 `락`을 획득한 스레드가 `removeObserver` 메서드도 실행한다. 그렇기 때문에 해당 스레드는 
관찰자 리스트에 접근이 가능했던 것이다.

###### 위 문제 해결 방법
다행히도 위 문제를 해결할 수 있는 방법이 있다. 동기화 블럭 내부에서 호출하던 외부 메서드를 블럭 밖으로 꺼내면 된다.
```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot)
        observer.added(this, element);
}
```
보면 외부 메서드인 `added` 메서드의 호출을 동기화 블럭 외부로 꺼냈다. 그와 동시에 관찰자 리스트를 복사하여 사용함으로써 문제를 해결할 수 있다.

이렇게 외부 메서드를 동기화 블럭 밖으로 빼내는 것을 `열린 호출`이라고 한다. 그런데 이 `열린 호출`방식을 이용하는 것 말고도 위 문제를 해결할 수 있는 또 다른 해결책이 있다.

###### 자바 동시성 컬렉션 라이브러리의 CopyOnWriteArrayList 사용
이름에서 알 수 있듯이 ArrayList 를 구현한 것인데 내부 배열 변경 작업을 할 시 항상
완벽한 복사본을 만들어 수행하도록 설계되어서 `락` 획득 없이도 순회할 수 있다는 장점이 있다.
```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}
```
위 코드는 CopyOnWriteArrayList를 이용하여 재구성한 것인데 보면 명시적으로 `synchronized`를 사용하지 않았음을 알 수 있다.

###### 가변 클래스 작성과 동기화 성능
가변 클래스 작성 시 동기화 관련 처리를 하기 위해선 2가지 선택지가 있다
- 해당 가변 클래스 내부에선 일체 동기화 처리를 하지 말고 이를 사용하는 클라이언트에서 알아서 동기화하도록 한다
- 동기화를 가변 클래스 내부에서 처리하고 thread-safe 클래스로 만든다

단, 두 번째 방법은 클라이언트가 객체 전체에 `락`을 거는 것보다 성능면에서 이점이 있을 때만 채택하는 것이 좋다.
그렇지 않다면 첫 번째 방법을 선택하고 문서에 해당 클래스는 스레드 안전하지 않음을 명시하자.

만약, 두 번째 방법을 이용하기로 했다면 '락 분할', '락 스트라이핑', '비차단 동시성' 제어와 같은 다양한 기법을 사용하여 동시성을 높일 수 있다.