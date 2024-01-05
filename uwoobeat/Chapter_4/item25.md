# Item 25. 톱레벨 클래스는 한 파일에 하나만 담으라

소스 파일 하나에 톱레벨 클래스가 여러 개 있어도 컴파일이 되도록 만들 수는 있지만, 위험하다.

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

```java
// Utensil.java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```

```java
// Dessert.java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

`javac Main.java` 를 실행하면 Main에서 Utensil 클래스 참조를 따라 Utensil.java를 참조하고 여기서 Utensil.NAME과 Dessert.NAME을 모두 찾아낸다.

`javac Main.java Dessert.java` 를 실행하면 Main에서 Utensil 클래스 참조를 따라 Utensil.java를 참조하고 여기서 Utensil.NAME과 Dessert.NAME을 모두 찾아낸다. 그리고 Dessert.java를 참조할 때 중복 정의된 Utensil 클래스를 발견하고 컴파일 에러를 발생시킨다.

`javac Main.java Utensil.java` 를 실행하면 Main에서 Utensil 클래스 참조를 따라 Utensil.java를 참조하고 여기서 Utensil.NAME과 Dessert.NAME을 모두 찾아낸다. 그리고 Utensil.java를 참조할 때 이미 아까 전에 찾아낸 것이므로 아무런 문제도 발생하지 않는다.

즉 탑레벨 클래스를 여러 개 정의하면 어떤 소스를 먼저 컴파일하느냐에 따라 동작이 달라진다. 따라서 되도록 분리하도록 하자.