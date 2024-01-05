java 8부터는 디폴트 메서드가 도입되면서, 기존 구현체를 깨뜨리지 않고, 인터페이스에 메서드를 추가하는 방법이 생겼다.
이 인터페이스를 상속받은 클래스가 디폴트 메서드를 재정의하지 않으면 이 클래스에서는 디폴트 구현이 쓰이게 된다.

## 디폴트 클래스의 문제점
```java
default boolean removeIf(Predicate<? super E> filter) {
	Objects.requireNonNull(filter);
    
    boolean result = false;
    for (Iterator<E> it = iterator(); it.hasNext(); ){
    if(filter.test(it.next()){
    	it.remove();
        result = true;
     }
   }
   return result;
}
```
이 코드는 `Collection`인터페이스에 추가된 디폴트 메서드이다. 하지만 이를 구현한 모든 `Collection`구현체와 잘 어우러지지는 않는다. 대표적으로 `SynchronizedCollection`은 위 `removeIf`와 어우러지지 못해 디폴트 클래스를 재정의하여 구현한다.

따라서 인터페이스의 디폴트 메서드가 모든 구현체와 어우러지기는 힘들고, 이럴 경우에는 `removeIf` 를 재정의하여, 디폴트 구현을 호출하기 전에 동기화 하도록 설계한다.

### 디폴트 메서드는 오류를 일으킬 가능성이 있다.
디폴트 메서드가  컴파일을 성공하였더라도, 이를 구현하는 구현체에서 런타임 오류를 발생시킬 수 있으므로, 기존 인터페이스에 새 메서드를 추가하는 일은 피해야 한다.