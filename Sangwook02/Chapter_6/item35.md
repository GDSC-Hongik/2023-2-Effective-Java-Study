# Item 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

```java
public enum Ensemble {
    SOLO, DUET, TRIO;

    public int numberOfMusicians() {
        return ordinal() + 1;
    }
}
```
위의 Enum 클래스에서는 ordinal() 메서드를 사용해 순서를 반환한다.  
스프링 공부를 하면서 Enum의 상수는 가능한 사용하지 않는 것이 좋다고 배웠다.  
중간에 있는 DUET이 사라진다면, TRIO의 ordinal() 메서드가 반환하는 값도 바뀌기 때문이다.  

인스턴스 필드를 이용해 아래처럼 사용하자.
```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3);

    private final int numberOfMusicians;

    Ensemble(int size) {
        this.numberOfMusicians = size;
    }

    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}
```