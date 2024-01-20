### Item10 : equals는 일반 규약을 지켜 재정의하라

###### Object 클래스에서 제공하는 메서드들
모든 클래스의 슈퍼 클래스인 **Object**는 하위 클래스들이 재정의하여 사용할 수 있는 메서드를 여러 개 보유하고 있다.
- toString
- hashCode
- equals
- finalize
- clone

이 메서드들은 서브 클래스에서 재정의할 시 지켜야 하는 일반 규약들이 정해져 있다.  
만약 규약에 맞지 않게 재정의하면 HashMap 혹은 HashSet과 같이 대상 클래스가 해당 규약을
잘 지켰다고 가정하고 동작하는 클래스에서 문제가 발생하게 된다.

이번 item에서는 그중에서도 equals 메서드에 대해 알아볼 것이다.

###### equals 메서드를 재정의할 필요가 없는 경우
- **각 인스턴스가 본질적으로 고유한 경우**
  - 값을 표현하는 클래스(ex. Integer, Double, String ..)가 아니라 그냥 동작하는 하나의 개체를 표현한 클래스(ex. Thread)의 경우
  굳이 equals 메서드를 재정의할 필요가 없다.
- **인스턴스의 `논리적 동치성(logical equality)`을 검사할 일이 없는 경우**
  - 예를 들어, 2차원 평면 상의 좌표를 나타내는 Point 클래스는 두 점의 좌표가 같은지 비교하기 위해 x, y 좌표 값을 비교해야 한다.
  그래서 equals 메서드를 용도에 맞게 재정의하는 것이 좋다.  
  하지만 이런 논리적 동치를 검사할 필요가 없다면 equals 메서드를 재정의할 필요가 없다.
- **상위 클래스에서 재정의한 equals 메서드가 하위 클래스에서도 딱 맞는 경우**
- **굳이 equals 메서드를 호출할 필요가 없는 경우**

###### equals 메서드를 재정의해야 하는 경우
- **인스턴스 간의 물리적 비교(참조값 비교)가 아닌 논리적 비교가 필요한 경우**
   - 이때, 상위 클래스에 논리적 비교를 하는 equals 메서드가 정의되지 않았다면
   equals 메서드를 재정의해야 한다.


- **주로 값을 나타내는 클래스들이 해당된다.(ex. Integer, String ...)**


- **이렇게 논리적 동치를 확인하는 equals 메서드를 재정의하면..**
  - 해당 클래스의 인스턴스를 Map의 key로 사용할 수도 있고  
  (Map의 get 메서드에서는 내부적으로 equals 메서드를 이용하여 key값을 비교하므로)
  - 해당 클래스의 인스턴스를 Set의 원소로 사용할 수도 있다.  
  (Set에 원소를 넣을 때는 기본적으로 equals 메서드를 이용하여 중복 객체를 파악하므로)

###### 그럼 값을 나타태는 클래스는 equals 메서드를 항상 재정의하나요?
꼭 그렇지만은 않다.

값 클래스라고 해도 싱글턴 패턴을 이용하는 인스턴스 통제 클래스이거나 Enum과 같이 값이 같은 인스턴스가
무조건 하나만 존재한다는 보장이 있다면 굳이 equals를 재정의할 필요가 없다.

###### Object 클래스의 명세에 적힌 규약
equals 메서드를 재정의할 때는 일반 규약을 준수해야 하는데 다음은 Object 명세에 적힌 규약이다.
- **반사성(reflexivity)**  
`null`이 아닌 모든 참조 값 `x`에 대해, `x.equals(x)`는 `true`다.
- **대칭성(symmetry)**  
`null`이 아닌 모든 참조 값 `x, y`에 대해, `x.equals(y)`가 `true`면 `y.equals(x)`도 `true`다.
- **추이성(transitivity)**  
  `null`이 아닌 모든 참조 값 `x, y, z`에 대해, `x.equals(y)`가 `true`이고, `y.equals(z)`도 `true`면 `x.equals(z)`도 `true`다.
