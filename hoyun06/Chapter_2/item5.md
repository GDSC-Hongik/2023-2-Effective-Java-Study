### Item5 : 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

> 각 클래스는 여러 자원에 의존한다.
```java
// Car 클래스는 내부적으로 Wheels와 Doors 클래스에 의존한다.
public class Car {
    private Wheels wheels;
    private Doors doors;
}
```

###### 외부 자원에 의존을 하는 클래스의 잘못된 설계
`정적 유틸리티 클래스를 잘못 사용한 예`
```java
public class SpellChecker {
    private static final Lexicon dictionary = new Lexicon();

    private SpellChecker() {...}

    public static boolean isValid(String word) {...}

    public static List<String> suggestions(String typo) {...}
}
```
`싱글턴 클래스를 잘못 사용한 예`
```java
public class SpellChecker {
    private final Lexicon dictionary = new Lexicon();

    private SpellChecker() {...}
    public static SpellChecker INSTANCE = new SpellChecker();

    public boolean isValid(String word) {...}

    public List<String> suggestions(String typo) {...}
}
```
- 위 두 경우 모두 하나의 자원에만 의존한다는 잘못된 가정을 세우고 설계한 클래스이므로 옳다고 보기 어렵다.
- 게다가 SpellChecker라는 클래스에서 외부 자원을 초기화하는 작업은 SRP에 어긋난다.
- 그때그때 dictionary 필드를 필요한 자원으로 갈아 끼우면 되지 않냐고 생각할 수 있지만 그러려면 final 키워드를 제거해야 하고 이 방식은
  오류에 취약할 뿐 아니라 멀티 쓰레드 환경에선 쓸 수 없다.

**즉, 여러 자원에 의존하면서 사용하는 자원의 종류에 따라 동작이 달라지는 클래스를 설계할 때에는 위 두 가지 방법은 옳지 않다**

###### 그럼 어떻게 해야 하나요?
해당 클래스에서 의존하는 여러 자원을 모두 지원할 수 있으면서도 클라이언트가 그때그때 자신이 원하는 자원을 사용할 수 있도록 해야 한다.

이것들을 제공할 수 있는 방법이 바로 `의존성 주입 패턴`이다.

###### 의존성 주입 패턴
특정 클래스가 의존하고 있는 인스턴스들에 대한 참조를 클래스 외부에서 주입해 주는 것을 의미한다.
```java
public class Computer {
    // Computer는 Keyboard interface에 의존
    private final Keyboard keyboard;
    
    // Computer는 Mouse interface에 의존
    private final Mouse mouse;
    
    // 생성자를 통해 외부로부터 Keyboard와 Mouse의 구현체를 넘겨받는다.
    public Computer(Keyboard keyboard, Mouse mouse) {
        this.keyBoard = keyBoard;
        this.mouse = mouse;
    }
}
```
```java
public class Main {
    public static void main(String[] args) {
        // 실제로 Computer 인스턴스를 생성할 때 키보드와 마우스에 대한 의존성을 주입한다.
        Computer computer = new Computer(new LogitechKeyboard(), 
                new LogitechMouse());
    }
}
```
- 각 필드의 final 키워드를 통해 불변성을 유지할 수 있으며 그에 따라 클라이언트는 어디서든 안심하고 위 객체를 사용할 수 있다.
- `의존성 주입 패턴`은 생성자, 정적 팩터리 메서드, 빌더 패턴 모두 적용 가능하다.

위의 예제에서는 생성자에 직접 의존 객체를 넘겨 주었다. 이 패턴을 조금 변형하면 객체 그자체를 넘기는 것이 아니라
객체 생성 팩터리를 넘기는 것도 가능하다.

###### 객체 생성 팩터리란?
디자인 패턴 중 `팩터리 메서드 패턴`을 구현한 것으로 호출될 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체다.
정말 단어 그대로 팩터리다.

- 팩터리의 대표적인 예시로 자바 8에서 추가된 Supplier< T >를 들 수 있다.
- Supplier< T >를 매개변수로 받는 메서드는 주로 `한정적 와일드카드 타입`을 이용하여 팩터리의 타입 매개변수를 제한하는데
- 클라이언트는 `한정적 와일드카드 타입`을 통해 자신의 원하는 타입 및 그 하위 타입을 생성하는 팩터리를 넘길 수 있다.

```java
import java.util.function.Supplier;

                                // 한정적 와일드 카드 타입 사용
public class KeyboardSupplierClass implements Supplier<? extends Keyboard> { 
    @Override
    public Keyboard get() {
        // 이곳에서 Keyboard를 구현한 클래스 중 
        // 원하는 클래스의 인스턴스를 반환 가능하다.
        return new LogitechKeyboard();
    }
}

public class MouseSupplierClass implements Supplier<? extends Mouse> {
    @Override
    public Mouse get() {
        // 이곳에서 Mouse를 구현한 클래스 중 
        // 원하는 클래스의 인스턴스를 반환 가능하다.
        return new LogitechMouse();
    }
}
```
```java
import java.util.function.Supplier;

public class Computer {
    private final Keyboard keyboard;
    private final Mouse mouse;
    
    public Computer(Supplier<? extends Keyboard> keyboard, 
                    Supplier<? extends Mouse> mouse) {
        this.keyboard = keyboard.get();
        this.mouse = mouse.get();
    }
}
```
```java
import java.util.function.Supplier;

public class Main {
    public static void main(String[] args) {
        
        // 팩터리 객체를 넘긴다.
        Computer computer = new Computer(new KeyboardSupplierClass()
                , new MouseSupplierClass());
    }
}
```
이 예제에서는 직접 Supplier를 구현했지만 람다식으로 대체 가능하다.

## 핵심 정리
클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 해당 클레스의 동작에 영향을 준다면
정적 유틸리티 클래스 혹은 싱글턴 방식을 사용하지 말자. 그대신 `의존 객체 주입 패턴`을 사용하자.