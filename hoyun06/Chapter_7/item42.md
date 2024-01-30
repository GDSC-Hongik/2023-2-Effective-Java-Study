### Item42 : 익명 클래스보다는 람다를 사용하라

###### 익명 클래스
예로부터 자바에서는 함수 타입을 나타낼 때 추상 메서드를 하나만 담은 인터페이스를 이용했다. 
해당 인터페이스의 인스턴스를 함수 객체라고 하며 이 함수 객체를 만드는 주요 수단으로 익명 클래스를 사용했다.
```java
public static void main(String[] args){
    Collections.sort(words, new Comparator<String>() {
        public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
        }
    });
}
```
위 코드는 익명 클래스의 예시다. 하지만 보다시피 코드가 너무 장황해진다.

###### 함수형 인터페이스. 람다
그래서 자바 8에서는 추상 메서드 1개짜리 인터페이스는 함수형 인터페이스라는 특별 취급을 받게 되었고
우리는 이 인터페이스의 인스턴스를 `람다식`을 사용해 만들 수 있게 되었다.
```java
public static void main(String[] args){
    Collections.sort(words, (s1, s2) -> 
    Integer.compare(s1.length(), s2.length()));
}
```
람다식을 잘 보면 타입이 어디에도 나와있지 않다. s1, s2는 `String` 타입, 람다식 자체는 `Comparator<String>` 타입 그리고
compare 메서는 `int` 타입을 반환하는데 이는 컴파일러 선에서 모두 타입 추론이 가능하다.

이전 주차 item 34 열거 타입을 설명하면서 열거 타입 내부 상수마다 다른 구현 내용을 가지게 하기 위해선 열거 타입 내부에 추상 메서드를 선언하고
각 상수가 자신의 목적에 맞게 구현하도록 한 방법을 기억할 것이다.  

그 대신에 각 상수마다 필요한 구현을 람다식으로 구현한 뒤 이를 인스턴스 필드에 저장하도록 하고 필요할 때 해당 람다식을 호출만 하는 형태로
더욱 간결하게 구현할 수 있다.
```java
public enum Operation {
    PLUS("+", (x, y) -> x + y);
    MINUS("-", (x, y) -> x - y);
    TIMES("*", (x, y) -> x * y);
    DIVIDE("/", (x, y) -> x / y);
    
    private final String symbol;
    private final DoubleBinaryOperator op;
    
    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }
    
    @Override
    public String toString() {
        return symbol;
    }
    
    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

###### 그럼 람다식이 간편하니 항상 사용하는 것이 좋겠군요
그렇지 않다.  
- 람다식은 이름도 없고 문서화도 하지 못한다. 그렇기에 코드가 길어지거나 람다식으로 교체한 코드가 스스로의 의미를 제대로 표현하지 못한다면 
람다식을 사용하지 않는 편이 나을 수도 있다.  
- 그리고 람다식은 함수형 인터페이스의 인스턴스를 만들 때만 사용된다. 
추상 클래스의 인스턴스를 만들 때는 익명 클래스를 여전히 사용해야 한다.  
- 그리고 람다식은 this를 제대로 사용할 수 없다. 람다식에서의 this는 바깥 인스턴스를 가리킨다. 반면 익명 클래스에서의 this는 자기 자신을 정확히 가리킨다.

## 핵심 정리
    함수형 인터페이스의 인스턴스 생성 시 람다를 이용하자. 
    익명 클래스는 함수형 인터페이스가 아닌 인스턴스를 생성할 때 이용하자. 