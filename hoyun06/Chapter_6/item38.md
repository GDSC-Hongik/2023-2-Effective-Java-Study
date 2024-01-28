### Item38 : 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

###### 다 좋은데 열거 타입의 단점이 있다면?
열거 타입은 확장이 불가능하다.  
열거 타입의 주 목적은 상수를 표현하는 것인데 만약 열거 타입을 확장할 수 있다면 문제가 발생한다. 
- 특정 열거 타입을 상속받은 또 다른 열거 타입의 상수는 두 열거 타입의 타입으로 인식되지만 base 열거 타입의 상수는 그 하위 열거 타입으로 인식되지 않게 된다.
- base 열거 타입과 하위 열거 타입의 상수를 모두 순회하는 것이 쉽지 않다.

###### 열거 타입을 확장 개념과 같이 사용할 수 있는 경우
열거 타입을 직접적으로 확장하는 것은 아니지만 '열거 타입은 인터페이스를 구현할 수 있다.'라는 특성을 이용하면 확장의 느낌을 살릴 수 있다.
```java
public interface Operation {
    double apply(double x, double y);
}
```
```java
public enum BasicOperation implements Operationn {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };
    
    private final String symbol;
    
    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
    
    @Override
    public String toString() {
        return symbol;
    }
}
```
연산 코드의 연산을 나타내는 Operation 인터페이스를 선언하고 그 안에 apply 메서드를 선언한다.

그리고 이 인터페이스를 구현하는 열거 타입을 선언하고 그 안에 원하는 상수들도 선언한 뒤 목적에 맞게 메서드를 구현하면 된다.  
물론, 이 경우에도 BasicOperation이라는 열거 타입을 직접 확장하는 것은 아니다. 하지만 만약 새로운 연산을 추가하고 싶다면 Operation 인터페이스를 구현하는
열거 타입을 생성하여 상수만 추가해주면 된다.  
```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    
    private final String symbol;
    
    // 생략
}
```
그럼 기존 클라이언트 코드에서 BasicOperation을 직접 사용하지 않고 Operation 인터페이스를 사용한 경우에는 어디서든 새로 추가된 상수를 바로 사용할 수 있게 된다.

###### 이 방식에서의 단점?
위 방식은 굉장히 편하고 확장성도 있지만 한 가지 단점이 있다. 바로 하위 열거 타입끼리는 메서드 구현을 공유할 수 없다는 것이다.  
만약 하위 열거 타입들이 구현하는 메서드가 '상태'에 의존적이지 않다면 인터페이스의 디폴트 메서드로 추가하여 공유할 수 있지만 그렇지 않다면 
디폴트 메서드로 제공하는 것도 쉽지 않다.

## 핵심 정리
    열거 타입은 상속이 불가능하다. 하지만 인터페이스와 함께 사용한다면 확장을 흉내낼 수 있다.