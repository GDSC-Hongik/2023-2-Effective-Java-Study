# private 생성자나 열거 타입으로 싱글턴임을 보증하라.

## 싱글톤이란
싱글톤이란 객체를 하나만 만들 수 있는 클래스이다.
싱글톤은 주로 무상태 객체(즉 내부에서 저장하는 값이 없는)나 설계상 유일해야 하는 시스템 컴포넌트가 있다.

다만 클래스를 싱글톤으로 구현하면 테스트가 어려워 질 수 있다.
* 첫번째로 생성된 싱글턴 인스턴스를 mock 인스턴스로 대체할 수 없다.

### 싱글톤을 만드는 세가지 방식
---
#### 첫째, public static final을 이용한다.
인스턴스를 public static final로 선언과 동시에 초기화하여 사용하는 방식이다.
```java
public calass Elvis{
public static final Elvis INSTANCE = new Elvis();
private Elvis(){}
}

```
#### 장점
* 싱글턴임이 명백히 드러난다.

다만 이 방식은 프로그램을 실행하면 무조건 인스턴스를 생성하므로, 로직에 따라 사용하지 않는 클래스라면 메모리 낭비가 될 수 있다.
또한 클라이언트가 리플렉션 api를 사용하여 private 생성자를 호출할 수 있다.
```java
Constructor<ExampleClass> constructor = ExampleClass.class.getDeclaredConstructor();
constructor.setAccessible(true);
```
위와 같이 생성자를 임의로 public으로 변경하여 사용할 수 있기에 위험하다.


#### 둘째 
정적 팩터리 메서드를 사용한다.
```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	prictea Elvis() {};
	public static Elvis getInstance() { return INSTANCE};
}
```
장점
* 1.API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	prictea Elvis() {};
	public static Elvis getInstance() { return INSTANCE};
	public static Elvis getNewInstance() {return new Elvis();}
}
```
위와 같이 기존의 API를 변경하지 않았음에도 새로운 인스턴스를 만들 수 있다.

* 2.원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다는 것이다.
```java
public static void main(){
	Elvis.getInstance<Elvis>();
}

public class<T> Elvis {
	private static final Elvis INSTANCE = new Elvis();
	prictea Elvis() {};
	public static <R> R getInstance() { return INSTANCE};
	public T Elvis getNewInstance() {return new Elvis};
}
```
여기서 중요한 것은 제네릭은 static변수나 static 메서드에는 사용할 수 없다는 것이다.(static은 전처리에서 결정되는 데 타입을 알 수 없으므로)
이때 사용하는 것이 제네릭 메서드이다(단순 제네릭을 사용한 static 메서드하고는 다르다.)
위를 보면 getInstance() 앞에 \<R>을 붙인 것을 볼 수 있고, 이게 제네릭 메서드이다. 즉 제네릭 메서드는 메서드가 호출 될 때 타입이 결정되므로 문제가 되지 않는다. 
참고로 R이 아니라 T를 붙이더라도 클래스의 제네릭하고는 별개이다. 즉 같은 T가 아니다. 

* 3.정적 팩터리의 메서드 참조를 공급자(suplier)로 사용할 수 있다는 것이다.
```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	prictea Elvis() {};
	public static Elvis getInstance() { return INSTANCE};
}

public static void main(){
	Supplier<Elvis> elvisConstructor = Elvis::getInstance;
}

```
근데 이렇게 사용하였을 경우에 장점은?

```java
public static void main(){
	readInputFromView(Elvis::getInstance()); // 그럼생성자에서 예외가 터졌을 때 예외처리가 간결하게 가능하다.
}

private <T> T readInputFromView(Supplier<T> reader) {  
    try {  
        return reader.get();  
    } catch (IllegalArgumentException e) {  
        outputView.printErrorMessage(e);  
        return readInputFromView(reader);  
    }  
}
```
바로 위와 같이, 일급컬렉션같은 객체를 생성하려 할때 내부에 validator가 존재할 수 있는데, 이런 일급 컬렉션이 여러개 존재하고 매번 이를 외부에서 try catch하려면 코드가 복잡해질 수 있다. 위를 보면 Supplier를 이용하여 생성자에서 예외가 터지면 동일하게 catch를 해줌으로써 코드가 매우 간결해질 수 있다.

\# 참고로 여기서 단순히 T보다 T extends ...으로 타입을 제한해 주는 것이 좋다.


### 세번째 enum을 사용한다.
```java
public enum Elivs {
	INSTANCE;
	public void leaveTheBuliding(){}
} 
```
왜냐하면 enum은 기본적으로 싱글톤을 제공하며, 
추가로 직렬화, 리플렉션 공격에도 막아준다.

다만 이는 실질적으로 잘 사용되지는 않는다.
왜냐하면 싱글턴이 Enum외의 다른 클래스를 상속할 수 없어 확장성에서 떨어지기 때문이다.
## 위의 세가지를 제외한 다른 싱글턴 방법
### Lazy initialization (늦은 초기화 방식)
```java
public class Singleton {

