### Item25 : 톱레벨 클래스는 한 파일에 하나만 담으라

자바에서는 하나의 java 파일에 여러 개의 톱레벨 클래스를 선언할 수 있다.
```java
public class Main { // Main.java
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```
```java
class Utensil { // Utensil.java
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```
```java
class Utensil { // Dessert.java
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```
컴파일러는 이에 대해 불평하지 않지만 어느 파일을 먼저 컴파일하느냐에 따라서 동작이 달라진다.  
- **javac Main.java Dessert.java를 하는 경우**
  - 컴파일 에러가 난다. 
  - 컴파일러는 Main.java를 컴파일하면서 Utensil 클래스의 참조를 확인하고 Utensil.java를 확인한다. 
  그 과정에서 Utensil, Dessert 클래스에 대해 인지한다. 그 다음으로 Main에서 나오는 Dessert 클래스 참조를 확인하고 Dessert.java를 확인하는데 이 과정에서 클래스가 중복 정의되었음을 알고
  컴파일 에러를 발생시킨다.
- **javac Main.java 혹은 javac Main.java Utensil.java를 하는 경우**
  - 마치 Dessert.java를 작성하지 않은 것처럼 "pancake"가 정상적으로 출력된다.
- **javac Dessert.java Main.java를 하는 경우**
  - "potpie"를 출력한다.

###### 해결책
톱레벨 클래스는 파일을 분리하자. 정 하나의 파일에 여러 개의 클래스를 담아야 한다면 이전 item에서 설명한
멤버 클래스를 고려하자.