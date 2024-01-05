# Item 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

두 가지 이상의 의미를 표현할 수 있고, 현재 표현하는 의미를 태그 값으로 알려주는 클래스를 태그 클래스라고 한다.  
```java
class Figure {
	enum Shape {RECTANGLE, CIRCLE};
	
	final Shape shape;
	
	// rectangle에서만 사용하는 필드
	double length;
	double width;
	
	// circle에서만 사용하는 필드
	double radius;
    
    Figure(double radius) {
        this.shape = Shape.CIRCLE;
		this.radius = radius;
    }

	Figure(double length, double width) {
		this.shape = Shape.RECTANGLE;
		this.length = length;
		this.width = width;
	}
	
    double area() {
        switch (shape) {
            case RECTANGLE -> {
                return length * width;
            }
            case CIRCLE -> {
                return 3.14 * (radius*radius);
            }
            default -> {
                throw new AssertionError(shape);
            }
        }
    }
}
```

위의 경우, 여러 단점이 있다.
- 열거 타입 선언, 태그 필드, switch문 등 불필요한 코드가 잔뜩
- 여러 구현이 한 클래스에 섞여있음 -> 가독성 저하
- 필드들을 final로 하려면 Rectangle을 정의하려면 radius도 초기화해줘야 하는 문제

=> 장황, 오류 만들기 쉽고, 비효율적

### 개선방안: 클래스 계층구조
최상위 클래스인 루트 클래스를 추상 클래스로 정의하자.  
- 기존 코드에서 태그에 따라 달라지는 메서드들은 추상 메서드로 정의
- 태그와 무관하게 같은 동작을 하는 메서드들은 일반 메서드로 정의
- 모든 하위 클래스에서 공통으로 사용하는 필드만 남기고 모두 제거

```java
abstract class Figure {
	abstract double area();
}

class Rectangle extends Figure {
	double length;
	double width;

	Rectangle(double length, double width) {
		this.length = length;
		this.width = width;
	}

	@Override
	double area() {
		return length * width;
	}
}
```
각 의미를 독립된 클래스에 담아 관련 없던 데이터를 제거  
-> 남은 필드는 모두 final

**태그 필드가 있다면 계층구조로 대체하도록.**