    private static Singleton instance;

    private Singleton(){}

    public static Singleton getInstance(){
        if(instance == null){
            instance = new Singleton();
        }
        return instance;
    }
}
```
이 방식은 위의 첫번쨰, 두번째 싱글톤의 문제점을 해결해준다. 이름에서도 알 수 있듯이, 클래스 로더가, 클래스를 접근할 때 인스턴스를 생성하지 않고, 프로그램이 getInstance를 처음 호출할 때 생성되기 때문이다. 또한 그 이후에 호출하였을 경우에는 이미 존재하는 인스턴스를 리턴하므로 싱글톤임이 보장된다.

하지만 이 싱글톤에도 문제가 존재한다! 바로 멀티 스레딩 문제인데, 만약에 instance == null을 체크하여 true가 나온 상황에서 다른 스레드로 넘어가 싱글톤 객체를 생성한다면, 이 후 스레드로 돌아왔을 때 객체를 다시 생성하는 문제가 생긴다.

이를 해결하기 위해서는?
### LazyHolder에 의한 초기화 방법을 사용한 싱글턴
```java
public class Singleton {
  
  private Singleton() {}
  
  public static Singleton getInstance() {
    return LazyHolder.INSTANCE;
  }
  
  private static class LazyHolder {
    private static final Singleton INSTANCE = new Singleton();  
  }
}
```
위와 같이 싱글톤 클래스 안에 또 하나의 LazyHolder 클래스를 두는 방법이다.
특징적으로 
* Singleton클래스 안에 LazyHolder에 대한 변수가 존재하지 않으므로, Singleton클래스 로딩 시 LazyHolder를 초기화 하지 않는다.
* 또한 holder 안에 static으로 선언하여 클래스 로딩 시점에 한번만 호출되도록 한다.
* 또한 LazyHolder안에 인스턴스를 private로 선언하여 외부에서 접근 못하게 하며, 오직 이 클래스를 중첩한 Singleton 클래스에서만 접근할 수 있게한다.
* 또한 위에서와 같이 instance == null 같이 체크하는 부분이 없으므로 생성이 atomic하게 돌아가서 멀티스레딩 문제가 없다.


## 참고 mock 객체란?
모의 객체, 가짜 객체로 복잡한 논리가 들어가는 로직을 테스트할 때 주로 사용된다.
예를 들어서, 특정 인스턴스가 외부 api를 로직에서 사용한다면, 이는 test할 수가 없다. 이때 mockito를 사용한다. mockito를 이용하면 원하는 매개변수를 받아, 임의의 결과를 리턴하게 할 수 있다.

## 참고 메서드 참조란?
람다 표현식이 단 하나의 메소드만을 호출할 경우, 매개변수를 생략할 수 있도록 해준다.
```java
(base, exponent) -> Math.pow(base, exponent);

Math::pow;

//사용예제
public void myFunc() {  
    List<Integer> myList = List.of(1, 2, 3, 4, 5);  
    myList.stream()  
            .map(this::adder)  
            .collect(Collectors.toList());  
}  
  
public int adder(int input) {  
    return ++input;  
}

```

## 참고 제네릭이란?
데이터의 타입을 일반화한다는 것. 예를 들어 프로그램 실행 전까지는 어떤 타입일지 모르는 경우가 존재할 수있고, 또는 한 메서드지만 여러 타입을 리턴해야하는 경우도 있다. 이럴 때 사용된다.

사용예시
```java
public static void main(){
	int value1 = 1;
	String value2 = "이거야";

	myFunc(valuel);
	myFunc(value2);
}

public <T> T myFunc(T param){
	return param
} 

```