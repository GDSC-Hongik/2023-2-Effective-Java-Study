# Item 40. @Override 애너테이션을 일관되게 사용하라

@Override는 메서드 선언에만 사용 가능
- 사용했다 == 상위 타입의 메서드를 재정의했다

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    // ...
}
```

위의 코드에서는 equals가 제 역할을 하지 못함  
=> 재정의임에도 @Override를 사용하지 않아, 
재정의가 아닌 다중 정의가 되어버림.  
상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override를 달자.