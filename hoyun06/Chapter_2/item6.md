### Item6 : 불필요한 객체 생성을 피하라

###### 불필요한 객체의 생성
똑같은 기능을 제공하는 객체를 자주 사용한다면 매번 새로운 객체를 생성하는 것보다 한 번 생성하여 
 재활용하는 것이 효율적이다.   
 ```java
String s = new String("Sponge Bob");
```
- 위 예시에서 String 생성자의 매개변수로 들어가는 문자열은 생성자를 통해 만들어지는 String 객체와
정확히 일치한다.  
- 만약 위와 같은 코드가 프로그램 전반적으로 반복된다면 불필요한 String 객체가 계속해서 생성된다..

따라서 다음과 같이 사용하자
```java
String s = "Sponge Bob";
```
- 이처럼 " " 를 이용한 String literal을 이용하면 해당 객체는 String Pool이라는 곳에 저장된다.  
- 이후에 같은 값을 가지는 String 객체를 생성하면 String Pool에 있는 객체의 래퍼런스를 참조하도록 하여 하나의 객체를
공유할 수 있다.

Item1에서 배웠던 것처럼 불변 클래스에서 생성자 대신 정적 팩터리 메서드를 제공하면
객체를 매번 생성하지 않고 재사용할 수 있다.
```java
public class Car {
    private static final Car Instance = new Car();
    
    private Car() {}
    
    public static Car getInstance() {
        return Instance;
    }
}
```
설령 해당 클래스가 가변 클래스이더라도 변하지 않을 것이란 확신이 있을 경우 동일 인스턴스를 재사용해도 문제가 없다.  

###### 객체 재사용의 이점
- 코드의 성능을 증가시킬 수 있다.  
생성 비용이 큰 객체를 매번 생성하는 것은 실행 속도를 저하시킨다.
```java
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
        + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
String 클래스의 matches 메서드는 매개변수로 주어진 regex 패턴 만족 여부를 알려준다.  
이 방식은 문자열의 형태를 파악하기 편하지만 성능상 그리 좋지만은 않다.

- matches 메서드는 내부적으로 regex용 Pattern 객체를 생성하는데 
이 Pattern 객체는 그 regex에 해당하는 유한 상태 머신을 생성하기 때문에 생성 비용이 매우 크다.  
- 심지어 matches 메서드 종료 이후 이 Pattern 객체는 GC의 수거 대상이 돼서 수거가 되므로 생성 비용이 큰 것에 비해 
그 쓰임은 비교적 초라하다.

그렇기에 위 코드처럼 메서드를 작성하지 말고 다음과 같이 작성하자
```java
public class RomanNumeral {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    
    public static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```
이와 같이 코드를 작성하면..
1. regex 패턴을 ROMAN이라는 이름으로 분리함으로써 코드상에서 의미가 더 명확해진다.
2. 클래스 생성 시 초기화된 ROMAN 객체를 재사용할 수 있다.


    생성 비용이 큰 객체를 시작 시에 생성/캐싱하고 필요할 때마다 재사용함으로써
    코드 성능을 증가시킬 수 있다.

###### 불필요한 객체를 만드는 예시
**오토 박싱**  
오토 박싱은 프로그래머가 primitive 타입과 해당 타입의 wrapper 클래스를 같이 사용할 때 자동으로 변환해주는 것을 의미한다  
```java
Long id = 1L; // long 타입을 Wrapper class인 Long으로 자동 변환
```
오토 박싱은 프로그래머가 형변환을 신경쓰지 않고 코드를 작성할 수 있게 해준다.  
그렇다고 해서 primitive type과 wrapper class 사이의 간극을 아예 없애주는 것은 아니다.

다음 예시를 보자
```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum;
}
```
이 코드는 실제로 의도한대로 잘 동작하지만 성능면에서는 그리 좋지 않다.  
문제가 되는 부분은 `sum += i` 부분인데 여기서 i는 long 타입이고 sum은 Long 타입이므로 
매 iteration 마다 덧셈 연산을 수행하면서 불필요한 Long 객체가 생성된다.

위 코드에서 `Long sum` 부분을 `long sum` 으로만 바꿔줘도 불필요한 Long 객체가 생성되지 않으면서 성능이 
훨씬 좋아진다!

###### item6 정리
이번 아이템은 '불필요한 객체 생성을 피하라'라는 주제였다.  
>하지만 이것이 절대적으로 객체 생성을 피하라는 뜻은 아니다 

요즘 JVM 성능도 좋아졌고 실제로 프로그램의 명확성, 간결성을 나타태기 위한 객체 생성은 도움이 된다.

> 반대로 객체 재사용성을 위해서 본인만의 객체 풀(pool)을 만드는 일은 웬만해선 하지 말자

물론 DB connection과 같이 생성 비용이 엄청 큰 객체에 대해서는 pool을 유지하면서 재사용을 하는 것이 성능면에서 훨씬
이득이 생긴다.  
하지만 이외에 대부분의 경우 객체 풀은 코드를 더 헷갈리게 하고 메모리적으로도 오히려 손해가 생긴다.
