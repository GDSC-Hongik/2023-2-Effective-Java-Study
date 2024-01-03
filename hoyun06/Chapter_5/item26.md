### Item26 : 로 타입은 사용하지 말라

###### 제네릭 클래스, 제네릭 인터페이스, 제네릭 타입
클래스 혹은 인터페이스 선언부에 타입 매개변수를 사용하면 `제네릭 클래스` 혹은 `제네릭 인터페이스`라고 한다.  
그리고 이를 통틀어 `제네릭 타입`이라고 한다.

실제로 해당 제네릭 타입을 사용할 때는 먼저 클래스 이름을 적고 꺽쇠('<', '>') 안에 실제 타입 매개변수를 적는다.
```java
List<String> names = new ArrayList<>();
```

그리고 제네릭 타입을 하나 선언하면 그에 딸린 로 타입도 같이 정의된다.  
로 타입 : 제네릭에서 타입 매개변수를 사용하지 않는 경우를 말한다.
```java
private final List names = ...;
```
이 경우 names에는 String만 넣도록 구현하지만 실제로는 어느 타입도 들어갈 수 있다. 실수로 Integer를 넣어도 문제가 생기지 않으며
런타임에서 해당 Integer를 꺼내서 String으로 타입 캐스팅을 하는 시점에서야 에러가 발생함을 인지할 수 있다.
```java
for (Iterator i = names.iterator(); i.hasNext(); ) {
    String name = (String) i.next(); // ClassCastException 발생!
}
```
그러므로 로 타입 대신에 아래와 같이 타입 매개변수를 명시하면 컴파일 타임에 잘못된 타입이 들어왔음을 인식할 수 있다.
```java
private final List<String> names = ...;
```
이렇게 작성하면 컴파일러는 위 컬렉션에서 원소를 꺼내는 모든 부분에 암묵적으로 타입 캐스팅 코드를 삽입한다. 그렇기 때문에 컴파일러에 의해
아무런 오류가 발생하지 않았다면 제대로 된 원소들만 들어갔음을 확신할 수 있다.

###### 그렇다면 로 타입을 사용할 수 있게 열어둔 이유는 뭔가요?
자바 진영에서 제네릭을 수용하기까지 꽤나 긴 시간이 걸리게 되면서 제네릭이 적용되지 않은 코드의 분량이 넘치게 되었다.  
쉽게 말하면 제네릭이 도입되면서 그 이전에 작성되었던 비제네릭 코드와의 호환성을 위해 로 타입을 남겨둔 것이다.

###### 임의 객체를 허용하는 매개변수화 타입
List names = .. 와 같이 로 타입을 사용하는 것은 위험하지만 
```java
List<Object> list = ... 
``` 
와 같이 사용하는 것은 괜찮다.
- 차이점
  - 로 타입은 아예 제네릭 관련 내용을 제외한 것이고, `List<Object>`는 어떤 객체라도 허용하겠다는 것을 컴파일러에게 알려주는 것이다.
  - 매개변수로 로 타입 List를 받는 메서드에 `List<String>`을 넘기는 것은 가능하나 매개변수로 `List<Object>`를 받는 메서드에는 `List<String>`을 넘길 수 없다.
    `List<String>`은 로 타입 List의 하위 타입으로 인식하지만 `List<Object>`의 하위 타입은 아니기 때문이다.

###### 비한정적 와일드카드
로 타입을 대신해서 사용할 수 있는 것이 있다. `비한정적 와일드카드`다.
```java
List<?> list
```
위와 같이 사용하는데 제네릭 타입을 사용하고 싶으면서도 안에 들어있는 구체 타입에 대해선 신경쓰고 싶지 않을 때 사용한다.
```java
public class TestClass {
  
    public void test(List<?> testList) {
        System.out.println(testList.size());
        String s = "hello";
        testList.add(null);
    }
}
```
test 메서드에서는 비한정적 와일드카드를 사용한 List를 받으므로 실제 타입 매개변수에 상관없이 모든 List를 받을 수 있다.  
단, testList로 받은 리스트에는 null 이외의 원소를 추가할 수 없다. ?를 사용했기 때문에 세부 타입을 알 수 없기 때문이다.

###### 로 타입 사용에 대한 사소한 예외
- Class 리터럴을 사용하는 경우에는 배열과 기본 타입을 제외한 모든 경우에는 로 타입을 사용해야 한다.
  ```java
  List.class, String.class, String[].class, int.class // 허용
  List<String>.class, List<?>.class                   // 불허
  ```
- instanceof 연산자 사용하는 경우에는 로 타입을 사용하는 것이 낫다.
  - 런타임에는 제네릭 정보가 지워진다. 따라서 instanceof 연산자는 비한정적 제네릭 타입 이외의 매개변수화 타입에는 적용할 수가 없다.
```java
if (o instanceof Set) {
    Set<?> s = (Set<?>) o;
}
```

## 핵심 정리
    로 타입은 안전하지 않다. 로 타입의 목적은 비제네릭 코드와의 호환성을 제공하기 위함이다.
    어떠한 원소라도 담고싶은 컬렉션을 사용하고 싶다면 Object를 실제 타입 매개변수로 가지는 제네릭을 선언하자.
    제네릭을 사용하면서 실제 타입에 대해선 신경쓰고 싶지 않다면 ?를 사용한 비한정적 와일드카드를 사용하자.


