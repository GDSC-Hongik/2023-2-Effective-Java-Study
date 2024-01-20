# 인스턴스화를 막으려거든 private 생성자를 사용하라.
종종 static메서드, static클래스를 만들 필요가 있고, 이럴 때 꼭 생성자를 private처리해주자.

다만 객체지향적 관점에서 static은 자제해야 한다. 왜냐하면 모든 변수 메서드를 static처리한다면, 캡슐화 관점에서 외부에서 데이터를 접근할 수 없게 해야하는 데 이와 상반되기 때문이다.

그럼에도 불구하고 static이 더 좋을 때가 있다.

1.java.lang.Math와 java.util.Arrays처럼 기본 타입 값이나 배열 관련 메서드들을 모아놓을 수 있다.
```java
public static final double E = 2.718281828459045;
public static final double PI = 3.141592653589793;
public static double sin(double a) {  
    return StrictMath.sin(a); // default impl. delegates to StrictMath  
}

```
위 코드는 Math 클래스 안에 존재하는 변수와 메서드를 가져온 것이다. 위에서 PI 같은 유일 값을 위하여 static 처리를 하였고, 메서드 내부적으로 static 변수를 사용해야 하므로 메서드 또한 static임을 볼 수있다.

2. 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아놓을 수 있다.
여기서 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드란?
```java
public interface MyInterface { 
	void myMethod(); 
} 
public class MyImplementations {
	public static MyInterface createImplementation1() { 
	return new MyInterface() { 
		@Override public void myMethod() { 
			System.out.println("Implementation 1"); 
			} 
		}; 
}
```
위를 보면 MyInterface를 createImplementatoin1에서 상속하여 객체를 만들고 있다. 이때 MyInterface는 myMethod를 구현해야하는데, 이를 MyInterface를 Override해주면서 생성해주고 있다.
즉 여기서 createImplementation1이 특정 인터페이스를 구현하는 객체를  생성해주는 정적 메서드인데, 이는 static메서드이므로, 이들을 모아놓는 static 클래스를 둬서 사용할 수 있다는 것이다.

3.final 클래스와 관련한 메서드들을 모아 놓을 때도 사용한다.
final 클래스를 상속해서 하위 클래스에 메서드를 넣는 것은 불가능하기 때문이다.
(final 키워드가 적용된 클래스를 만들면 이 클래스는 상속이 불가능하다.)


참고로 추상 클래스를 만들어 생성자를 못 만들게 할 수 있지 않나라는 의문이 들 수 있는데, 이는 옳지 못하다, 상속받아서 인스턴스화 하면 그만이기 때문이다.

생성자가 private니 바깥에서 접근할 수 없지만, 혹시라도 클래스 안에서 생성자를 호출 할 수 있으니 생성자 내부에 예외를 터트리는 로직을 둬도 좋다.

참고로 private 생성자는 상속을 불가능하게 하는 것도 있다. 상속받은 클래스는 super()를 통해서 상위 클래스의 생성자를 호출해야 하는데 , private이므로 호출할 수 없다.