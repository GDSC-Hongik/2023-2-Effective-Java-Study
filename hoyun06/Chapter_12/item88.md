### Item88 : readObject 메서드는 방어적으로 작성하라

readObject 메서드는 사실상 바이트 스트림을 매개변수로 받는 public 생성자와 동일하게 생각해야 한다.
따라서 보통의 생성자에서 실시하는 작업들을 모두 반영하여 작성해야 한다.

###### 유효성 검사
readObject는 바이트 스트림을 전달받아 역직렬화 과정을 통해 객체를 재구성한다.  
이때 매개변수로 넘어오는 바이트 스트림이 항상 유효한 값을 가지는 바이트 스트림이라는 보장은 없다.  
따라서 커스텀 readObject 메서드를 작성하여 deaultReadObject 메서드를 실행한 뒤에 생성된 객체의
유효성 검사(불변식 검사)를 진행해야 한다.
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    if(start.compareTo(end) > 0) {
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```
위 코드에서는 불변식 검사를 통과하지 못 하면 'InvalidObjectException'을 던지도록 설계됐다.

###### 방어적 복사
설령 해당 클래스가 불변 클래스이더라도 해당 클래스 객체를 직렬화한 바이트 스트림 뒤에 '악의적인' 객체 참조를 추가하면
그 내부를 수정할 수 있게 된다. 따라서 '불변'에 의존하지 말고 커스텀 readObject 메서드를 작성하여 방어적 복사를 실시해야 한다.
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    start = new Date(start.getTime());
    end = new Date(end.getTime());
    
    if(start.compareTo(end) > 0) {
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```
위 코드에선 start, end 필드에 대해 방어적 복사를 실시하고 나서 바로 불변식 검사까지 실시하는 것을 볼 수 있다.
이렇게 해야 비로소 안전한 역직렬화가 가능해진다.  
다만 final이 붙은 필드(start, end)는 방어적 복사가 불가하기 때문에 위 방법을 사용하기 위해선 어쩔 수 없이
final을 제거해야 한다. 하지만 이를 통해 얻는 직렬화/역직렬화에서의 안전성이라면 충분히 고려할만 하다.

###### 재정의 가능 메서드 호출 금지
만약 해당 클래스가 final 클래스가 아니라면 readObject 내에서 재정의 가능 메서드를 (직/간접적으로)호출해선 안된다.  
이는 생성자에서 적용됐던 내용과 일맥상통하는 부분인데 readObject 내에서 재정의 가능 메서드를 호출하면 
아직 역직렬화가 제대로 진행되지 않은 하위 클래스의 재정의 메서드가 실행되면서 해당 하위 클래스 객체 내부가 
깨질 위험이 있다.

## 핵심 정리
    안전한 readObject를 메서드를 만드는 지침
    - private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하자
    - 모든 불변식을 검사하고 어긋나는 게 발견되면 InvalidObjectException을 던지자.
      방어적 복사 후에는 반드시 불변식 검사가 뒤따라야 한다.
    - 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하자
    - 직접이든 간접이든, 재정의할 수 있는 메서드는 호출하지 말자.