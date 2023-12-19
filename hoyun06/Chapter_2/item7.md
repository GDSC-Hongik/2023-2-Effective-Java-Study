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
- 여기서 추가적인 처리란 더 이상 사용되지 않는 객체에 대한 처리를 뜻한다.  

이 경우엔 스택에서 꺼내진 객체들은 GC에 의해 수거되지 않는다.  
'pop 메서드를 통해 반환된 객체를 사용하니까 당연한 거 아닌가?'라고 생각할 수도 있다.   
하지만 위 경우에는 반환된 객체가 더 이상 사용되지 않더라도 GC에 의해서 수거되지 않는다.
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
- 그렇기 때문에 GC는 이 객체가 더 이상 사용되지 않는 미사용 객체라는 것을 알 수가 없다.

**정리하자면, 자신만의 메모리 공간을 유지하는 경우 명시적으로 null 참조를 해주는 것이 바람직하다.**

###### 이외에도 메모리 누수를 일으킬 수 있는 경우
1. 객체 참조를 캐시(cache)에 저장하고 놔두는 경우  
    - Map 자료형 안에 객체를 저장해놓고 이후에 그 객체가 더 이상 사용되지 않아서 그 존재에 대해
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
   HashMap은 value "1" 의 key인 `key` 참조 객체에 null이 들어가도 "1" 엔트리는 여전히 유지된다.  
   실제로 System.gc()를 호출하고 일정 시간이 지난 뒤에 map의 size를 출력하면 2가 나온다.
   ```java
   System.out.println(map.size()); // 2가 나온다
   ```
    
    반면...
     ```java
    Map<Key, String> map = new WeakHashMap<>();
    
    Key key = new Key();
    map.put(key, "1");
    
    key = null;
    System.gc();
    ```
   WeakHashMap은 key 값에 null이 들어가면 해당 `key` 인스턴스는 GC에 의해서 수거되며 value가 "1"인 엔트리는 map에서 제거된다.
   ```java
   System.out.println(map.size()); // 1이 나온다
   ```
2. 리스너(listener) 혹은 콜백(callback)
    - api 제공 시에 클라이언트가 콜백을 등록만 할 수 있고 해지할 수 있는 수단을 제공하지 않으면
   콜백이 쌓이기만 하고 수거가 되지 않아서 메모리 누수가 발생한다.
    - 위에서 언급한 WeakHashMap의 key 값으로 콜백을 등록하면 메모리 누수를 막을 수 있다.

### 추가 내용
###### Reference의 종류
자바에는 참조의 종류가 여러 개 있다. 
- Strong Reference
- Soft Reference
- Weak Reference
- Phantom Reference

그리고 이 참조들은 GC가 수거 대상을 판단하는 것과 관련이 있다. 
###### Strong Reference
우리가 흔히 사용하는 참조다.
```java
Car car = new Car();
```
강한 참조를 가지는 인스턴스는 기본적으로 GC의 수거 대상이 아니다.   
만약 해당 인스턴스가 수거되길 원한다면 명시적으로 `car = null`과 같이 null 세팅을 해야 한다.

###### Soft Reference
- Strong Reference보다 강도가 약한 참조다.
- java.lang.ref에서 제공하는 SoftReference 클래스를 이용하여 생성할 수 있다.
- 어떤 인스턴스를 가리키는 참조가 오직 Soft Reference만 있으면서 메모리가 부족한 상황이라고 판단될 경우
GC 실행 시 이 인스턴스는 수거 대상에 포함된다.

아래 예시 코드를 보면 이해가 쉽다.
```java
Car car = new Car(); // Strong
SoftReference<Car> softCar = new SoftReference<>(car); // soft

car = null // 이제 해당 인스턴스는 soft 참조만 가지는 상황

System.gc() // 메모리 상황에 따라 수거될 수도 있고 수거되지 않을 수도 있다

car = softCar.get(); // 만약 수거되지 않았다면 get 메서드를 통해 인스턴스 접근이 가능하다 
```
만약 메모리 상황이 좋지 않아 위 인스턴스가 수거되었다면 `get()` 메서드 호출 시 null이 반환된다.

