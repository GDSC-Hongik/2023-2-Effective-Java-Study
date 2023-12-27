# Comparable을 구현할지 고려하라
---
## 먼저 Comparable과 compareTo가 무엇일까
이것은 객체를 비교하기 위해 존재한다.
숫자의 경우 등호, 부등호등으로 쉽게 비교할 수 있지만, 객체의 경우 무슨 기준으로 비교할 지 명확하지 않다.
따라서 이런 기준을 만들고, 비교를 하기 위해서 사용되는 것이 `Comparable`인터페이스의 `compareTo`메서드이다.

참고로 `compareTo`메서드는 리턴값이 양수, 0, 음수인지에 따라 결과를 판단한다.
또한 비교가 불가능한 객체가 입력으로 들어오면 `ClassCastException`을 던진다.

예를 들어보자
```java
public class Student implements Comparable<Student> {
	@Override
	public int compareTo(Student o) {
		return this.age - o.age;
	}
}
```
위는 `Comparable`인터페이스를 상속하여 `compareTo`메서드를 재정의 한 것이다.

리턴값으로는
* 자기 자신의 나이가 더 많으면 양수
* 같으면 음수
* 적으면 음수
를 리턴한다.

## 혹시 Comparator하고 햇갈리지 않았는가?
`Comparator`인터페이스는 `compare(T o1, T o2)`를 구현해야 한다.
위의 `compareTo(T o)`는 `compareTo(T B)`와 같이 사용하여 자기 자신 객체와 B라는 객체를 비교하는 것에 비해, 이는 `compate(T A, T B)`처럼 사용하여 두 객체 A와 B를 비교한다.


---
그럼 이제 본격적으로 Comparable을 구현할지 고려하라에 대해 알아보도록 하자.

 
Object에도 equals가 존재하는 데, 왜 굳이 `Comparable`을 `implements`하여 `compareTo`를 구현하는 지 의문이 들 수 있다.
## Object의 equals와 무엇이 다를까?
`equals`는 단순히 같은 지만 판단하는 반면, `compareTo`는 크고,같고, 작음등의 순서까지 비교할 수 있다.

따라서 `compareTo`를 구현한 클래스는 순서를 가짐을 인식할 수 있고, 
따라서 순서가 있으므로 다음과 같이 쉽게 객체들을 정렬할 수 있다.
```java
Arrays.sort(a);
```

## `Comparable`을 구현하면 뭐가 좋을까?
다음 코드를 다시 보자.
```java
Arrays.sort(a);
```
이는 `Arrays.sort()`메서드를 사용한 것이다. 별도의 조치없이도 객체들을 정렬할 수 있었다.

이는 `Comprable` 인터페이스를 활용하는 수많은 제네릭 알고리즘과 컬렉션이 존재하기 때문이다.

따라서 `compareTo`만 구현한다면 정렬, 최대값,검색등의 관리를 매우 쉽게 할 수 있다.

참고로 `java`에서는 모든 값 클래스와 열거 타입이 `Comparable`을 구현했다.

따라서 순서가 있는 값 클래스를 구현한다면 `compareTo`를 구현하자

### compareTo 메서드 일반규약

> 이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의정수를 같으면 `0`을, 크면 양의 정수를 반환한다. 비교할 수 없는 타입이 주어지면 `ClassCastException`을 던진다.

- 아래 설명에서 `sgn`은 부호함수를 뜻하며, 표현식의 값이 `음수`, `0`, `양수`일 때 `-1`, `0`, `1`을 반환하도록 정의했다.
- 만약, 기존 클래스를 확장한 구체 클래스가 값 필드를 추가했다면 `compareTo`규약을 준수할 방법이 없다. 따라서 이런 경우 조합(Composition)을 이용해 우회적으로 규약을 준수하자.

#### 대칭성

두 객체참조의 순서를 바꿔 비교해도 예상한 결과가 나와야한다.  
`Comparable을 구현한 클래스는 모든 x, y에 대하여 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다. 따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질때에 한해 예외를 던져야 한다.`

#### 추이성

첫 번째가 두번째보다 크고 두 번째가 세번째보다 크면 첫번째는 세번째보다 커야한다.  
`Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (x.compareTo(y)>0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0` 이다.

#### 반사성

크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같다.  
`Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))이다.`

#### equals

`compareTo` 메서드로 수행한 동치성테스트 결과가 `equals`와 같아야한다.  
`(x.compareTo(y) == 0) == (x.equals(y))여야 한다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. (주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다.)`




## compareTo를 구현할 때는 <,>대신에 compare를 이용하자
정수 기본 타입 필드와, 실수 기본 타입 필드를 비교할 떄는 오류를 유발하는 <,>대신에 Integer.compare, Double.compare, Float.compare등을 사용하자.

## 클래스에 핵심 필드가 여러개라면 가장 핵심적인 필드부터 비교해 나가자.
가장 핵심적인 필드부터 비교해나가다가, 
결과가 0이 아니라면 그 결과를 바로 리턴하며
결과가 0이라면 똑같지 않은 필드를 찾을 때까지 그 다음으로 중요한 필드를 비교해 나간다.

## `Compartor`의 메서드 연쇄 방식 비교자 생성
자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.

```java
private static final Comparator<PhoneNUmber> COMPARATOR = 
	comparingInt((PhoneNumber pn) -> pn.areaCode)
		.thenComparingInt(pn-> pn.a)
		.thenComparingInt(pn -> pn.b);

public int compareTo(PhoneNumber pn) {
	return COMPARATOR.compare(this, pn);
}
```
이와 같이,
먼저 pn의 areaCode를 비교대상으로 삼아 comparingInt를 통해 정렬한다.
추가로 정렬 필요하다면
thenComparingInt를 통해 정렬한다.

또한 이의 리턴 값은 `Comparator`이다.
따라서 이렇게 COMPARATOR 클래스를 생성하고, 
이 안의 compare메서드를 호출하여 비교한다.


참고와 이 `comparingInt`와 같이 `Comparator`는 많은 보조 생성 메서드를 가지고 있다.
* long과 double용으로는 comparingInt와 thenComparingInt의 변형 메서드도 있다.