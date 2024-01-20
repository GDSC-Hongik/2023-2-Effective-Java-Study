### Item31 : 한정적 와일드카드를 사용해 api 유연성을 높이라

###### 제네릭 코드의 허점
제네릭 코드는 item 28에서 언급했던 것처럼 **불공변**이다.
```java
public static void print(List<Object> objects) {...}

public static void main(String[]args) {
    print(new ArrayList<String>(); // 컴파일 에러
}
```
즉, 타입 매개변수 간의 상하관계가 분명함에도 캐스팅이 불가능하다는 소리다.
다행히도 이걸 허용할 수 있는 방법이 있다.

###### 한정적 와일드카드
한정적 와일드카드는 '?'를 사용하여 타입 매개변수의 boundary를 설정하는 것이라고 이해하면 된다.
```java
class Car {
    Car() {}
}

class RaceCar extends Car {
    RaceCar() {}
}
```
```java
public void print(Collection<? extends Car> c) {
    for (Car r : c) {
      System.out.println("Car: " + car);
    }
}
```
print 메서드처럼 매개변수로 들어온 제네릭 컬렉션에서 `produce`하는 경우에는 extends 키워드를 사용하여 상한 경계를 설정한다.
```java
public void add(Collection<? super Car> c) {
    c.add(new Car());
}
```
add 메서드처럼 매개변수로 들어온 제네릭 컬렉션에서 `consume`하는 경우에는 super 키워드를 이용하여 하한 경계를 설정한다.

이 공식을 일반화한 것이 `PECS(Producer-Extends, Consumer-Super)`다. 다만, 매개변수로 넘긴 제네릭 컬렉션이
producer와 consumer의 역할을 동시에 수행한다면 이때는 한정적 와일드카드를 사용하는 것이 오히려 독이 된다.  
그리고 매개변수에 한정적 와일드카드를 사용했다고 해서 해당 메서드의 리턴타입의 타입 매개변수에도 와일드카드를 적용하지 말자. 이는 오히려
클라이언트 코드에서도 와일드카드를 사용하도록 하는 꼴이 된다.

###### 타입 매개변수와 와일드카드
타입 매개변수와 와일드카드는 공통된 부분이 많아서 메서드 정의 시 둘 중 아무것이나 사용해도 된다.  
```java
public static <E> void swap(<List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```
위 코드는 실제로 동일하게 동작한다. 그렇다면 둘 중 어느 것이 더 나을까?  
이때 고려할 수 있는 기본적인 규칙을 책에선 다음과 같이 설명한다. 
> 메서드 선언 시 타입 매개변수가 한 번만 나온다면 와일드카드로 사용하라.  
> 비한정적 타입 매개변수는 비한정적 와일드카드로, 한정적 타입 매개변수는 한정적 와일드카드로 사용하라. 

그런데 와일드카드를 사용한 swap 메서드를 사용할 땐 주의해야할 점이 있다.
```java
public static void swap(List<?> list, int i, int j){
    list.set(i, list.set(j, list.get(i));
}
```
이와 같은 코드를 작성하면 에러가 발생한다. 왜냐하면 비한정적 와일드 카드를 적용한 컬렉션에는 `null`만을 삽입할 수 있기 때문이다.  
그래서 이때는 형변환을 위한 private 도우미 메서드를 활용하면 된다.
```java
public static void swap(List<?> list, int i, int j){
    swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j){
    list.set(i, list.set(j, list.get(i));
}
```
private 도우미 메서드에서는 타입 매개변수를 명시함으로써 list에 값을 정상적으로 넣을 수 있다.

## 핵심 정리
    PECS(Producer-Extends, Consumer-Super)