###### Weak Reference
- Soft Reference와 강도가 유사한 참조다.
- java.lang.ref에서 제공하는 WeakReference 클래스를 이용하여 생성할 수 있다.
- 다만, Soft Reference와는 다르게 어떤 인스턴스를 가리키는 참조가 오직 Weak Reference만 있는 경우
해당 인스턴스는 메모리 상황과 상관없이 항상 GC에 의해 수거된다. 

```java
Car car = new Car(); // Strong
WeakReference<Car> weakCar = new WeakReference<>(car); // weak

car = null // 이제 해당 인스턴스는 weak 참조만 가지는 상황

System.gc() // GC 실행 시 위 인스턴스는 무조건 수거된다

car = weakCar.get(); // null 반환 
```

### WeakHashMap의 내부 동작 원리
본격적으로 WeakHashMap의 동작 원리에 대해 공부하기 전에 알고 가면 좋은 사실들이 있다.

###### WeakHashMap 내부 자료구조

우리는 WeakHashMap 내부에 key, value 쌍을 저장한다.
이때 실질적으로 WeakHashMap 내부에 저장되는 Key, value 쌍은 `Entry<K, V>` 라는 인스턴스이다.  
이 Entry 인스턴스를 배열의 형태로 저장한다.
```java
public class WeakHashMap<K,V> extends AbstractMap<K,V> implements Map<K,V> {
    // 생략

    Entry<K, V>[] table;
    
   // Reference queue for cleared WeakEntries
   private final ReferenceQueue<Object> queue = new ReferenceQueue<>();
}
```
```java
private static class Entry<K, V> extends WeakReference<Object> implements Map.Entry<K,V> {
    V value;
    final int hash;
    Entry<K,V> next;
    
    /**
    * Creates new entry.
    */
    Entry(Object key, V value, 
          ReferenceQueue<Object> queue, int hash, Entry<K,V> next) {
       
        // WeakReference 클래스의 생성자 호출
        super(key, queue);
        this.value = value;
        this.hash = hash;
        this.next = next;
    }
    
    
    // 생략
}
```
코드를 보면 Entry 클래스는 WeakReference 클래스를 상속한다. 특히, key값을 WeakReference로 참조한다.

###### ReferenceQueue
이름에서 알 수 있듯이 주로 Reference(soft, weak) 객체들이 담기는 자료구조라고 생각하면 된다.  
하지만 Reference 객체라고 해서 무조건 ReferenceQueue에 담기는 것은 아니다.  

**결과적으로 말하면 WeakReference 객체가 내부적으로 참조하던 객체가 GC에 의해 수거되면
해당 내부 참조를 null로 만들면서 WeakReference 객체 자체는 자신과 연결된 ReferenceQueue에 enqueue 된다.**

```java
public static void main(String[] args) throws InterruptedException {
    NyPizza nyPizza = new NyPizza(5);

    // ReferenceQueue 생성
    ReferenceQueue<Object> rq = new ReferenceQueue<>();
        
    // NyPizza 객체를 가리키는 WeakReference 객체 생성.
    // 이때, 이 WeakReference 객체가 나중에 enqueue될 ReferenceQueue를 지정 가능하다.     
    WeakReference<NyPizza> weakPizza = new WeakReference<>(nyPizza, rq);

    // 내부적으로 참조하는 NyPizza 객체가 잘 반환된다!
    System.out.println("weakPizza.get: " + weakPizza.get());
    
    // poll 메서드는 ReferenceQueue 내부에 Reference 객체가 있다면 무조건 하나 반환한다.
    // 현재는 rq가 비어있다. 그래서 출력 결과는 "poll: null"이 나온다.    
    System.out.println("poll: " + rq.poll());

    nyPizza = null; // 강한 참조 제거
    
    // GC 실행 요청.
    // 첫 번째 줄에서 생성한 인스턴스는 약한 참조만 존재하므로 GC 수거 대상이다.
    System.gc();    
    Thread.sleep(5000); // GC가 확실히 실행될 수 있도록 5초간 대기
        
    // get 메서드를 이용해서 이전에 가리키던 객체에 접근하려고 할 시 null이 반환된다.     
    System.out.println("weakPizza.get: " + weakPizza.get());
    
    // poll 메서드 호출하면 weakPizza 객체의 정보가 출력된다.
    System.out.println("poll: " + rq.poll());
}
```
**출력 결과**
```java
// GC 호출 전(인스턴스 수거 전)
weakPizza.get: hoyun06.Chapter_2.NyPizza@682a0b20
poll: null

// GC 호출 후(인스턴스 수거 후)
weakPizza.get: null
poll: java.lang.ref.WeakReference@214c265e
```