- **일관성(consistency)**  
  `null`이 아닌 모든 참조 값 `x, y`에 대해, `x.equals(y)`를 반복해서 호출하면 항상 `true`이거나 `false`다.`
- **null-아님**  
  `null`이 아닌 모든 참조 값 `x`에 대해, `x.equals(null)`은 `false`다.

위 규약은 언뜻 보면 굉장히 복잡해 보인다. 하지만 잘 읽어보면 당연한 말들이고 만약 본인이 이산 수학을 배웠다면
Relation에서 배운 내용을 떠올리다면 비교적 이해가 쉬울 것 같다.  
위 규약들은 equals 재정의 시 반드시 지켜야 한다. 그렇지 않으면 프로그램에서 예상치 못한 오류가 발생하고 그 오류의 원인을 파악하는 것도 결코 쉽지가 않기 때문이다.

지금부터 각 규약에 대해 조금 더 자세히 알아보려고 한다.

###### 반사성
단순히 말하면 객체는 자기 자신과 같아야 한다는 것을 뜻한다. 이건 사실 너무 당연한 규칙이다

###### 대칭성
두 객체는 서로에 대한 동치 여부에 같은 답을 해야 한다는 것을 뜻한다. 이건 코드를 보면 이해가 더 쉽다.  
다음 코드를 보면 대칭성이 무너져있다.
```java
public final class CaseInsensitiveString{
  private final String s;

  public CaseInsensitiveString(String s){
    this.s = Obejcts.requireNonNull(s);
  }

  @Override public boolean equals(Object o){
    if(o instanceof CaseInsensitiveString)
      return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
    if(o instanceof String) // 한방향으로만 작동한다.
      return s.equalsIgnoreCase((String) o);
    return false;
  }
}
```
대소문자 구분하지 않는 문자열을 저장하는 CaseInsensitiveString은 equals 메서드를 재정의했다.  
이때 매개변수를 Object로 받아서 CaseInsensitiveString간의 비교도 가능하고 일반 String과의 비교도 가능하게 재정의했다.

그래서 다음과 같은 코드를 작성하면
```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";

