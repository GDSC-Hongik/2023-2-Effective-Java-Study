### Item54 : null이 아닌, 빈 컬렉션이나 배열을 반환하라

###### 컬렉션과 배열의 null 반환
개발자들 중에 빈 컬렉션이나 빈 배열을 반환할 때 null을 반환하는 경우가 많다. 하지만 api 제공자 측에서 이렇게
null을 반환하게 되면 이를 사용하는 클라이언트 코드에서도 null 관련 처리를 직접 해줘야 하므로 제공자와 클라이언트 모두
null 관련 특별 처리를 해주는 상황이 발생하여 그리 효율적이지 않다.  
이를 위해 빈 컬렉션이나 빈 배열을 반환하는 것이 null 반환보다 성능 상의 불이익이 있을 것이라고 생각하지만 정말 특수한 경우가 
아니라면 성능에는 그렇게 큰 영향을 미치지 않는다고 한다.

###### 그렇다면 대안은
만약 반환하려고 하는 컬렉션 혹은 배열이 비어있다면 null 대신 빈 컬렉션 혹은 빈 배열을 반환하면 된다.
```java
public List<Car> getCars() {
    return carsInGarage.isEmpty() ? Collections.emptyList() 
        : new ArrayList<>(carsInGarage);
}
```
만약 빈 컬렉션을 매번 할당해서 반환하는 것이 마음에 걸린다면 `빈 불변 컬렉션`을 반환하자. 불변 컬렉션은
공유해서 사용해도 안전하므로 Collections.emptyList()를 사용하여 빈 컬렉션을 반환해도 문제가 없다.

빈 배열을 반환해야 하는 경우에도 null 대신 크기가 0인 배열을 반환하자.
```java
private static final Car[] EMPTY_CAR_ARRAY = new Car[0];

public Car[] getCars() {
    return carsInGarage.toArray(EMPTY_CAR_ARRAY);
}
```
위 코드에서는 `<T> T[] List.toArray(T[] a)` 메서드를 사용하여 배열을 반환한다. 이 메서드의 경우
매개변수로 전달받은 배열의 크기가 충분히 크다면 해당 배열에 원소를 담아서 반환하고 그렇지 않다면 새로운 배열을 생성하여
그 안에 원소들을 담아 반환하게 된다. 따라서, 위 코드에서는 `carsInGarage`리스트가 원소를 1개이상 가지고 있다면 새로운 배열이
생성되어 반환될 것이고 비어있다면 `EMPTY_CAR_ARRAY`가 반환될 것이다.