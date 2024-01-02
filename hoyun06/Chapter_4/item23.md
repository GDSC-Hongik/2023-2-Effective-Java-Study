### Item23 : 태그 달린 클래스보다는 클래스 계층 구조를 활용하라

###### 태그 달린 클래스
한 클래스가 여러 상태를 나타낼 수 있으며 해당 상태는 클래스 내부에 있는 태그 필드를 통해 나타내는 클래스
```java
public class Figure {
    
    enum Shape { RECTANGLE, CIRCLE };
    
    // 태그 필드
    final Shape shape;
    
    // RECTANGLE용 필드
    double length;
    double width;
    
    // CIRCLE용 필드
    double radius;
    
    // RECTANGLE용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }
    
    // CIRCLE용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }
    
    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```
위 클래스는 
- 열거 타입 선언, 태그 필드, switch문 등 불필요한 코드들이 많다.
- 필드를 final로 선언하려면 불필요한 필드까지도 초기화하는 코드가 포함되어야 한다.
- 새로운 의미를 나타내려면 코드를 수정해야 한다(ex. OVAL 추가 시 switch문 수정).

**정리하면 태그 달린 클래스는 장황하고, 오류를 내기 쉬우며 비효율적이다.**

이를 해결하고자 객체지향 언어에서는 클래스 계층 구조를 사용할 수 있다.

###### 태그 달린 클래스를 클래스 계층 구조로 변경하기
이건 우리가 흔히 상속에서 배우는 그대로 설계하면 된다.
1. 계층 구조의 root가 될 추상 클래스를 선언한다
   - 공통 필드, 공통 메서드를 추상 클래스에 선언/구현한다.
   - 태그 값에 따라 동작이 달라지는 메서드의 경우 추상 메서드로 선언만 한다.
2. 각각의 의미를 나타내는 구체 클래스를 생성하고 root 클래스를 상속한다
   - 각각의 구체 클래스에 필요한 필드들을 추가하고 추상 메서드를 본인의 목적에 맞게 구현한다.
```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}    
 
class Rectangle extends Figure {
    final double length;
    final double width;
    
    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    @Override
    double area() {
        return length * width;
    }
}
```
클래스 계층 구조는
- 각 클래스에는 필요한 필드만 있으므로 final을 붙일 수 있고 이에 대한 초기화 여부도 컴파일 타임에 체킹된다.
- 타입 사이의 계층 관계를 명확히 나타낼 수 있다.
- 확장성이 좋다.