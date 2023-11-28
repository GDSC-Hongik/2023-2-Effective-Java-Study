### Item4 : 인스턴스화를 막으려거든 private 생성자를 사용하라

###### 정적 멤버만 담은 유틸리티 클래스
- 정의: static 필드와 static 메서드만 포함하고 있는 클래스
- 예시
    - java.lang.Math, java.util.Arrays  
      (기본 타입이나 배열 관련된 메서드들을 제공하는 기능)
    - java.util.Collections  
      (ex. Collections.list(), Collections.singletonList(), Collections.emptyMap()...)

###### 유틸리티 클래스의 목적
저자는 이런 유틸리티 클래스가 객체지향적인 방법은 아니라고 했다.
그럼에도 사용하는 이유를 굳이 꼽자면..
- helper의 성격을 갖는 유틸성 클래스 ex) 현재 진행 중인 프로젝트에서도 CookieUtil 이라는 정적 유틸리티 클래스 사용
- 유틸리티 클래스는 모든 멤버가 다 static 선언이 되어 있으므로 인스턴스를 굳이 생성하지 않아도 public 멤버에 접근 가능하다.

사실 유틸리티 클래스를 설계한 것부터가 해당 클래스의 인스턴스를 생성하지 않기 위함이라고 해도 무방하다.

>#### 따라서 유틸리티 클래스는 인스턴스화를 막아줘야 한다
###### 잘못된 방법
1. 생성자를 명시하지 않는다
    - 사용자가 명시적으로 생성자를 선언하지 않으면 컴파일러에 의해 매개변수가 없는 기본 생성자가 생성되므로 인스턴스화가 가능해진다
    - 클라이언트는 이 기본 생성자가 자동 생성된 것임을 알지 못하므로 해당 생성자를 사용할 여지가 생긴다
2. 추상 클래스로 선언한다
    - 추상 클래스는 원칙적으로는 인스턴스화가 될 수 없는 것이 맞다.
    - 하지만 추상 클래스를 상속하는 서브 클래스에서 모든 추상 메서드를 오버라이딩 하면 서브 클래스의 인스턴스를 생성할 수 있게 되고
      이 말은 곧 추상 클래스도 인스턴스화가 될 수 있음을 뜻한다.(서브 클래스의 생성자가 호출되면 자연스레 슈퍼 클래스의 생성자도 호출이 되므로)

###### 옳은 방법
- 유틸리티 클래스의 인스턴스화를 막는 방법은 굉장히 간단하다.
- 바로 `private 생성자`를 사용하는 것이다.
    ```java
    public class TestClass {
        // private 생성자를 통한 인스턴스화 방지
        private TestClass() {
            throw new AssertionError();
        }
    }
    ```
- 생성자 안에 있는 throw문은 필수적이지 않으나 클래스 내부에서의 인스턴스 생성까지 막기 위한 방법이다.
- private 생성자는 인스턴스화를 막는 방법으로 사용되면서도 상속을 막는 부수적인 효과도 가지고 있다.

###### 이외에 사용할 수 있는 방법
- 추상 클래스 + private 생성자 사용하기
- final 클래스 + private 생성자 사용하기 등

**핵심은 `private 생성자 사용하기`**

### 추가 내용
###### static 이란?
- 자바 내에서 정적 멤버/클래스임을 나타내는 키워드
- static 키워드를 붙인 멤버는 각 인스턴스에 속하는 것이 아니라 클레스에 소속된다.
- static 멤버는 클래스 로딩 시점에 heap 영역이 아닌 static 영역에 할당된다.  
  따라서 해당 클래스의 모든 인스턴스가 공유할 수 있지만 GC의 관리 대상이 아니므로 사용에 주의해야 한다.

###### 우선 inner class란?
- outer class의 멤버 필드 위치에 선언된 class.
```java
    public class OuterClass { // 외부 클래스(outer class)
        
        private String outerClassString;
        
        public class InnerClass { // 내부 클래스(inner class)
            
        }
    }
```
한 클래스 내부에 또 다른 클래스를 정의하는 경우 각각을 외부 / 내부 클래스라고 칭한다.

