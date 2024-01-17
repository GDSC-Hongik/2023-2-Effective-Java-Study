### Item36 : 비트 필드 대신 EnumSet을 사용하라

예전부터 열거 타입 상수를 단독이 아닌 집합으로 사용하는 경우 비트 필드를 애용했다.
```java
public class Text {
    public static final int STYLE_BOLD      = 1 << 0; // 1
    public static final int STYLE_ITALIC    = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
    
    public void applyStyles(int styles) { ... }
```
```java
public static void main(String[] args) {
    text.applyStyles(Text.STYLE_BOLD | STYLE_ITALIC);
}
```
이와 같이 사용하면 비트별 연산을 이용하여 합집합, 교집합과 같은 연산을 수행할 수 있다.  
하지만 이와 같은 방식은 한계가 명확하다
- or 연산을 하는 경우 한 비트 필드에 여러 상수가 적용된 것일 수 있는데 이때 적용된 모든 상수를 순회하는 것이 까다롭다
- 비트 값이 외부에 그대로 노출되면 일반 정수 열거 패턴보다 오히려 더 가독성이 떨어진다
- 상수를 표현하는데 최대로 필요한 비트 수를 미리 결정하여 타입을 결정해야 한다

###### EnumSet
비교적 불안정한 비트 필드 대신 `EnumSet`을 사용하자.  
EnumSet은 열거 타입의 상수로 이루어진 집합을 효율적으로 나타낼 수 있다. Set 인터페이스를 구현하며 타입 안전하다.
그리고 내부적으로는 비트 벡터로 구현되어 있어 열거 타입의 상수 개수가 64개 이하라면 long 변수 하나로 모든 상수를 표현할 수 있다.