### Item18 : 상속보다는 컴포지션을 사용하라

###### 상속의 문제점
상속을 하면 서브 클래스는 슈퍼 클래스에 의존적이게 된다. 슈퍼 클래스의 구현에 따라 서브 클래스의 동작이 달라지기 때문이다.  
만약 다음 릴리즈에서 슈퍼 클래스 내부 구현이 바뀐다면 서브 클래스는 제대로 동작하지 않게 되어 그에 맞게 코드를 변경해야 할 수도 있다. 

예시)
```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet() {}

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```
- HashSet에 여태까지 몇 개의 원소가 추가됐는지를 유지하는 InstrumentedHashSet 클래스이다. HashSet을 상속했다.  
- 코드를 보면 add, addAll 메서드를 오버라이딩했고 각 메서드 내부에는 addCount를 증가시키는 로직을 포함하고 있다.
- 이 코드는 잘 동작할 것으로 보이지만 실제론 그렇지 않다. **size가 3인 리스트를 위 addAll 메서드를 통해 추가하면 우리는 addCount가 3일 것이라
기대하지만 실제론 6이 나온다.**

왜 6이 나올까? 
> super.addAll 을 호출하면 HashSet의 addAll이 호출되는데 이때 내부적으로 add가 호출되면서 오버라이딩한 add가 호출되기 때문이다.

우리는 addAll의 내부 구현에 대한 정보가 없으니 이런 문제가 발생하게 된다.  
'그렇다면 addAll만 재정의를 안하면 되는 것 아닌가?'라고 생각할 수도 있다. 어찌 보면 맞긴 하지만 이 역시나 'addAll은 내부적으로 add를 사용한다'라는 사실을
기반으로 코드를 작성하는 것이기 때문에 만약 addAll 메서드 내부 로직이 변경된다면 이전과 같은 문제가 발생한다.

**그럼 아예 상위 클래스의 메서드를 재정의하지 않고 모두 새로운 메서드만 하위 클래스에 작성하는 방법은 어떨까?**  
이는 재정의 방식보단 안전하겠지만 여전히 문제점들이 있다. 만약 서브 클래스에서 새로 작성한 메서드와 동일한 시그니처를 가지면서 다른 반환 타입을 가지는 
메서드가 다음 릴리즈에 슈퍼 클래스에 추가된다면 어떨까? 아니면 아예 리턴타입까지 같은 메서드라면? 보다시피 상속 자체에서 오는 문제가 있다.

###### 상속의 대체제
상속의 대체제로 `컴포지션`을 사용하자

`컴포지션`  
- 외부 클래스를 생성하여 상속하려고 했던 클래스를 private 필드로 참조하도록 한다. 해당 클래스 내부에는 
상속하려고 했던 클래스에 대응되는 메서드를 제공한다.
- 이 방식은 `전달(forwarding)`이라고 하며 이때 사용되는 메서드들을 `전달 메서드(forwarding method)`라고 한다.
```java
public class Car {
    private int position;
    
    public Car(int position) {
        this.position = position;
    }
    
    public void move() {
        this.position++;
    }
}

public class ColorCar {
    private Car car;
    private String color;
    
    public ColorCar(Car car, String color) {
        this.car = car;
        this.color = color;
    }
    
    // view 메서드
    public Car asCar() {
        return car;
    }
    
    public void move() {
        car.move();
    }
}
```
적당한 예시 클래스를 작성했다. ColorCar는 컴포지션 방식을 사용하여 내부적으로 Car 인스턴스를 참조한다.

메서드를 보면 ColorCar를 Car 형태로 사용할 수 있도록 view 메서드를 제공하고 move 메서드도 내부적으론 Car 클래스의 move를 호출하도록 구현했다.

###### 그렇다면 상속은 언제 사용하나요?
상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 사용해야 한다.  
즉, 클래스 B가 클래스 A와 `is-a 관계`일 때만 A를 상속해야 한다. 정말로 B가 A인지를 생각해 보고 만약 그렇지 않다면
상속이 아닌 컴포지션을 사용해야 한다.  
만약 정말로 상속을 꼭 해야 한다고 결정을 내렸으면 다음과 같은 질문을 마지막으로 해보자.  
- 확장하려고 하는 클래스의 api에 결함이 있는가?
- 결함이 있다면 내가 구현하려고 하는 서브 클래스가 해당 결함을 내포해도 괜찮은가?

