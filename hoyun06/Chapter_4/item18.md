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