### Item16 : public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

###### 안 좋은 예시
```java
public class point {
    public double x;
    public double y;
}
```
public 클래스의 내부 필드를 public으로 노출하면...
- api를 수정하지 않고는 내부 구현을 바꿀 수 없다.(public 멤버는 공개 api에 노출되므로)
- 외부에서 필드 접근 시 부가 작업이 불가하다.(getter 처럼 메서드를 통해 값을 가져오는 것이 아니라 필드 자체를 사용하므로)

###### 좋은 예시
```java
public class point {
    private double x;
    private double y;
    
    public double getX() {return x;}
    public double getY() {return y;}
    public void setX(double x) {this.x = x;}
    public void setY(double y) {this.y = y;}
}
```
멤버를 public으로 두지 말자.  
그대신 private으로 선언하고 해당 멤버에 접근할 수 있는 접근자 메서드를 public으로 제공하자.

이렇게 하면 막말로 x,y를 package-private으로 수정하는 것도 api에 영향받지 않고 가능하다.

하지만 public이 아닌 package-private 클래스나 아님 private inner class 같은 경우는 public 멤버 선언을 고려할만 하다.  
이 두 클래스는 외부로 공개되는 클래스도 아니거니와 같은 패키지 혹은 자신을 담고있는 외부 클래스에서만 접근 가능하기 때문에 (getter/setter)보다 
훨씬 나은 경우도 있다.

## 핵심 정리
    public 클래스는 반드시 private 멤버와 그를 위한 접근자 메서드를 활용하자.
    package-private 혹은 private inner class 의 경우에는 public 멤버를 사용하는 것도 좋다.