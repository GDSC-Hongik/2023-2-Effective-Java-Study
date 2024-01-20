### Item30 : 이왕이면 제네릭 메서드로 만들라

###### 제네릭 메서드
제네릭 메서드는 제네릭 클래스와 사실상 동일하게 선언할 수 있다. 
```java
public static <E> Set<E> union (Set<E> s1, Set<E> s2){
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```
**타입 매개변수 목록은 메서드의 제한자와 리턴 타입 사이에 반드시 와야 한다.**
그 이외에는 제네릭 클래스와 동일하게 사용하면 된다.

###### 제네릭 싱글턴 팩터리
상태를 가지지 않는 불변 클래스를 여러 타입으로 이용하고 싶은 경우 각 타입에 대해서 인스턴스를 매번 생성하는 것은
불필요한 작업이다. 하나의 인스턴스를 만들어 놓고 때에 따라서 원하는 타입으로 캐스팅해서 사용할 수 있는 방법이 있는데
바로 `제네릭 싱글턴 팩터리` 방식이다.
```java
public class Collections {
    public static final Set EMPTY_SET = new EmptySet<>();
    
    public static final <T> Set<T> emptySet() {
        return (Set<T>) EMPTY_SET;
    }
}
```
위 코드는 실제 Collections 클래스에서 가져온 예시 코드다. emptySet 메서드가 `제네릭 싱글턴 팩터리`를 사용했다.

###### 재귀적 타입 한정
자기 자신의 표현식을 이용하여 타입 매개변수에 들어갈 수 있는 타입을 제한하는 방식이다.  
이 방식은 주로 Comparable에서 사용된다. 
```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if(c.isEmpty()) {
        throw new IllegalArgumentException("컬렉션이 비었습니다.");
    }
    
    E result = null;
    for (E e : c) {
        if (result == null || e.compareTo(result) > 0) {
            result = Objects.requireNonNull(e);
        }
    }
    return result;
}
```
Comparable을 구현한 컬렉션을 매개변수로 받는 메서드는 대부분이 해당 컬렉션을 정렬하거나 최대값/최소값을 찾거나
검색하는 용도로 사용한다. 그러기 위해선 원소 간의 비교가 가능해야 하므로 사실상 같은 타입의 원소들만이
해당 컬렉션 안에 들어있어야 한다. 이런 제약을 적용할 수 있는 것이 위에서 본 재귀적 타입 한정이다.

## 핵심 정리
    클라이언트가 직접 타입 캐스팅을 하는 일반 메서드보다 제네릭 메서드가 타입 안전하다.