# Item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.  
### 상수 인터페이스 안티패턴
아래와 같이 static final 필드로만 가득 찬 인터페이스를 상수 인터페이스 안티패턴이라고 한다.  
이러한 상수들은 내부 구현에 해당하는데 이걸 외부 인터페이스에 두고 사용하면 내부 구현을 노출하는 것과 같다.
```java
public interface PhysicalConstants {
	static final double AVOGADROS_NUMBER = 6.022_140;
	static final double BOLTZMANN_CONSTANT = 1.380_648;
	static final double ELECTRON_MASS = 9.109_383;
} 
```

상수를 처리하기 위한 방법은 여러 대안이 있다.
1. 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스에 직접 추가
2. 열거가 적당한 경우라면 Enum 타입으로 만들어도 좋다
3. 인스턴스화 할 수 없는 유틸리티 클래스로 만들기  
아래는 위의 인터페이스를 상수 유틸리티 클래스로 만든 예시이다.
    ```java
    public class PhysicalConstants {
        private PhysicalConstants() {}
        
        public static final double AVOGADROS_NUMBER = 6.022_140;
        public static final double BOLTZMANN_CONSTANT = 1.380_648;
        public static final double ELECTRON_MASS = 9.109_383;
    } 
    ```
   
**정리**  
interface는 타입을 정의하는 용도로만 사용하고 상수 공개용으로 사용하지 말자.