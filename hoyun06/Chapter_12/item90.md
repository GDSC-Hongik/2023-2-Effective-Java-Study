### Item90 : 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

###### 직렬화 프록시
이번 주차 item에서 지속적으로 다루고 있지만 직렬화/역직렬화는 결국 언어에서 지정한 정상적인
객체 생성 매커니즘을 사용하지 않는다. 그래서 이 방식은 버그와 보안 문제가 발생할 여지가 충분하다.  
이를 대처하기 위해 사용할 수 있는 방법이 `직렬화 프록시`다

이 방법은 복잡하지 않다.  
외부 클래스의 논리적 상태를 정확히 나열하는 내부 중첩 클래스(private static)를 설계하면 된다. 이때 이 내부 중첩 클래스에는
오로지 한 개의 생성자만 있어야 하는데 이 생성자는 매개변수로 바깥 클래스를 받아야 한다. 그리고 매개변수로 받은 바깥 클래스 인스턴스로부터
단순히 데이터를 복사하면 된다.   
당연하지만 바깥 클래스와 내부 중첩 클래스 모두 `Serializable`을 구현한다고 선언해야 한다.

```java
public class Period implements Serializable {
    private final Date start;
    private final Date end;
    
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        
        if (this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException();
        }
    }
    
    private Object writeReplace() {
        return new SerializationProxy(this);
    }
    
    private static class SerializationProxy implements Serializable {
        private final Date start;
        private final Date end;
        
        SerializationProxy(Period p) {
            start = p.start;
            end = p.end;
        }
        
        private static final long serialVersionUID = 100222315L;
    }
}
```
위 코드에서 바깥 클래스(Period)에 writeReplace 메서드를 정의한 것을 볼 수 있다.  
이는 자바 직렬화 시스템이 바깥 클래스 인스턴스 대신 SerializationProxy 인스턴스를 반환하도록 하는 메서드다.
즉, 직렬화가 실행되기 전에 Period 인스턴스를 SerializationProxy 인스턴스로 변환하는 메서드다.

이 다음으로 SerializationProxy 클래스 내부에 readResolve 메서드를 추가하면 된다.
```java
private Object readResolve() {
    return new Period(start, end);
}
```
이 메서드는 writeReplace와 정반대 역할을 하는 메서드라고 생각하면 되는데 역직렬화 시 SerializationProxy 인스턴스를
바깥 클래스의 인스턴스로 변환해주는 역할을 한다.

###### 직렬화 프록시 장점
- 자바 언어의 정석적인 객체 생성 방식을 준수
  - 기존의 직렬화 방식과는 다르게 직렬화 프록시의 경우 생성자 혹은 정적 팩터리를 통해 객체를 생성한다
  그러므로 이전의 직렬화 방식에서 사용하던 유효성 검사와 같은 추가적인 수단을 필요로 하지 않는다 
- 악의적인 바이트 스트림 공격도 프록시 수준에서 막을 수 있다
- 직렬화 프록시의 경우 바깥 클래스의 필드를 final로 선언할 수 있다
  - 기존 직렬화 방식의 경우 방어적 복사로 인해서 final 선언을 지웠어야 했는데 직렬화 프록시의 경우 final을 그대로
  사용할 수 있어 불변 클래스를 유지할 수 있다

###### 직렬화 프록시 한계
- 클라이언트가 멋대로 확장할 수 있는 클래스에 대해선 적용할 수 없다
- 객체 그래프에 순환이 있는 클래스의 경우에도 적용할 수 없다
- 일반적인 직렬화 방식에서의 readObject 메서드 + 방어적 복사를 사용한 방식보다 속도가 느리다