###### 그럼 static class란?
- inner class에 static 키워드가 붙은 class.
```java
    public class OuterClass { // 외부 클래스(outer class)
        
        public static class StaticClass { // 내부 클래스(static inner class)
            
        }
    }
```
###### 각 클래스의 특징은?
**(Non-static) inner class**
- inner class는 outer class에 대한 참조(외부 참조)를 내부적으로 가지는데 이는 숨겨져있다.
```java
// 원본 java 파일
public class TestClass {
    public TestClass() {}

    public class InnerClass {
        public InnerClass() {} // 생성자의 매개변수 공란
    }
}
```
```java
// 위 java 파일의 .class 파일에서 InnerClass 부분만 디컴파일한 것
public class TestClass$InnerClass {
    public TestClass$InnerClass(TestClass this$0) { // 위 java 파일에선 보지 못한 TestClass에 대한 숨겨진 참조
        this.this$0 = this$0;
    }
}
```
- 인스턴스를 생성하기 위해서는 outer class의 인스턴스가 먼저 생성되어야 한다.

```java
public class Main {
    public static void main(String[] args) {
        // outer class인 TestClass 인스턴스 먼저 생성
        TestClass testClass = new TestClass();
        // TestClass 인스턴스를 이용하여 InnerClass 인스턴스 생성
        TestClass.InnerClass innerClass = testClass.new InnerClass();
    }
}
```
- 외부 참조를 가지므로 inner class에서 outer class의 필드나 메서드에 접근할 수 있다.
```java
public class TestClass {
    // TestClass 내부 private 필드
    private String testClassField;
    
    public TestClass() {}
    
    public class InnerClass {
        public InnerClass() {}
        
        // inner class에서는 outer class의 
        // private 필드까지도 접근 가능하다.
        public void print() {
            System.out.println("testClassField = " + testClassField);
        }
    }
}
```
- 그래서 오로지 inner class 인스턴스만 생성하고 싶더라도 먼저 outer class 인스턴스를 생성해야만 한다.


**static class**
- static class는 outer class에 대한 참조(외부 참조)를 가지지 않는다.
```java
// 원본 java 파일
public class TestClass {
    public TestClass() {}

    public static class StaticClass {
        public StaticClass() {} // 생성자의 매개변수 공란
    }
}
```
```java
// 위 java 파일의 .class 파일에서 StaticClass 부분만 디컴파일한 것
public class TestClass$StaticClass {
    public TestClass$StaticClass() { // 숨겨진 참조가 없다 = outer class에 대한 참조를 필요로 하지 않는다.
    }
}
```
- outer class의 인스턴스가 없어도 인스턴스를 생성할 수 있다.
```java
public class Main {
    public static void main(String[] args) {
        // StaticClass 인스턴스를 바로 생성할 수 있다.
        TestClass.StaticClass staticClass = new TestClass.StaticClass();
    }
}
```
- 외부 참조를 가지지 않으므로 StaticClass에서는 static이 붙지 않은 outer class의 필드나 메서드에 접근할 수 없다.
```java
public class TestClass {

    // non-static field
    private String field;
    // static field
    private static String staticField;

    public TestClass() {}
    
    public static class StaticClass {
        public StaticClass() {}

        public void print() {
            // non-static 필드는 접근 불가. 컴파일 에러
            System.out.println("field = " + field);
            // static 필드는 접근 가능하다.
            System.out.println("staticField = " + staticField);
        }
    }
}
```
###### Item24에서는 static class를 사용하라던데..
맞는 말이다. inner class는 외부 참조 덕분에 outer class의 멤버에 접근할 수 있지만 그 외부 참조 때문에 메모리 누수 위험이 생긴다.  
위에서 inner class는 outer class 인스턴스가 생성된 이후에야 생성될 수 있다고 했다.
inner class 인스턴스 생성 목적으로 사용된 outer class 인스턴스는 그 목적을 달성했으므로 GC에 의해 할당받은 메모리를 반환하는 것이 바람직하다.  
하지만 inner class는 내부적으로 외부 참조를 가지고 있기 때문에 해당 outer class 인스턴스는 제거되지 않고 계속 메모리 공간을 차지하게 된다.
이런 방식으로 inner class 인스턴스를 지속적으로 생성하다 보면 메모리 누수가 생기는 현상이 발생한다.

> 만약 outer class의 멤버에 굳이 접근할 필요가 없다면 inner class는 static으로 선언하는 것이 좋다.