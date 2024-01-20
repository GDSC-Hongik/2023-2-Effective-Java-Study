# Item 11. equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 HashMap이나 HashSet 같은 컬렉션을 사용하는 클래스를 사용할 때 문제가 발생할 수 있다. 이는 equals를 재정의하지 않은 클래스를 HashMap이나 HashSet에 담아 사용할 때 발생한다. 이런 경우에는 equals가 아닌 hashCode를 사용하기 때문이다. 

HashMap이나 HashSet은 내부적으로 해시 테이블을 사용하는데, 이 때 hashCode를 사용한다. 따라서 equals를 재정의하지 않은 클래스를 HashMap이나 HashSet에 담아 사용하면 hashCode가 다르기 때문에 같은 객체라고 판단하지 않는다. 이는 equals를 재정의하지 않은 클래스를 HashMap이나 HashSet에 담아 사용할 때 발생하는 문제이다.

해시코드를 반환할 때는 서로 다른 인스턴스들을 32비트 정수 범위에 분배해야 한다. 필드 값을 어떻게 계산하여 result로 변환할 것인지는 타입마다 달라진다. 하지만 롬복의 EqualsAndHashCode을 쓰면 간단하다. 하지만 이때 변하는 멤버를 대상으로 해시코드를 재정의하면 안된다. 이는 해시코드를 계산하는 과정에서 변하는 멤버가 있다면 해시코드가 달라지기 때문이다.
