# Item 1. 생성자보다 정적 팩토리 메서드

객체지향 프로그래밍 수업 시간에 배우게 되는 생성자 외에도 알아둬야 할 방식이 있다.  
바로 정적 팩토리 메서드이다.  
무작정 정적 팩토리 메서드를 쓰는 것보다 장단점을 알고 사용하는 편이 좋다.  
우선 장점부터 살펴보자.

### 이름을 가질 수 있다.
생성자는 매개변수의 개수나 종류와 무관하게 그 이름은 클래스의 이름이 결정되는 순간에 하나로 정해져 버린다.  
하지만, 정적 팩토리 메서드를 사용하면 그 종류나 개수에 따라 메서드명을 달리 지어줄 수 있으므로 목적과 의미 파악에 유리하다.

### 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
String, Boolean, Integer와 같은 불변 클래스는 불필요한 객체 생성 없이 재활용이 가능하다.  
인스턴스의 생명주기를 통제하여 싱글톤 클래스로 만들 수도 있다.

### 반환 타입의 하위 타입 객체를 반환할 수 있고 매개변수에 따라 다른 클래스를 반환할 수도 있다.
Cafe라는 interface가 있고, 이를 implement 하는 Starbucks와 TwosomePlace라는 class가 있다고 하자.
```java
public interface Cafe {
    static Cafe of(String cafeName) {
        if (cafeName == "starbucks") {
            return new Starbucks();
        }
        else if (cafeName == "twosome place") {
            return new TwosomePlace();
        }
        else {
            return new UnknownCafe();
        }
    }

    public String getCafeName();
}
```

```java
public class Starbucks implements Cafe{
    private final String cafeName = "스타벅스";

    @Override
    public String getCafeName() {
        return cafeName;
    }
}
```
Cafe의 정적 메서드 of를 호출하여 Cafe의 하위 타입인 Starbucks를 아래와 같이 반환받을 수 있다.  
Starbucks class를 직접 부르지 않았지만, 매개변수에 따라 다른 하위 클래스가 반환된 것이다.
```java
public class main {
    public static void main(String[] args) {
        Cafe sb = Cafe.of("starbucks");
        System.out.println("sb.getCafeName() = " + sb.getCafeName());

        Cafe tp = Cafe.of("twosome place");
        System.out.println("tp.getCafeName() = " + tp.getCafeName());
    }
}
```
public이나 protected 생성자가 있어야 한다는 단점도 있지만, 장점에 비해 임팩트가 약하다.  
정적 팩토리 메서드 만으로는 하위 클래스를 만들 수 없기 때문이다.

정적 팩토리 메서드를 사용했을 때, public 생성자를 사용하는 것에 비해 유리한 경우가 더 많으므로 적절한 이름으로 활용하면 좋다.  
아래는 흔히 쓰이는 이름들이다.
- from
- of
- valueOf
- create
- new{클래스 이름}