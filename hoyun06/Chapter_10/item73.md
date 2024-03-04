### Item73 : 추상화 수준에 맞는 예외를 던지라

저수준 예외를 그대로 상위 레벨까지 노출시킨다면 내부 구현을 노출하게 되는 것 뿐만 아니라 api 자체도
오염된다. 이를 방지할 수 있는 방법을 알아보자.

###### Exception Translation
저수준에서 발생한 예외를 적절히 catch 하여 고수준 예외로 변환하여 상위 레벨로 전파하는 방식이다.
```java
try {
    doSomething();
} catch (LowerLevelException e) {
    throw new HigherLevelException();
}
```
###### Exception Chaining
만약 저수준에서 발생한 예외에 대한 정보가 유지되어야 할 경우에는 예외 연쇄 방식을 사용할 수 있다.
```java
try {
    doSomething();
} catch (LowerLevelException e) {
    throw new HigherLevelException(e);
}

public class HigherLevelException extends Exception{
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}
```
이렇게 저수준에서 발생한 예외를 고수준 예외 생성 시 실어서 보낼 수가 있고 해당 고수준 예외 생성자에서는
super 를 통해 상위 클래스로 전달하게 된다. 그리고 나중에 저수준 예외에 대한 정보가 필요한 경우
Throwable의 getCause 와 같은 메서드로 접근할 수 있다.

## 핵심 정리
    저수준 예외를 노출하지 않으려면 예외 번역을 이용하자. 만약 예외 상황에 대한 근본적인
    원인을 유지해야 한다면 예외 연쇄를 사용하자.