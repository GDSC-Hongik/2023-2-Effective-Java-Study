# Item 52. 다중정의는 신중히 사용하라

재정의한 메서드 -> 동적으로 선택  
다중정의한 메서드 -> 정적으로 선택

```java
class Fruit {
  String name() {
    return "과일";
  }
}

class Apple extends Fruit {
  @Override
  String name() {
    return "사과";
  }
}

public class Main {
  public static void main(String[] args) {
    Fruit[] fruits = {
            new Fruit(), new Apple()
    };
    for (Fruit fruit : fruits) {
      System.out.println(fruit.name());
    }
  }
}
```
가장 하위에서 정의한 재정의 메서드가 선택됨  

### 다중정의된 메서드 간 객체의 타입은 중요하지 않다
선택은 컴파일 타임에 이뤄지기 때문

### 다중 정의는 혼동을 일으킴
- 매개변수 수가 같은 다중정의는 지양하자
- 생성자라면 어쩔 수 없다
    - 매개변수의 타입이라도 다르게 하자

### 정리
다중정의는 혼란을 일으킬 수 있으니 활용에 주의하자
