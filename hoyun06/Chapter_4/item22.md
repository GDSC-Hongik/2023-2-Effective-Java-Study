### Item22 : 인터페이스는 타입을 정의하는 용도로만 사용하라

###### 인터페이스의 바람직한 역할
인터페이스의 역할은 그걸 구현하는 클래스의 인스턴스로 할 수 있는 것들을 나타내는 것이다. 명세서라는 것이다.  
이 역할을 충족하지 못하는 안 좋은 인터페이스의 예시가 있다.

###### 인터페이스 안티 패턴(상수 인터페이스)
`상수 인터페이스`  
- 메서드는 없고 static final로 선언된 필드들만 있는 인터페이스.
```java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
    
    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```
위 인터페이스는 잘못된 인터페이스의 예시이다.  
클래스 내부에서 사용되는 상수는 공개 api가 아닌 내부 구현에 해당한다. 그러므로 굳이 노출해선 안된다.

###### 상수를 외부로 노출해야 하는 경우
만약 상수를 굳이 외부로 노출해야 한다면 다른 선택지를 고르자.
1. 인터페이스에 상수를 선언하지 말고 해당 클래스 내부에 상수를 선언하고 공개하자.
   - 기본 타입의 래퍼 클래스인 Integer, Double 등의 MAX_VALUE, MIN_VALUE가 여기에 해당한다.
2. 열거 타입으로 나타내기 적합하다면 열거 타입으로 공개하자
3. 정적 유틸리티 클래스에 상수를 선언하자
   - 인스턴스 불가 클래스를 설계하고 그 내부에 상수를 선언하자. 그러면 '클래스명.상수'와 같은 식으로만 접근할 수 있다.

## 핵심 정리
    인터페이스는 타입 정의 용도로만 사용하자. 상수를 공개하고자 한다면 다른 선택지를 고르자.