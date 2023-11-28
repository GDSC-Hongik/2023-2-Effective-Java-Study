# Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

어떤 클래스가 복수의 자원을 사용할 수 있다고 하자.  
<br>
```java
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = dictionary;
    }
}
```
SpellChecker 클래스가 여러 종류의 dictionary를 사용할 수 있는 상황이다.  
이런 경우 사용하는 자원에 따라 동작이 달라지므로 정적 유틸리티 클래스나 싱글턴 방식은 적합하지 않다.  

생성자를 통해 사용할 자원을 매개변수로 넘겨주는 방식을 사용하게 되는데, 이를 의존 객체 주입 패턴이라고 부른다.  
정리하자면, 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스의 동작에 영향을 준다면 이 자원들을 클래스가 직접 만들도록 해서는 안 되므로 필요한 자원을 생성자에게 넘겨줘서 의존 객체를 주입하도록 할 수 있다.  
이 방법을 사용하면 코드의 유연성과 재사용성을 높일 수 있다.