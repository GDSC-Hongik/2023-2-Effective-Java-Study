### Item14 : Comparable을 구현할지 고려하라

###### Comparable 인터페이스
```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```
Comparable 인터페이스는 compareTo라는 메서드 단 한 개만 지원한다.  
이 메서드는 equals와 거의 유사하지만 다른 점이 있다.
- compareTo 메서드는 `동치성 비교 + 순서 비교`가 가능하다.
- `제네릭 프로그래밍`이 가능하다.
Comparable을 구현한 클래스는 객체 간 순서를 정할 수 있음을 의미하므로 다음과 같이 sort 메서드로 쉽게 활용할 수 있다.
```java
Arryays.sort(cars)
```

###### compareTo 메서드 일반 규약
이 메서드는 자신과 매개변수로 주어진 객체를 비교한다.  
만약 자신이 더 앞선다면 음수를, 순서가 동일하다면 0을, 더 뒤라면 양수를 반환한다.

**아래에서 사용되는 sgn은 괄호 안의 값이 음수면 '-', 양수면 '+', 0이면 0을 반환한다.**  

**대칭성**
> sgn(x.compareTo(y)) == -sgn(y.compareTo(x))을 보장해야 한다.  
> (만약 x.compareTo(y)가 예외를 던지면 y.compareTo(x)도 던져야 한다.)

**추이성**
> x.compareTo(y) > 0 이고 y.compareTo (z) > 0 이면 x.compareTo(z) > 0을 보장해야 한다.  

**반사성**
> x.compareTo(y)==0 이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z)) 여야 한다.

**equals와의 관계**
> (x.compareTo(y) == 0) == (x.equals(y))여야 한다.
> (단, 이 규약은 지키면 좋으나 강제성은 없다.)

위 compareTo 규약을 제대로 지키지 않으면 순서 비교를 활용하는 클래스를 사용할 때 문제가 생긴다.

그리고 규약의 이름들을 보면 equals 규약과 많이 일치하는 것을 알 수 있다. 그래서 사실 문제점도 똑같다.

**위 compareTo 일반 규약을 준수하면서 서브 클래스에서 새로운 값 컴포넌트를 추가하는 것은 불가능하다. 하지만 다행인 점은 해결법도 똑같다는 것이다.**

Comparable을 구현하는 클래스를 상속하면서 새로운 값 컴포넌트를 추가하고 싶은 경우 `컴포지션`을 활용하자.
새로운 클래스를 생성하고 그 내부 필드로 `Comparable을 구현하는 클래스`를 참조하도록 하자. 그리고 해당 참조를 반환하는 view 메서드까지 제공하면 된다.
```java
public class Car {
    // 구현
}

public class ColorCar {
    private Color color;
    private Car car;
	
    public Car asCar() {
        return car;
    }
}
```
위와 같이 코드를 작성하면 우리가 원하는대로 compareTo 메서드를 구현하여 ColorCar 클래스 내부에 위치시킬 수 있다.

그리고 마지막으로 'equals와의 관계' 규약은 필수는 아니지만 꼭 지키는 것이 좋다. 이 규약을 지키지 않더라도 코드는 동작하긴 한다.
하지만 정렬된 컬렉션에 해당 인스턴스를 넣으면 compareTo와 equals 사이의 불일치로 인한 문제가 발생할 것이다.

###### compareTo 작성 요령
equals와 대부분 유사하나 차이점은 있다.
1. 구현한 Comparable은 제네릭이므로 compareTo 메서드의 매개변수 타입은 컴파일 타임에 결정된다.
   - 따라서, 굳이 타입 체크 및 타입 캐스팅을 할 필요가 없다.
2. 비교하려고 하는 객체가 내부적으로 다른 객체를 참조하고 있다면 해당 객체의 compareTo를 재귀적으로 호출하자.
    - 만약, 내부적으로 참조하고 있는 클래스가 Comparable을 구현하지 않았다면 Comparator을 이용하자(직접 생성 or 자바 제공 비교자 사용).
    ```java
    public class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
        
        public int compareTo(CaseInsensitiveString cis) {
            return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s); 
        }
    }
    ```
3. compareTo 내부에서 기본 타입에 대한 비교 연산 수행 시 compare 정적 메서드를 사용하자.
    - 자바 7부턴 기본 타입을 박싱한 Wrapper 클래스에서 비교 메서드인 compare을 제공한다. 따라서 정수든 실수든 모두 compare를 사용하자.
4. 비교할 필드가 여러 개라면 핵심 필드부터 비교하자
    - 이것도 equals랑 유사한 느낌인데 순서가 명확할 것 같은 필드부터 비교하자. 만약 해당 필드를 비교한 결과가 0이 아니라면 바로 결과를 반환하면 된다.

###### Comparator 인터페이스
아까 위에서 Comparator에 대해 잠시 언급했다.

Comparator는 자바 8부터 메서드 체이닝 방식을 이용하여 비교자를 생성할 수 있도록 해줬다.
```java
private static final Comparator<PhoneNumber> COMPARATOR =
    Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)
            .thenComparingInt(pn -> pn.prefix)
            .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber phoneNumber) {
    return COMPARATOR.compare(this, phoneNumber);
}
```
위 코드는 2개의 비교자 생성 메서드를 사용하여 비교자를 생성했다.
- **comparingInt 메서드**
  - Comparator의 정적 메서드
  - 순서를 결정 짓는 '키'를 추출하는 함수를 매개변수로 넘기면 해당 '키'로 순서 비교를 하는 비교자를 리턴한다.
  - 여기선 람다식으로 함수를 넘겼는데 이때 타입을 명시해줘야 한다.
- **thenComparingInt 메서드**
  - Comparator의 인스턴스 메서드
  - 마찬가지로 '키' 추출 함수를 넘기면 해당 '키'를 사용하는 비교자를 리턴한다.
  - comparingInt로 비교했을 때 순서가 동일한 경우에 다음 비교 대상으로 비교하는 경우에 실행된다.
  - 원하는 만큼 호출할 수 있다.
  - 이 메서드에선 굳이 타입을 명시할 필요가 없다.

위 두 메서드 말고도 long, double 형을 위한 비교자 생성 메서드도 제공한다.
또한 참조하고 있는 객체에 대한 비교자 생성 메서드도 제공한다!
- **comparing** (메서드 오버로딩이 되어 있다)
  - 매개변수로 키 추출자 함수만 받는 경우 -> 추출한 키의 자연적 순서로 비교
  - 매개변수로 키 추출자 함수, 비교자 받는 경우 -> 키를 추출하고 해당 키를 이용하는 비교자로 비교
- **thenComparing** (메서드 오버로딩이 되어 있다)
  - 비교자만 받는 경우 -> 해당 비교자로 비교
  - 키 추출자 함수만 받는 경우 -> 해당 키의 자연적 순서로 비교
  - 키 추출자 함수, 비교자 -> 키 추출하고 해당 키를 이용하여 비교자 사용

###### 주의점
가끔 보면 비교하려는 두 대상의 차('-')를 이용하여 순서 비교를 하는 경우가 있는데 이는 굉장히 위험하다.
자칫하면 `정수 오버플로우`나 `부동 소수점 계산 방식에 따른 오류`가 발생할 수도 있다.

## 핵심 정리
    순서를 비교하고자 한다면 Comparable을 꼭 구현하자. 
    기본 타입에 대한 비교는 박싱 클래스의 compare 메서드를 이용하고 
    참조하는 객체를 비교할 때는 비교자를 이용하자.