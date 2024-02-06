### Item50 : 적시에 방어적 복사본을 만들라

자바는 비교적 다른 언어에 비해 안전한 편이다. 타 언어에 비해 메모리 관련 오류에서 자유로운 편이다. 하지만 그렇다고 해서
마냥 안심할 수만은 없다.

###### 방어적 프로그래밍
실제로 그렇진 않겠지만 개발자들은 코드를 작성할 때 악의적인 사용자들이 언제라도 내 코드를 망가뜨릴 수 있다고 가정을 하고
작성을 하는 것이 좋다. 꼭 악의적인 사용자가 아니더라도 미숙한 사용자가 의도치 않게 코드를 망가뜨리는 경우도 많다. 그러므로
항상 위험을 예상하고 방어적으로 프로그래밍해야 한다.

###### 객체 내부의 변경
당연히 우리는 코드를 작성할 때 특정 클래스 인스턴스의 상태를 외부에서 함부로 수정할 수 없게끔 작성한다. 하지만 우리가 자칫하면 놓칠 수 있는 부분에서
허점이 발생한다.
```java
public class Car {
    private final Date createdAt;
    private final Date soldAt;
    
    public Car(Date createdAt, Date soldAt) {
        if (createdAt.compare(soldAt) > 0) {
            throw new IllegalArgumentException();
        }
        this.createdAt = createdAt;
        this.soldAt = soldAt;
    }
    
    public Date getCreatedAt() {
        return createdAt;
    }
    public Date getSoldAt() {
        return soldAt;
    }
}
```
위 코드는 예시 코드로서 자동차를 나타내는 클래스이며 내부에는 자동차 생산 시점과 판매 시점을 나타내는 필드를 Date 타입으로 가지고 있다. 그리고
해당 날짜들은 변하면 안되므로 final로 설정하여 변경 가능성을 제거했다.  
따라서 위 코드는 외부에서의 변경으로부터 안전해 보이지만 사실은 허점이 존재한다.
```java
public static void main(String[]args){
    Date createdAt = new Date();
    Date soldAt = new Date();
    Car car = new Car(createdAt, soldAt);
    createdAt.setYear(98); // 외부에서의 변경
}
```
Car 인스턴스가 내부적으로 유지하는 Date 객체들은 재할당될 수 없는 것이 맞다. 하지만 해당 객체가 가지는 내부값 자체를 변경할 순 있다.  
바로 위 코드에서도 Date 객체를 이용하여 Car 객체를 생성한 뒤 해당 Date 객체의 메서드를 이용해서 값을 변경한다. 이렇게 되면 결국엔 Car 객체를
외부에서도 변경 가능하게 된 것이다. 이 허점을 보완하려면 적절한 시점에 `방어적 복사`를 실시하면 된다.
```java
public Car(Date createdAt, Date soldAt) {
    this.createdAt = new Date(createdAt.getTime());
    this.soldAt = new Date(soldAt.getTime());
	
    if (createdAt.compare(soldAt) > 0) {
        throw new IllegalArgumentException();
    }
}
```
이렇게 Car 객체 생성 시점에 Date 객체를 방어적으로 복사하여 생성하고 그 인스턴스를 저장하면 만약 외부에서 Date 객체를 수정하더라도 실제 Car 객체 안에 
들어있는 Date 객체와는 별개의 객체이므로 외부에서의 수정을 막을 수 있다.  
그리고 위 코드에서는 불변식 검사를 방어적 복사 이후에 수행하는 것을 알 수 있다. 그 이유는 멀티 쓰레드 환경에서 만약 검사를 먼저 실행할 경우 검사 완료와 방어적 복사가
일어나는 그 찰나의 순간에 다른 쓰레드에서 원본 객체를 수정할 수도 있기 때문이다.  
그리고 방어적 복사도 clone() 메서드를 사용하지 않았음을 알 수 있다. 생성자의 매개변수로 받는 객체가 다른 사용자들이 상속하여 사용할 수 있는 클래스라면 clone()을 섣불리 사용하면 안된다.

이로써 외부로부터의 변경 가능성을 모두 없앴다고 생각할 수 있지만 아직 한 가지 문제가 더 있다.
```java
public static void main(String[]args){
    Date createdAt = new Date();
    Date soldAt = new Date();
    Car car = new Car(createdAt, soldAt);
    car.getCreatedAt().setYear(98); // 외부에서의 변경
}
```
getter 메서드에서 내부 필드를 그대로 반환하므로 외부에서는 getter 메서드를 호출한 뒤 Date 객체를 수정할 수 있게 된다. 다행히 이 문제는 해결하기가 쉽다.
```java
public Date getCreatedAt() {
    return new Date(createdAt.getTime());
}
public Date getSoldAt() {
    return new Date(soldAt.getTime());
}
```
마찬가지로 getter 메서드 내부에서도 방어적 복사를 한 인스턴스를 반환하면 되는 것이다.  
getter 메서드의 경우에는 clone() 메서드를 이용해서 복사한 객체를 리턴해도 된다. 해당 객체를 clone() 복사해도 안전함이 명확하기 때문이다.

## 핵심 정리
    외부에서 받는 매개변수 혹은 외부로 보낼 인스턴스가 가변이라면 방어적 복사를 고려하자.