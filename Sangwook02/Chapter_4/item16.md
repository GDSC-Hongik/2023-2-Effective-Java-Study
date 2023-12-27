# Item 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

인스턴스의 필드들을 public으로 지정하고 직접 접근하도록 하는 것은 캡슐화의 이점을 제공하지 못한다.  
```java
class Point {
    public double x;
    public double y;
}
```
이런 경우, 인스턴스의 필드들을 private으로 바꾸고, getter를 통해 접근하도록 할 수 있다.  
외부에서 접근하는 일을 막아야 한다면 getter를 구현하지 않음으로써 보호할 수 있다.  
```java
class Point {
    private double x;
    private double y;
    
    public double getX() {
		return x;
    }

	public double getY() {
		return y;
	}
}
```

### 정리
public 클래스는 가변 필드를 public으로 노출해서는 안 된다.  