### 와일드 카드
###### 와일드 카드의 필요성
자바의 객체 및 배열은 상하관계가 존재한다. 그리고 이에 대해 공변성을 제공한다.
```java
class Car {
    Car() {}
}
class RaceCar extends Car {
    RaceCar() {
        super();
    }
}
public class Main {
    public static void main(String[] args) {
        Car car = new RaceCar();
		
        Object[] objects = new Integer[10];
    }
}
```
상위 클래스인 Car 참조 변수로 하위 클래스인 RacingCar를 가리킬 수 있으며 상위 클래스인 Object 배열로 Integer 배열을 가리킬 수 있다.

하지만 제네릭 클래스의 경우 상하관계가 성립하지 않고 그에 따라 공변성도 제공하지 않는다.
```java
ArrayList<Car> cars = new ArrayList<>();                
ArrayList<RaceCar> raceCars = new ArrayList<>();        
cars = raceCars; // 컴파일 에러

ArrayList<Object> objects = new ArrayList<String>(); // 컴파일 에러
```
그래서 제네릭한 코드에서도 공변성을 제공하는 수단으로 사용할 수 있는 것이 `와일드카드`다.

###### 비한정적 와일드카드
```java
List<?> unknownList = ...;
```
아무런 경계를 설정하지 않은 와일드카드. 와일드카드를 이용함으로써 어떤 타입 매개변수에 대해서도 가리킬 수 있게 된다.
여기서 '?'를 `any type`보다는 아직 정해지지 않았다는 의미인 `unknown type`이라고 생각하자. '**아직 정해지지 않았기에
가리키는 것에 있어서 제약이 없다**'정도로 이해하면 될 것 같다.

###### 한정적 와일드카드
특정 타입을 기준으로 상한 및 하한 범위를 지정함으로써 말그대로 boundary를 설정할 수 있는 와일드카드다.  

**상한 경계 와일드카드**
- 와일드카드 타입에 extends를 사용해서 상한 경계를 설정한다.
  ```java
  public void print(Collection<? extends Car> c) {
      for (RaceCar r : c) {   // 컴파일 에러
          ...
      }
      for (Car r : c) {       // 컴파일 성공
          ...
      }
      for (Object o : c) {   // 컴파일 성공
          ...
      }
  }
  ```
  맨 첫 번째 for-each 문은 컴파일 에러가 발생한다. Collection에는 Car 및 Car의 하위 클래스가 들어갈 수 있는 것은 맞다. 
  하지만 안에 있는 원소가 항상 RaceCar 타입이라는 보장은 없다. 막말로 Car를 상속하는 또 다른 하위 클래스인 FlyingCar 타입일 수도 있는 것이다.
  ```java
  public void add(Collection<? extends Car> c) {
      c.add(new RaceCar());        // 컴파일 에러
      c.add(new Car());            // 컴파일 에러
      c.add(new Object());         // 컴파일 에러
  }
  ```
  c의 경우 Car와 Car를 상속하는 모든 하위 클래스 타입만 들어갈 수 있다고 경계를 설정한 것이지 실제로 들어있는 원소타입이 정해진 것이 아니다. 그래서 
  첫 번째와 두 번째 add 메서드 모두 컴파일 에러를 발생시킨다. 세 번째 add같은 경우 Object 타입을 넣으려고 하지만 애초에 Object는
  `extends Car` 를 만족하지 못한다.

**하한 경계 와일드카드**
- 와일드카드 타입에 super를 사용해서 하한 경계를 설정한다.
  ```java
  public void add(Collection<? super Car> c) {
      c.add(new RaceCar());        // 컴파일 성공
      c.add(new Car());            // 컴파일 성공
      c.add(new Object());         // 컴파일 에러
  }
  ```
  첫 번째와 두 번째 add메서드는 정상적으로 컴파일된다. c는 Car와 Car의 부모 타입이라면 담을 수 있는 컬렉션이므로 안에 들어가는 원소는
  최소한 Car 타입임은 보장되기 때문이다. 반면 세 번째 add는 에러가 발생한다. Car의 부모 타입이라고 해서 항상 Object 타입이라는
  보장이 없기 때문이다. 막말로 c에 Car의 부모 타입인 Vehicle이라는 원소들이 담겨있을 수도 있는데 이때는 Object를 넣을 수 없다.
  ```java
  public void print(Collection<? super Car> c) {
      for (RaceCar r : c) {   // 컴파일 에러
          ...
      }
      for (Car r : c) {       // 컴파일 에러
          ...
      }
      for (Object o : c) {   // 컴파일 성공
          ...
      }
  }
  ```
  첫 번째, 두 번째 for-each문은 당연히 안된다. c에는 Car와 그 부모 타입만 들어갈 수 있다. 그걸 하위 타입인 Car 혹은 RaceCar로 캐스팅하는 것은 불가능하다.
  세 번째 for-each만 정상적으로 컴파일된다. 자바에서 Object는 모든 클래스의 부모 클래스이기 때문이다.