###### WeakHashMap에서 entry가 자동으로 제거되는 원리
우선 WeakHashMap 클래스의 put 메서드를 보자
```java
public V put(K key, V value) {
   Object k = maskNull(key);
   int h = hash(k);
   Entry<K,V>[] tab = getTable();
   int i = indexFor(h, tab.length);
   
   for (Entry<K,V> e = tab[i]; e != null; e = e.next) {
      if (h == e.hash && matchesKey(e, k)) {
          V oldValue = e.value;
          if (value != oldValue)
              e.value = value;
          return oldValue;
      }
   }
   
   modCount++;
   Entry<K,V> e = tab[i];
   tab[i] = new Entry<>(k, value, queue, h, e);
   if (++size >= threshold)
      resize(tab.length * 2);
   return null;
}
```
복잡해 보인다.  
간단히 말하자면 기존에 있던 Entry에 대해선 value값만 업데이트하고  
처음 넣는 Entry라면 새로운 Entry 객체를 생성한다. 이때, Entry의 생성자가 호출되면서 key값을 가리키는 약한 참조도 생긴다.

나중에 시간이 흘러서 외부에서 key값을 가리키던 강한 참조가 사라지면 바로 이전 소주제에서 다뤘던 것처럼
WeakReference 객체의 내부 참조는 null로 바뀌고 WeakReference 자체는 ReferenceQueue에 들어간다.

>여기선 내부적으로 참조하던 key가 null로 바뀌고 
**Entry 인스턴스가 ReferenceQueue에 들어간다**  

ReferenceQueue에는 Reference 객체들이 들어간다고 했는데 위에서 봤듯이 Entry 클래스는 WeakReference 클래스를 상속하므로
Entry 인스턴스가 ReferenceQueue에 들어가게 된다.

그리고 WeakHashMap에서 해당 Entry가 실질적으로 제거되는 시점은 아래 메서드가 실행되는 시점이다.
```java
private void expungeStaleEntries() {
    for (Object x; (x = queue.poll()) != null; ) {
        synchronized (queue) {
            Entry<K,V> e = (Entry<K,V>) x;
            int i = indexFor(e.hash, table.length);

            Entry<K,V> prev = table[i];
            Entry<K,V> p = prev;
            while (p != null) {
                Entry<K,V> next = p.next;
                if (p == e) {
                    if (prev == e)
                        table[i] = next;
                    else
                        prev.next = next;
                    e.value = null; // Help GC
                    size--;
                    break;
                }
                prev = p;
                p = next;
            }
        }
    }
}
```
이 메서드는 WeakHashMap 클래스 내부 private 메서드다.  
이 메서드는 개발자가 직접 호출하진 못하고 size 메서드나 getTable 메서드를 통해 호출된다.

###### 정리
1. WeakHashMap에 key, value 쌍을 put(내부적으로 Entry 객체 생성되어 해시 테이블에 유지)
2. key값을 참조하는 WeakReference 객체 생성
3. 이후에 코드 상에서 해당 key값의 강한 참조가 모두 사라짐.
4. 해당 key값을 참조하던 WeakReference가 ReferenceQueue에 들어감(이 경우엔 Entry 객체)
5. WeakHashMap 클래스의 expungeStaleEntries 메서드 실행
6. ReferenceQueue에 들어있는 모든 Entry 객체들을 해시 테이블에서 제거
