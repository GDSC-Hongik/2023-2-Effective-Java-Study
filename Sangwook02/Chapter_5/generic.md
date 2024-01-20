# Generic

```java
public <T> T methodName(T t) {
    return t;
}
```
제네릭 메서드는 다른 메서드와 선언이 조금 다르다.  
접근 지정자 다음에 반환 타입이 바로 오지 않고, 제네릭 매개변수 목록이 먼저 온다.  
위의 경우 <T>는 타입 매개변수 목록이고, 반환 타입은 T이다.  
타입 매개변수가 있기 때문에 다양한 타입에 대해 동작할 수 있고, 이를 통해 재사용성을 높일 수 있다.

## 타입 매개변수를 다룰 때 주의할 점
```java
public class Person<T> {
    static <T> T getPerson(T t) {
        return t;
    }
}
```
위의 Person 클래스는 타입 매개변수 T를 사용하고, getPerson 메서드는 타입 매개변수 T를 사용한다.  
같은 T지만, 둘은 독립적이다.  
Person 클래스의 T는 클래스 레벨에서 선언된 타입 매개변수로, 
클래스 내에서 사용되는 T는 대체로 이 T와 같은 타입이다.  
예를 들어, 아래와 같은 경우에는 Person 클래스의 T와 getPerson 메서드의 T는 같은 타입이다.
```java
public class Person<T> {
	
	static T getPerson(T t) {
        return t;
    }
}
```
그렇다면, 아까 다르다던 둘의 T는 왜 다를까?  
그 이유는 제네릭 메서드에서의 T는 지역변수이기 때문이다.  
지역변수이기 때문에 메서드가 호출되는 시점에 실제 타입으로 대체된다.

## 제네릭 메서드는 static 사용 가능
만약 아래와 같은 클래스가 있다고 하자.  
```java
public class Person<T> {
	static T name;
	
    static T getPerson(T t) {
        return t;
    }
}
```
컴파일 에러가 발생할 것이다.  
static은 인스턴스가 생성되기 전에 호출될 수 있는데, 타입 매개변수 T는 인스턴스가 생성될 때 대체된다.  
T가 어떤 타입인지 결정되지 않았기 때문에 위의 코드에서 문제가 생기는 것이다.  

반면에, 제네릭 메서드는 static으로 사용할 수 있다.  
앞서 제네릭 메서드의 타입 매개변수 T는 지역변수라는 것을 언급했다.  
바로 제네릭 메서드를 static으로 사용 가능한 이유를 설명하기 위함이었다.  
지역 변수이므로 호출시에 타입이 결정되기 때문에 static으로 사용 가능하다.
```java
public class Person<T> {
    static <T> T getPerson(T t) {
        return t;
    }
}

public static void main(String[] args) {
    Person<String> person = new Person<>();
	/*
        getPerson 메서드는 static으로 선언되어 있기 때문에
        인스턴스 생성 없이 호출 가능하고, 호출 되는 순간에 매개변수의 타입이 결정된다.
	 */
    String name = person.getPerson("chamsae");
    System.out.println(name);
}
```

##### reference
- [제네릭 메서드](https://devlog-wjdrbs96.tistory.com/201)