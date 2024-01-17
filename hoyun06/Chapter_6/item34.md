### Item34 : int 상수 대신 열거 타입을 사용하라 

###### 열거 타입
일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입. 클래스의 일종.
```java
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, 
    THURSDAY, FRIDAY, SATURDAY, SUNDAY;
}
```

###### 열거 타입 이전의 자바
열거 타입이 자바에 도입되기 전엔 개발자들은 `정수 열거 패턴`을 사용했다.
```java
public static int APPLE_FUJI = 0;
public static int APPLE_PIPPIN = 1;
public static int APPLE_GRANNY_SMITH = 2;

public static int ORANGE_NAVEL = 0;
public static int ORANGE_TEMPLE = 1;
public static int ORANGE_BLOOD = 2;
```
단점
- 타입 안전성이 보장되지 않음
- 표현력도 좋지 않음
- 같은 정수형 타입이므로 잘못된 값을 넘겨도 컴파일러는 문제를 일으키지 않음
- 다의어 같은 경우 개발자가 직접 접두어 같은 요소를 이용하여 이름 구분을 해야함
  (ex. PLANET_MERCURY, ELEMENT_MERCURY)
- 디버거를 확인하거나 특정 상수를 문자열로 출력하고 싶어도 정수값만 나오므로 확인이 쉽지가 않음

> 이러한 단점을 보완하여 사용할 수 있는 것이 '열거 타입'이다.

###### 자바의 열거 타입
자바의 열거 타입은 겉보기엔 다른 프로그래밍 언어에서 사용하는 열거 타입과 동일하게 보인다. 하지만 단순 정수 상수를 나타내는
다른 언어에서의 열거 타입과는 다르게 자바의 열거 타입은 하나의 클래스처럼 동작한다.
- 열거 타입은 하나의 클래스이다.
  - 그 내부에 정의되는 상수마다 자신의 인스턴스를 만들어 **public static final**로 외부에 노출한다.
- 생성자는 외부에서 호출이 사실상 불가능하므로 final 클래스와 동일하다.
  - 즉, `인스턴스 통제 클래스`이고 `싱글턴`의 일반형이라고 생각할 수 있다.
- 컴파일 타임 타입 안전성을 제공한다.
  - 특정 열거 타입을 매개변수로 받는 메서드를 정의했다면 런타임에는 (null이 아니라면) 해당 열거 타입이 담겨 있음을 보장할 수 있다.
  컴파일 타임에 이미 타입 체킹을 마치기 때문이다.
- 여러 개의 열거 타입 내부에 이름이 겹치는 상수가 있어도 문제가 없다.
- 열거 타입의 상수는 toString을 통해 출력을 하기 용이하다.
- 추가적으로 필드나 메서드를 선언할 수 있다.
- 인터페이스를 구현하여 사용할 수 있다.(ex. Comparable, Serializable 등)

###### 열거 타입에 필드 혹은 메서드를 추가하는 경우
열거 타입에 선언된 각 상수와 관련된 값들을 상수 내부에 저장하고 싶다면 필드를 얼마든지 추가할 수 있다.
필드를 추가하고 그 필드에 넣을 실제 값은 생성자를 통해 주입이 가능하다.
```java
public enum ErrorCode {
    USER_NOT_FOUND(HttpStatus.NOT_FOUND, "존재하지 않는 사용자입니다."), 
    AUTHORITY_NOT_FOUND(HttpStatus.NOT_FOUND, "존재하지 않는 권한입니다.");
    
    private final int statusCode;
    private final String message;
	
    ErrorCode(HttpStatus httpStatus, String message) {
        this.statusCode = httpStatus.value();
        this.message = message;
    }
}
```
열거 타입은 `values()` 라는 메서드를 기본적으로 제공한다. 이 메서드는 열거 타입 안에 정의된 상수들을 배열에 담아 반환해준다.
그리고 열거 타입의 `toString()`은 각 상수의 이름을 문자열로 변환하여 반환하므로 출력에 굉장히 용이하다. 그리고 `valueOf(String)`라는 메서드도 자동 생성되는데
보다시피 매개변수로 상수 이름을 넘기면 해당 상수를 반환해준다.

###### 열거 타입 내부 상수마다 다른 동작을 하는 경우
열거 타입 내부에 정의된 상수마다 정해진 동작을 해야 하는 경우 switch-case 문을 이용하여 코드를 작성할 순 있다.
하지만 그렇게 바람직하진 않으며 상수가 추가될 때마다 switch-case 문도 같이 수정해야 한다는 단점도 존재한다. 

그래서 열거 타입은 추상 메서드를 선언한 뒤에 각 상수가 자신의 목적에 맞게 해당 메서드를 구현할 수 있도록 하는 기능을 제공한다.
```java
public enum Operation {
    PLUS {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE {
        public double apply(double x, double y) {
            return x / y;
        }
    };
  
    public abstract double apply(double x, double y);
}
```
이렇게 각 상수마다 apply 메서드를 다르게 재정의하면 어떤 상수를 사용하냐에 따라 동작이 달라지게 된다.

###### 전략 열거 타입 패턴
위 소주제에서 언급한 것처럼 상수들은 각자 자신의 목적에 따라 메서드를 재정의할 수 있다. 하지만 만약 여러 상수에서 같은 구현을 
요구한다면 코드가 장황해지는 것을 막기가 힘들다. 이때 효율적으로 사용할 수 있는 방법이 있다.
```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);
  
    private final PayType payType;
  
    PayrollDay(PayType payType) { this.payType = payType; }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
  
    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };
  
        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
  
        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```
PayRollDay 열거 타입 안에 내부 열거 타입인 PayType을 선언하여 주말과 평일에 적용할 계산 형태를 정의할 수 있다.  
그리고 PayRollDay 열거 타입 안에 상수를 선언할 때는 자신에게 맞는 PayType을 선택할 수 있도록 함으로써 코드 중복은 없애고 
객체지향적인 특성을 살린 패턴이다.

## 핵심 정리
    정수 상수는 위험하다. 그에 비해 열거 타입은 안전하다. 필요하다면 필드 및 메서드를 사용하여
    상수의 동작을 정의할 수 있고, 상수마다 서로 다른 동작을 부여할 수 있다. 만약 특정 상수 그룹이 같은 
    동작을 해야 한다면 전략 열거 타입 패턴을 사용하자.