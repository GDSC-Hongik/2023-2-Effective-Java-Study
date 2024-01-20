# Item 34. int 상수 대신 열거 타입을 사용하라

## 정수 열거 패턴의 단점
```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;
```

- 타입 안전을 보장하지 못한다.
- 문자열로 출력하기 까다롭다.

### 그렇다면 문자열 상수 패턴은?
오타 있어도 컴파일러가 확인할 수 없어서 런타임에 발견  
=> 더 나쁨.

## Enum
- 밖에서 접근할 수 있는 생성자를 제공하지 않음 => 사실상 final
- 싱글턴을 일반화한 형태
- 타입 안전성을 제공
- toString 메서드는 출력하기 적합한 문자열을 반환

```java
public enum Planet {
	MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6);
	// ...
	
	private final double mass;
	private final double radius;
    private final double surfaceGravity;
    
    private static final double G = 6.67300E-11;
    
    Planet(double mass, double radius) {
    	this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }
    
    public double mass() { return mass; }
    public double radius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }
    
    public double surfaceWeight(double mass) {
    	return mass * surfaceGravity;
    }
}
```
열거 타입 상수 각각을 특정 데이터와 연결하려면, 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 됨

## 값에 따라 다른 동작
추상 메서드를 이용하면 됨
```java
public enum Operation {
    PLUS { double apply(double x, double y) { return x + y; } },
    MINUS { double apply(double x, double y) { return x - y; } },
    TIMES { double apply(double x, double y) { return x * y; } },
    DIVIDE { double apply(double x, double y) { return x / y; } };
    
    abstract double apply(double x, double y);
}
```
- 추상 메서드를 재정의하지 않으면 컴파일러가 알려줌