왜냐하면 상속을 하면 슈퍼 클래스의 결함까지도 서브 클래스로 온전히 상속되기 때문이다.

## 핵심 정리
    상속은 상위/하위 클래스의 관계가 is-a 관계일 때만 사용하자. 하지만 이 경우에도 두 클래스의 패키지가 다르다거나 
    상위 클래스가 확장 가능성을 제대로 고려하지 않고 설계되었다면 문제가 생길 수 있다. 그렇기에 꼭 상속이 필요한 경우가 아니라면
    컴포지션을 사용하자


###### 데코레이터 패턴 사용하지 않는 상황
피자 관련 코드들을 작성한다고 생각해보자.  
초기에는 클라이언트가 불고기 피자, 치즈 피자, 페퍼로니 피자만 요구했기에 이 3개의 클래스만 작성하여 사용했다.
```java
public abstract class Pizza {
        
    void printName() {
        System.out.println("토마토 소스가 담긴 피자입니다.");
    }
}
public class BulgogiPizza extends Pizza {
    @Override
    public void printName() {
        System.out.print("불고기, ");
        super.printName();
    }
}
public class CheesePizza extends Pizza {
    @Override
    public void printName() {
        System.out.print("치즈, ");
        super.printName();
    }
}
public class PepperoniPizza extends Pizza {
    @Override
    public void printName() {
        System.out.print("페퍼로니, ");
        super.printName();
    }
}
```
하지만 시간이 흐르고 클라이언트의 요구 사항이 늘어나서 새로운 피자 종류들을 요구하며
기존 피자들 중에서도 재료를 섞은 피자를 요구한다고 하자.(ex. 페퍼로니 치즈 피자, 불고기 치즈 피자...)

위 경우에는 새로운 종류의 피자에 대해선 당연히 새로운 클래스를 작성해야 한다. 심지어 기존 재료들을 그저 같이 토핑한 피자에 대해서도
새로운 클래스를 생성해서 사용해야 한다.(ex. PepperoniCheesePizza, BulgogiCheesePizza).

여기서 유용하게 사용할 수 있는 것이 `데코레이터 패턴`이다.

###### 데코레이터 패턴
> 주어진 상황에 따라 특정 객체에 기능을 덧붙이는 패턴으로, 기능 확장이 필요할 때 상속을 대신할 수 있는 유연한 대안이 될 수 있다.

쉽게 말해 원본 객체에 기능들을 덧붙이는 패턴이다. 이 패턴에서는 컴포지션을 이용하여 로직의 실행을 위임한다.

###### 데코레이터 패턴 구조

<img src="https://cdn.stackoverflow.co/images/jo7n4k8s/production/9edc178caf76a8166ed915e808b8865f4ab6f226-1024x709.png?auto=format"/>
출처: https://stackoverflow.blog/2021/10/13/why-solve-a-problem-twice-design-patterns-let-you-apply-existing-solutions-to-your-code/

- **Component** : 원본 객체와 Decorator 객체들을 하나의 타입으로 묶어주는 역할을 한다. interface를 사용하여 원본 객체 및 Decorator 객체들이 구현해야 할 기능에 대해 정의한다.
- **ConcreteComponent** : 원본 객체를 나타낸다. 데코레이팅의 대상이 되는 객체.
- **Decorator** : 추가할 기능들을 나타내는 Decorator 클래스를 추상화 해놓은 객체. abstract 클래스로 보통 선언하며 원본 객체를 컴포지션 형태로 참조한다.
- **ConcreteDecorator** : 실제 추가할 기능들을 각각 담당하는 구체 Decorator 클래스들.

**데코레이터 패턴을 실제 코드로 살펴보려고 한다.**

###### Component
```java
public interface Pizza {
    void printName();
}
```
원본 객체와 구체 Decorator 객체들이 구현해야 할 기능을 정의하는 역할을 한다. 여기선 각 Pizza에 들어간 재료를 출력하는 메서드를 구현해야 한다.

