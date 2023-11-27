# Item 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

싱글턴이란 단 하나의 인스턴스만 생성할 수 있는 클래스를 말한다.  
싱글턴을 만드는 방식은 3가지가 있다.  
1. static final 필드 방식
2. 정적 팩터리 방식
3. 열거 타입

### static final 필드 방식
```java
public class Example {
    public static final Example INSTANCE = new Example();
    private Example() {}
}
```
생성자를 private로 설정하여 외부 접근을 막고, public static final인 필드를 통해서만 접근할 수 있도록 한다.  
private 생성자는 public static final Example INSTANCE를 초기화하는 시점에서 딱 한 번만 호출된다.  

싱글턴임이 명백히 드러나고 간결하다는 점이 장점이다.

### 정적 팩터리 방식
```java
public class Example {
    private static final Example INSTANCE = new Example();
    private Example() {}
    public static Example getInstance() {
        return INSTANCE;
    }
}
```
앞선 방식과 유사하지만 이번에는 메서드를 통해서만 접근할 수 있다.  
getInstance() 메서드는 항상 같은 객체의 참조를 반환하므로 싱글턴이다.  

API를 바꾸지 않고도 싱글턴이 아니게 바꿀 수 있다는 점에서 유연함하다고 할 수 있고, 원한다면 정적 팩터리 메서드를 제네릭 싱글턴 팩터리로 만들 수 있다는 점도 장점이다.  
메서드 참조로 사용할 수 있다는 점도 장점이다.

### 열거 타입
이번 item에서 가장 생소한 부분이다.  
원소가 하나인 Enum 타입으로 선언하는 방식이다.  
```java
public enum Example {
    INSTANCE;
    ...
}
```
인스턴스에 접근하기 위해 Example.INSTANCE를 하면 새로운 instance가 생기는 일 없이 싱글턴을 보증할 수 있다.