일단 태그가 무엇일까?
## 태그
태그란 해당 클래스가 어떤 타입인지에 대한 정보를 담고있는 맴버 변수를 의미한다.

다음은 원과 사각형에 대한 정보를 담고 있는 태그 달린 클래스를 의미한다.
```java
public class Figure {
    enum Shape {RECTANGLE, CIRCLE};

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    public FigureWithTag(double radius) {
        this.shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    public FigureWithTag(double length, double width) {
        this.shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
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

그러나 나도 이러한 구조를 본적이 없지만, 이런 경우를 발견하면 계층구조를 사용하자.

그럼 어떤 이유로 태그달린 클래스말고 계층구조를 사용해야 할까?
## 태그 달린 클래스
- 여러 구현이 하나의 클래스에 혼합되어 있어 가독성이 나쁘다.
- 다른 의미를 위한 코드도 항상 함께하니 메모리도 많이 사용한다.
- 필드들을 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야한다.
- 새로운 의미를 추가하려면 모든 switch문을 찾아 새 의미를 처리하는 코드를 추가해야한다.
- 즉, 장황하고 오류를 내기 쉽고 비효율적이다.


반면 계층구조를 사용하면 위와 같은 문제점을 피할 수 있다.