###### ConcreteComponent
```java
public class DefaultPizza implements Pizza {
    @Override
    public void printName() {
        System.out.println("토마토 소스가 담긴 피자입니다.");
    }
}
```
Pizza 인터페이스를 구현한다. 데코레이팅을 받을 원본 객체를 나타내는 클래스다. 여기선 아무 토핑이 올라가지 않은 토마토 소스만 바른 피자가 원본이다.  
이제 이 원본 객체에 하나씩 데코레이팅을 할 수 있다.

###### Decorator
```java
public abstract class PizzaDecorator implements Pizza {

    // 컴포지션
    private Pizza pizza;
    
    public PizzaDecorator(Pizza pizza) {
        this.pizza = pizza;
    }
    
    @Override
    public void printName() {
        pizza.printName();
    }
}
```
Decorator 클래스는 인스턴스화를 하는 것이 목적이 아니므로 추상 클래스로 선언한다. 데코레이팅을 적용할 원본 객체 혹은 
이미 데코레이팅이 적용된 원본 객체에 또다른 데코레이팅을 적용할 수 있도록 설계하는 것이 목표이다. 그래서   
원본 객체와 Decorator 클래스들을 모두 아우르는 **Pizza 인터페이스 타입**을 컴포지션 형식으로 참조한다.  
printName 메서드 내부 구현을 봐도 자신이 내부적으로 참조하고 있는 원본 객체에게 로직 실행을 위임하는 것을 알 수 있다.

###### ConcreteDecorator
```java
public class PepperoniDecorator extends PizzaDecorator {
    public PepperoniDecorator(Pizza pizza) {
        super(pizza);
    }
    
    @Override
    public void printName() {
        System.out.print("페퍼로니, ");
        super.printName();
    }
}
```
```java
public class MushroomDecorator extends PizzaDecorator {
    public MushroomDecorator(Pizza pizza) {
        super(pizza);
    }
    
    @Override
    public void printName() {
        System.out.print("버섯, ");
        super.printName();
    }
}
```
데코레이팅을 적용할 원본 객체는 super() 를 통해 참조를 유지하고 printName 메서드에서는 각 Decorator 클래스들이 제공하고자 하는 데코레이팅 기능을 
구현한다. 

실제 클라이언트 코드는 다음과 같을 수 있다.
```java
public class Main {

    public static void main(String[] args) {
        boolean pepperoniEnabled = true;
        boolean mushroomEnabled = true;
    
        Pizza pizza = new DefaultPizza();
        pizza.printName();
    
        if (pepperoniEnabled) {
            pizza = new PepperoniDecorator(pizza);
        }
        if (mushroomEnabled) {
            pizza = new MushroomDecorator(pizza);
        }
    
        pizza.printName();
    }
}
```
출력 결과
```java
// 원본 피자 객체
토마토 소스가 담긴 피자입니다.

// 데코레이터에 의해 기능이 추가된 피자 객체
버섯, 페퍼로니, 토마토 소스가 담긴 피자입니다.
```
위 코드와 같이 런타임에 원본 객체에 필요한 기능을 추가할 수 있다.  
DeafaultPizza의 생성자를 통해 원본 객체를 생성하고 그 뒤에 필요한 토핑을 데코레이팅 한다.  
물론 이 경우에도 새로운 피자 종류가 필요하면 새로운 Decorator 클래스를 작성할 수밖에 없다.  
하지만 기존 재료들을 활용한 피자를 만들고자 한다면 Decorator를 다중 적용하여 생성이 가능하다.
```java
public class Main {
    
    public static void main(String[] args) {
        Pizza pizza = new PepperoniDecorator(new MushroomDecorator(new DefaultPizza()));
    }
}
```

###### 데코레이터 패턴 장/단점
**장점**
- 확장이 필요한 경우에 서브 클래스를 만들지 않고도 기존 Decorator 클래스를 재활용할 수 있다.
- 컴파일 타임이 아닌 런타임에 동적으로 기능을 변경할 수 있다.
- 각 Decorator 클래스마다 자신의 데코레이팅 기능만 맡으므로 SRP를 준수한다.

**단점**
- 데코레이팅을 적용한 뒤에 나중에 그 데코레이팅된 기능을 제거하는 것이 까다롭다.
- 여러 Decorator가 적용된 인스턴스 생성 시 생성자가 중복되어 작성되므로 가독성이 떨어진다.
- 데코레이팅을 적용하는 순서에 따라 로직 실행 순서가 바뀔 수 있다.