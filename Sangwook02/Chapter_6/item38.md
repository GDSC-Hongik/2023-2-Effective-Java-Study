# Item 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

열거 타입보다 타입 안전 열거 패턴이 더 나은 예외적인 상황  
- 타입 안전 열거 패턴은 확장이 가능
- 그러나 열거 타입을 확장하는 것은 좋지 않은 생각
```java
public interface Operation {
    double apply(double x, double y);
}
```

```java
public enum BasicOperation implements Operation {
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

    @Override public String toString() {
        return symbol;
    }
}
```
위의 코드는 Operation 인터페이스를 구현한 열거 타입이다.
만약 새로운 연산을 추가하고 싶다면, Operation 인터페이스를 새로 구현하면 된다.
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

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```
새로 만든 ExtendedOperation은 BasicOperation이 아니라 Operation을 구현하므로, 
Operation을 사용하던 곳에서 큰 문제없이 사용할 수 있다.  

### 문제점
문제가 되는 부분이 있다면, 열거 타입끼리 구현을 상속할 수 없다는 점이다.  
공유하는 부분이 많다면, 별도의 클래스나 메서드로 분리하면 됨.
