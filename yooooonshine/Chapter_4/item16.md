# public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
---
우선적으로 이 클래스를 보자
```java
class Point {
	public double x;
	public double y;
}
```

이 클래스에서 x, y는 데이터 필드에 직접 접근이 가능하고, 불변식을 보장할 수 없으며, 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.

다음 클래스를 보자
```java
class Point {
	private double x;
	private double y;
	public Point(double x, double y) {
		this.x = x;
		this.y = y;
	}
	public double getX() {return x;}
	public double getY() {return y;}
	public double setX() {this.x = x;}
	public double setY() {this.y = y;}
}
```
이 방식은 직접 x,y에 접근하는 것이 아니며, setter와 getter를 통해 접근하고 class를 package-private  혹은  private 중첩 클래스라면 외부에서 접근할 수 없으므로 필드를 노출한다해도 하등 문제가 없다.

즉 public 클래스는 절대 가변 필드를 직접 노출하면 안되고
불변 필드라면 노출해도 덜 위험한 편이며
package-private클래스나 private 중첩 클래스에서는 노출해도 무관하다.