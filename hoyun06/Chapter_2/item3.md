### Item3 : private 생성자나 열거 타입으로 싱글턴임을 보증하라

###### 싱글턴이란?
    인스턴스를 오직 하나만 생성할 수 있는 클래스
그런데 클래스를 싱글턴으로 설계하면 클라이언트 코드에서는 이를 테스트하기가 어렵다.  
**why?**   
이따 뒤에 나오겠지만 싱글턴으로 설계하기 위해서는 해당 클레스의 생성자를 private으로 지정한다.
따라서 해당 클래스가 특정 인터페이스를 구현한 싱글턴 클래스가 아니라면 mock 객체를 생성하는 것은 쉽지가 않다.  
(mocking framework는 참고할 인터페이스가 있다면 비교적 쉽게 mock 인스턴스를 생성할 수 있다.)

###### 싱글턴을 달성할 수 있는 방법
`1. private 생성자 + public static final 인스턴스 필드`
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {..}
}
```
- private 생성자는 INSTANCE를 초기화할 때 딱 한 번만 실행된다.
- private 생성자이고 겉으로 드러나는 INSTANCE가 static final이므로 싱글턴임이 보장된다.
- 단, 클라이언트 코드에서 리플렉션 기능을 이용하여 private 생성자를 호출할 순 있다.  
  이를 막기 위해 생성자 내부에서 INSTANCE의 초기화 여부를 판단하고 exception을 던지면 된다.

`2. private 생성자 + private static final 인스턴스 필드 + public 정적 팩터리 메서드`
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {..}
    
    public static Elvis getInstance() {
        return INSTANCE;
    }
}
```
- 마찬가지로 private 생성자는 INSTANCE를 초기화할 때 딱 한 번만 실행된다.
- private 생성자이고 INSTANCE가 static final이므로 싱글턴임이 보장된다.
- 여기서도 리플렉션에 의한 위험은 여전히 존재한다.

**2번 방식**의 장점
1. 언제든지 싱글턴 방식이 아니도록 수정할 수 있다.
   getInstance 메서드에서 매번 새로운 인스턴스를 생성하여 반환하도록 하면 된다.
2. 원한다면 정적 팩터리를 제네릭 싱글턴 방식으로 변경할 수 있다.
3. 정적 팩터리의 메서드 참조를 Elivis::getInstance와 같이 Supplier\<Elvis>로 넘길 수 있다.

###### 위 두 가지 방식의 단점
위 두 가지 방식을 이용하여 싱글턴을 달성하면 직렬화/역직렬화 과정에서 매번 새로운 인스턴스가 생성된다.  
단순히 Serializable을 implement한다고 선언하는 것은 이 문제를 막을 수 없다.

###### 그럼 어떻게 해야 하나요...
- 우선은 모든 멤버 필드에 transient 키워드를 붙여 직렬화 과정에서 해당 필드들을 제외한다고 선언을 해줘야 한다.  
  (스프링에서 쓰는 @Transient아님!)
- 그 다음에 해당 클래스 내부에 readResolve 메서드를 직접 정의하고 역직렬화 과정에서 새로 생성된 인스턴스 대신 기존에 싱글턴 방식으로 생성한
  INSTANCE를 반환하도록 명시해줘야 한다.
```java
// Elvis 클래스 내부에 정의해주자
private Object readResolve() {
    return INSTANCE;    
}
// 위 메서드를 Elvis 클래스에 정의하면 deserializing 과정에서 deafault readObject 메서드가 호출되어서   
// 새로운 인스턴스가 생성되더라도 실제로 그것을 반환하지 않고 readResolve 메서드에서 반환하는 INSTANCE를 반환할 수 있다.
```
###### 너무 귀찮은데 다른 방법은 없나요?
있다. 위 두 가지 방법 말고도 한 가지 방법이 더 있다.  
`원소가 하나인 열거 타입 선언하기`
```java
public enum Elvis {
    INSTANCE;
    public void action() {..}
}
```
- 위 1번 방식과 유사한 방법이지만 추가적인 작업 없이도 직렬화/역직렬화 과정에서 새로운 인스턴스가 생기는 것을 방지해주고
- 리플렉션으로 인한 위험성도 없다.
- 다만, 싱글턴으로 만들고자 하는 클래스가 Enum이 아닌 일반 클래스를 상속해야 한다면 이 방식은 사용할 수 없다.  
  (다른 인터페이스를 implement하는 것은 가능하다!)