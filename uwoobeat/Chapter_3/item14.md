# Item 14. Comparable을 구현할지 고려하라

Comparable 인터페이스를 구현하기 위해서는 해당 클래스의 인스턴스를 어떠한 순서에 따라 비교할 수 있다는 것이다.따라서 해당 인스턴스에 대한 컬렉션을 `Collections.sort()` 메서드를 통해 정렬할 수 있다. 뿐만 아니라 이러한 비교 메서드를 통해 구현되는 검색 기능 등을 사용할 수 있다. 

String 역시 Comparable 인터페이스를 구현하고 있다. 따라서 String 인스턴스들을 정렬하거나 검색하는 등의 기능을 사용할 수 있다.

Comparable을 구현하기 위해서는 해당 인터페이스의 유일한 메서드인 `compareTo()`를 구현해야 한다. 이 메서드는 인자로 받은 객체와 자기 자신을 비교하여, 자신이 더 작으면 음수, 같으면 0, 더 크면 양수를 반환한다. 이때 compareTo 메서드의 규악을 지켜서 구현해야 한다. 이 규약을 간단히 정리하자면 다음과 같다.

- a를 b와 비교한 값과 b를 a와 비교한 값의 부호가 반대여야 한다.
- 추이성, 즉 비교결과에 대하여 a > b 이고 b > c 이면 a > c 여야 한다.
- x와 y를 compareTo 메서드를 통해 비교한 값이 0이라면 이는 즉 x.equals(y) 이다.

만약 Comparable을 구현한 클래스를 확장시키고 싶다면 상속이 아닌 컴포지션을 통해 구현해야 한다. 그리고 이 클래스에 대하여 Comparable을 다시 구현하는 방식을 채택할 수 있다.

compareTo를 어떻게 구현하면 좋을까? 해당 클래스 멤버 중 가장 중요한 것부터 비교해 나가면 된다. 일종의 early return을 지켰다고 보면 된다. 만약 핸드폰 번호를 비교한다고 해보자. 이때 번호는 (area code, prefix, line number) 순으로 비교하면 된다. 이때 area code를 비교하고, 같다면 prefix를 비교하고, 같다면 line number를 비교하면 된다. 이때 각각의 비교 결과를 반환하면 된다. 

```java
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0) {
            result = Short.compare(lineNumber, pn.lineNumber);
        }
    }
    return result;
}
```

하지만 indentation이 깊어지는 문제가 발생한다. 람다식과 스트림을 사용하면 훨씬 더 깔끔하게 구현할 수 있다.

```java
private static final Comparator<PhoneNumber> COMPARATOR =
    comparingInt((PhoneNumber pn) -> pn.areaCode) // areaCode를 비교한다.
        .thenComparingInt(pn -> pn.prefix) // areaCode가 같다면 prefix를 비교한다.
        .thenComparingInt(pn -> pn.lineNumber); // prefix가 같다면 lineNumber를 비교한다.

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

스트림의 장점은 선언형 프로그래밍이 가능하다는 것이고 위의 코드 예시가 아주 대표적이다. '어떻게' 보다는 '무엇을'에 집중하는 것! 하지만 고전적인 방식에 비해 성능이 떨어질 수 있다는 점을 염두에 두어야 한다. 