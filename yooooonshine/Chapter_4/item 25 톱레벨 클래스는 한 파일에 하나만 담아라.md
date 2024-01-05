일단 톱레벨 클래스가 뭘까?
## 톱레벨 클래스
톱레벨 클래스란 중첩 클래스가 아닌 클래스를 의미한다.
즉 우리가 사용하는 일반 적인 클래스를 의미한다.

이와 반대로 중첩 클래스란?
## 중첩 클래스
다른 클래스의 내부에 선언된 클래스를 의미한다.

# 톱레벨 클래스는 한 파일에 하나만 담아라
---
소스 파일 하나에 여러 톱레벨 클래스를 선언하여도  문제되지는 않는다.
하지만 이는 좋지 못한 행위이다.
왜냐하면 톱레벨 클래스를 중복 정의하게되면서 오류를 불러올 수 있기 때문이다.
## 톱레밸  클래스의 중복 정의
다음과 같은 예시를 보자
```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```
main클래스에서 `Utensil`클래스와 `Dessert`클래스를 사용하고 있다.

그럼 `Utensil`소스 파일에 다음과 같이 선언한다면
```java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```
위의 `Main`을 실행하면 pencake가 출력된다.

다시 `Dessert` 소스 파일을 똑같은 두 클래스를 담아 만들어보자
```java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

이렇게 똑같은 클래스를 중복 정의했을 경우 컴파일러의 동작이 달라질 수있다.

컴파일러는 우선 `Main`을 컴파일하면서 순서대로 읽다가 `Utensil`을 컴파일 할 것이다. 그리고 `Dessert`파일을 컴파일하면서 같은 이름의 클래스가 있음을 알게 된다. 이건 Main을 먼저 읽었을 경우이며 반대로 다른 거 먼저 읽으면 동작이 달라질 수 있다.
```java
javac Main.java Dessert.java // 컴파일 오류 발생
javac Main.java // pancake 출력
javac Main.java Utensil.java // pancake 출력
javac Dessert.java Main.java // potpie 출력
```
이처럼 컴파일러에 어떤 소스코드를 먼저 건네냐에 따라 결과가 달라질 수 있으므로 
톱레벨 클래스들을 서로 다른 소스파일로 분리하여 문제를 해결해야 한다.