### Item89 : 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라

###### 싱글턴 클래스
item 3에서 싱글턴 패턴을 소개하며 작성했던 클래스는 다음과 같다
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    
    public void leaveTheBuilding() {...}
}
```
보다 시피 생성자를 `private`으로 선언하여 외부에서의 객체 생성을 막은 '싱글턴 패턴'이다.  
하지만 위 Elvis 클래스가 `Serializable`을 구현하면 '싱글턴 패턴'이 깨지게 된다. 설령, 기본 직렬화 대신
커스텀 직렬화 형태를 설계하고 커스텀 readObject 메서드를 구현하여 호출하더라도 말이다.  
어느 방법을 쓰든 간에 직렬화/역직렬화 과정에서 Elvis 클래스가 처음 초기화되며 생성됐던 인스턴스와는 다른 인스턴스가 생성된다.  
그래도 이를 막을 수 있는 방법이 있는데 readResolve 메서드를 정의하는 것이다.

###### readResolve 메서드
readResolve 메서드를 이용하면 readObject 메서드에 의해 새로 생성된 객체를 다른 객체로 대체할 수 있다.  
역직렬화된 객체의 클래스가 readResolve 메서드를 적절히 구현했다면 역직렬화된 해당 객체를 인수로 하는 readResolve 메서드가 호출된다.
그리고 readResolve 메서드에선 해당 역직렬화된 객체 대신 다른 객체를 반환할 수 있고 이때 역직렬화된 객체는 GC의 수거 대상이 된다.
```java
public class Elvis implements Serializable {

    ...
    
    private Object readResolve() {
        return INSTANCE;
    }
}
```
위 코드처럼 readResolve 메서드를 추가하면 항상 `INSTANCE`를 반환하므로 역직렬화된 객체는 매번 GC 수거 대상이 된다.

###### 주의할 점
- readResolve 메서드를 인스턴스 통제 목적으로 사용한다면 해당 클래스 내부의 모든 객체 참조 필드는 'transient'로 선언해야 한다
  - 어차피 클래스 초기화 단계에서 생성된 객체를 항상 반환하므로 직렬화 형태에 내부 참조 객체에 대한 정보를 굳이 담을 필요가 없다
- readResolve 메서드를 사용한다고 해서 무조건 인스턴스 통제가 되는 것은 아니다
  - 이전 item에서 봤던 것처럼 악의적인 바이트 스트림을 이용한 공격에 노출될 수 있다.  
    (원래는 readResolve 메서드에서 GC의 수거 대상이 되었어야 할 역직렬화된 객체를 static 필드에 담아두었다가 나중에 악의적으로 사용하는 공격)

###### 원소가 하나인 열거 타입
그래서 더 나은 방법은 열거 타입을 이용하여 '직렬화 가능한 인스턴스 통제 클래스'를 작성하는 것이다.
```java
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs = new String[] {"a"};
    public void printSongs {...}
}
```

## 핵심 정리
    직렬화 가능한 인스턴스 통제 클래스가 필요하다면 원소가 하나인 열거 타입을 이용하자.
    어쩔 수 없이 readResolve 메서드를 사용한다면 해당 클래스의 모든 객체 참조 필드를
    transient로 선언하자

### 싱글턴(열거 타입)
싱글턴은 인스턴스를 오직 한 개만 가지는 패턴을 의미한다. 엄청 예전에 item 3번에서 싱글턴 패턴을 구현하는 방법에 대해 공부한 적이 있다.
- private 생성자 + public static 필드
- private 생성자 + private static 필드 + public static 팩터리 메서드
- 원소가 한 개인 열거형

이때 저자는 열거형을 이용한 방법을 추천했다. 그 이유는 다음과 같았다.
1. 리플렉션 공격으로부터 안전하다
2. 복잡한 직렬화 상황에서도 싱글턴을 유지할 수 있다

과연 열거 타입은 어떻게 직렬화/역직렬화가 되길래 싱글턴이 보장될까

### 열거 타입과 Enum 클래스
개발자들이 열거 타입을 새로 선언하는 경우 굳이 명시적으로 `Serializable`을 구현한다고 표시할 필요가 없다.
```java
public enum Car implements Serializable { // 불필요한 코드
    BMW, AUDI, HYNDAI, KIA 
}
```
왜냐하면 모든 열거형 타입은 기본적으로 `Enum` 이라는 추상 클래스를 base 클래스로 가지는데  
이 `Enum` 클래스가 `Serializable`을 구현하고 있기 때문이다.
```java
public abstract class Enum<E extends Enum<E>> implements Constable, Comparable<E>, Serializable {
    ...
}
```
그러므로 열거형도 직렬화/역직렬화가 가능은 하지만 그 매커니즘이 좀 다르다.

### 열거형의 직렬화
열거형은 알다 시피 상수로 구성된다. 

그래서 실제로 특정 상수를 직렬화하면 `ObjectOutputStream`은 해당 상수의 `name` 메서드를 호출하고 그 결과로 받은 'name' 정보만을
직렬화 스트림에 포함하고 해당 열거형에 들어있는 다른 필드들은 직렬화 스트림에 포함하지 않는다.

### 열거형의 역직렬화
직렬화 스트림을 이용하여 역직렬화를 시도하면 `ObjectInputStream`은 해당 스트림에서 'name' 정보를 읽어낸다. 그 뒤 해당 정보를 가지고
`Enum`클래스에서 제공하는 `valueOf`라는 메서드를 이용하여 해당 'name'에 대응되는 상수를 불러온다.

### 그렇다면 열거형 클래스 내부에 커스텀 직렬화를 구현한다면
이전 item에선 커스텀 직렬화를 고려하라고 언급하며 해당 클래스 내부에 `readObject`, `writeReplace`, `readResolve` 등을
구현하라고 했다.  
그럼 열거형에 대해서도 커스텀 직렬화를 사용하기 위해 위 메서드들을 열거형 클래스 내부에 정의하면 될까?

그렇지 않다. 열거형 클래스 내부에 위와 같은 메서드들을 직접 구현하더라도 실제 직렬화 과정에선 모두 무시된다. 

이에 더해 `serialVersionUID`을 임의의 값으로 재설정하더라도 이 또한 무시된다. 자바 내부 매커니즘에 따르면 모든 열거형은 `serialVersionUID`에 대해 `0L`을 고정적으로 가진다고 한다. 