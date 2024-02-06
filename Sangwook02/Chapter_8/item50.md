# Item 50. 적시에 방어적 복사본을 만들라

자바로 작성한 클래스는 다른 언어에 비해 안전함.  
여전히 안전하지 않은 클래스도 존재

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException();
        }
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
}
```
Period는 불변 클래스가 맞지만,  
Date는 가변이므로 Date를 조작하면 쉽게 Period의 불변식을 깨뜨릴 수 있다.
```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
start.setMonth(11);
```

이런 공격을 방어하기 위해 생성자에서 받은 가변 매개변수 각각을 복사하여 사용
```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0) {
        throw new IllegalArgumentException();
    }
}
```

### 유효성 검사 전에 복사본을 먼저 만든다
유효성 검사를 한 후 복사본을 만들기까지의 순간에 금방 다른 스레드가 조작할 수 있기 때문

### clone 메서드를 사용하지 않는다
하위 클래스에서 악의적으로 재정의할 수 있기 때문

#### 정리
항상 객체가 잠재적으로 변경될 수 있는지를 생각해보자