cis.equals(s); // true
```
정상적으로 비교할 수 있고 true가 반환된다. 

하지만 여기서 문제점은 이 비교가 일방적이라는 것이다. CaseInsensitiveString은 재정의한 equals 덕분에 일반 String과 비교할 수 있다.

하지만 String 객체는 CaseInsensitiveString에 대한 정보가 없기 때문에 다음과 같이 코드를 작성하면 false를 반환한다.
```java
s.equals(cis); // false
```
이는 대칭성 위반이다. 그렇기 때문에 equals 메서드를 다음과 같이 수정하여 대칭성을 달성해야 한다.
```java
@Override public boolean equals(Object o){
  return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s); 
}
```
String 객체와 비교하는 내용을 제거함으로써 일방적인 비교 로직을 삭제했다.

###### 추이성
쉽게 말해
> if A = B and B = C then A = C 를 뜻한다.

이 규약은 '구체 클래스'를 상속하는 하위 클래스에 새로운 필드를 추가하며 equals를 재정의하는 경우 문제가 발생한다.  
이것도 예시를 보면 이해가 쉽다.  
`구체 클래스`
```java
public class Point {
    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override public boolean equals(Object o) {
        if(!o instanceof Point)
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```
`서브 클래스`
```java
public class ColorPoint extends Point {
  private final Color color;

  public ColorPoint(int x, int y, Color color) {
    super(x, y);
    this.color = color;
  }
  ...
}
```
서브 클래스인 ColorPoint 클래스에 Color라는 필드가 추가되었다.

만약 ColorPoint에서 equals를 재정의하지 않는다면 Point의 equals를 그대로 사용하게 되고 그러면 '색깔'이라는 속성을
비교하지 않게 된다. 이는 바람직하지 않다. 

우선은 잘못 재정의한 equals 예시를 보려고 한다.
```java
@Override public boolean equals(Object o) {
    if(!o instanceof ColorPoint)
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```
이 메서드는 ColorPoint 타입끼리의 equals 연산만 수행한다.  
즉, 이런 방식의 equals는 대칭성이 무너진다는 소리다. 실제로 아래와 같은 코드가 있으면 서로 다른 결과가 나온다.
```java
Point p = new Point(1,2);
ColorPoint cp = new ColorPoint(1,2, Color.RED);
p.equals(cp); // true
cp.equals(p); // false
```

그렇다면 equals를 아래와 같이 재정의하면 어떨까
```java
@Override public boolean equals(Obejct o){
  if(!(o instanceof Point))
    return false;
  if(!(o instanceof ColorPoint))
    return o.equals(this);
  return super.equals(o) && ((ColorPoint) o).color == color;
}
```
위와 같이 구현하면 대칭성 문제는 해결된다. 하지만 추이성이 무너진다.  
왜냐하면 Point와 ColorPoint 간의 비교에서는 좌표만 비교하고 ColorPoint 간의 비교에서는 색깔까지 비교하기 때문이다.  
실제로 아래와 같은 코드를 작성하면 추이성이 무너졌음을 파악할 수 있다.
```java
ColorPoint p1 = new ColorPoint(1,2, Color.RED);
Point p2 = new Point(1,2);
ColorPoint p3 = new ColorPoint(1,2, Color.BLUE);
p1.equals(p2); // true 
p2.equals(p3); // true 
p1.equals(p3); // false
```
> 그럼 해결책은 뭔가요..?

저자는 사실 이 문제가 모든 객체 지향 언어의 동치관계에서 발생하는 근본적인 문제라고 한다.  
'구체 클래스'를 확장하는 서브 클래스에서 새로운 필드를 추가함과 동시에 규약을 만족하는 equals를 재정의할 수 있는 방법은 존재하지 않는다고 한다.

###### 그럼 아예 방법이 없는 건가요?
그런 것은 아니다. 구체 클래스의 하위 클래스에는 필드를 추가하며 규약에 맞는 equals를 재정의할 순 없지만 그대신 `컴포지션`을 이용하면 된다.

`컴포지션`: 기존 클래스를 확장하는 대신에 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하는 방식

ColorPoint 클래스가 Point를 상속하는 것이 아니라 private 필드로 참조하게 하는 것이다. 그리고 만약 일반 Point가 필요하면 해당 필드를 반환하도록 하는 메서드를 제공하면 된다!
예시 코드를 보면 이해가 쉽다.
```java
public class ColorPoint {
  private final Point point;
  private final Color color;

  public ColorPoint(int x, int y, Color color) {
      point = new Point(x, y);
      this.color = Objects.requireNonNull(color);
  }
  
  // 일반 Point가 필요한 경우 point 필드 값을 반환(뷰 메서드)
  public Point asPoint(){ 
    return point;
  }

  @Override public boolean equals(Object o){
    if(!(o instanceof ColorPoint)){
      return false;
    }
    ColorPoint cp = (ColorPoint) o;
    return cp.point.equals(point) && cp.color.equals(color);
  }
}
```
이렇게 하면 규약을 준수하면서도 Color라는 필드를 추가한 equals 메서드를 구현할 수 있다.

> +) 위에서 지속적으로 '구체 클래스'라고 언급을 했었다. 그렇게 한 데에는 이유가 있다.   
> 만약 '추상 클래스'를 상속하는 서브 클래스라면 새로운 필드를 추가함과 동시에 규약을 만족하는 equals
> 메서드를 재정의할 수 있다.   
> 더 일반적으로 말하면 슈퍼 클래스가 인스턴스화가 되지 않는다는 보장만 있다면
> 새로운 필드를 추가하면서 규약을 만족하는 equals를 재정의할 수 있다! 


###### 일관성
두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 것을 뜻한다.

물론 가변 객체야 매번 비교하는 시점마다 그 결과가 다르게 나오겠지만 불변 클래스에 대해서는 
언제나 equals의 결과가 같아야 한다.

그러나 여기서 주의할 점은 불변 객체든 가변 객체든 `equals 로직에 신뢰할 수 없는 자원이 끼어들어선 안된다.`  
즉, equals 메서드 내부에서 자원 비교를 할 때 항상 메모리에 존재하는 자원만을 사용하여 비교를 해야 한다는 뜻이다.
이것을 제대로 지키지 못한 예시가 있다.  
바로 `java.net.URL`의 `equals` 메서드다. 이 equals 메서드는 주어진 URL과 매핑된 호스트의 IP주소를 이용해 비교하는데,
호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야 하므로 우리가 컨트롤할 수 없는 변수가 생길 수도 있다.

###### null-아님 (공식적인 이름은 아님)
이름에서 알 수 있듯이 그냥 모든 객체는 null과 같지 않아야 한다는 것을 뜻한다.

그래서 많은 사람들이 null 체크 로직을 다음과 같이 명시적으로 포함하는데
```java
@Override
public boolean equals(Object o) {
  if(o == null) { 
      return false;
  }
}
```
사실 불필요한 로직이다. 그대신에 다음과 같이 instanceof 키워드를 사용하면 된다.
```java
@Override
public boolean equals(Object o) {
  if(!(o instanceof MyType)) { 
      return false;
  }
  MyType myType = (MyType) o;
}
```
이것이 가능한 이유는 instanceof 키워드 자체가 첫 번째 피연산자가 null이라면 바로 false를 반환하도록 설계가 되어있기 때문이다.

###### 규약은 다 봤는데 그럼 equals 메서드를 어떻게 하면 잘 작성하는 건가요?
1. `==` 연산자를 이용하여 입력이 자기 자신의 참조인지 확인하자
  - equals 내부의 비교 로직이 복잡하고 오래 걸린다면 == 연산자를 이용하여 참조값을 비교하자. 참조값이 같을 때 바로 true를 리턴하도록 하면 최적화를 달성할 수 있다.
2. `instanceof` 연산자로 입력이 올바른 타입인지 확인하자
  - 매개변수로 들어온 타입이 예상한 타입과 다를 경우 바로 false를 반환하도록 하면 된다.
  - 보통은 instanceof의 두 번째 피연산자로 해당 equals 메서드가 포함된 클래스를 많이 사용하지만 가끔은 상위 인터페이스 타입을 사용할 수 있다.  
  그렇게 하면 해당 인터페이스를 구현한 모든 클래스들 간의 equals 연산을 수행할 수 있다.
3. 입력을 올바른 타입으로 형변환하자
  - 앞선 2번에서 instanceof로 타입 체크를 하므로 이건 자연스럽게 달성할 수 있다.
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사하자
  - 제대로 된 비교를 하기 위해선 대상 필드들이 모두 일치하는지 확인해야 한다.

##### 추가적인 주의사항
- `float`, `double`을 제외한 기본 타입
  - `==` 연산자 비교
- 객체
  - `equals` 비교
- `float`, `double`
  - 정적 메서드인 `Float.compare(float, float)`와 `Double.compare(double, double)` 사용(부동 소수점 때문에 그렇다..)
- 비교하기 복잡한 필드를 가진 경우 각 필드의 표준형(canonical form)을 저장하고 표준형끼리의 비교를 하면 훨씬 편하다.
  ```java
  public final class CaseInsensitiveString{
    private final String s;
    private final String canonicalForm;
  
    public CaseInsensitiveString(String s){
      this.s = Obejcts.requireNonNull(s);
      this.canonicalForm = s.toLowerCase();
    }
    
    @Override public boolean equals(Object o){
      return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).canonicalForm.equals(canonicalForm);
    }
  }
  ```
  여기선 소문자로만 구성된 문자열을 표준형으로 두고 해당 표준형을 비교하는 방식으로 equals를 구현했다.
- 다를 가능성이 높은 필드 혹은 비교 비용이 싼 필드를 먼저 비교하자
  - 성능 면에서 조금이라도 우위를 점할 수 있다.
- equals를 재정의할 때는 반드시 hashCode도 재정의하자
  - 이건 다음 item에서 다룬다
- equals 메서드의 매개변수로 Object가 아닌 다른 타입을 받도록 하지 말자
  - 매개변수로 Object가 아닌 다른 타입을 받로록 하면 그건 `오버라이딩`이 아니라 `메서드 오버로딩`이다.

마지막으로 규약을 잘 지킨 equals 예시를 보며 마무리하려고 한다
```java
public class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if(val < 0 || val > max) {
            throw new IllegalArgumentException(arg + ": " + val);
        }
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if(o == this) {
            return true;
        }

        if(!(o instanceof PhoneNumber)) {
            return false;
        }

        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }
}
```

## 핵심 정리
    웬만해선 Object가 제공하는 equals를 사용하자. 
    만약, 재정의해야 한다면 규약을 반드시 지키면서 모든 필드를 비교하도록 구현하자 