### Item24 : 멤버 클래스는 되도록 static으로 만들라

###### 중첩 클래스
한 클래스 안에 정의된 클래스.   
중첩 클래스는 반드시 그 바깥 클래스에서만 사용되어야 하며 만약 다른 곳에서도 사용되어야 한다면
중첩 클래스가 아닌 탑레벨 클래스로 설계해야 한다.   
중첩 클래스의 종류로는
- 정적 멤버 클래스
- 비정적 멤버 클래스
- 익명 클래스
- 지역 클래스

여기서 정적 멤버 클래스를 제외하곤 모두 내부 클래스(inner class)다.

###### 정적 멤버 클래스
```java
public class TestClass {
    private String outerString;
    
    static class InnerClass {
        private String innerString;
    
        public void print() {
            TestClass testClass = new TestClass();
            System.out.println(testClass.outerString);
        }
    }
}
```
- 다른 클래스 안에 선언된다.
- 바깥 클래스의 private 필드에도 접근 가능하다.
- 바깥 클래스의 다른 정적 멤버와 동일한 접근 규칙을 적용받는다.

###### 비정적 멤버 클래스
```java
public class TestClass {
    private String outerString;
    
    class InnerClass {
        private String innerString;
    
        public void print() {
            System.out.println(outerString);
        }
    }
}
```
- 비정적 멤버 클래스의 인스턴스는 암묵적으로 바깥 클래스의 인스턴스와 연결된다.
- 내부 클래스 인스턴스 메서드에서 정규화된 this로 바깥 클래스 멤버 접근이 가능하다.
  - **정규화된 this**: '클래스명.this'로 바깥 클래스 이름을 명시하여 사용하는 this.

따라서 내부 클래스가 외부 클래스의 멤버에 직접적으로 접근할 필요성이 없다면 내부 클래스는 
정적 멤버 클래스로 선언하는 것이 좋다. 만약 비정적 멤버 클래스로 선언하면 외부 클래스 인스턴스 없이는 내부 클래스 인스턴스를 생성할 수 
없기 때문이다. 이와 관련된 추가 정보는 [item4](https://github.com/GDSC-Hongik/2023-2-Effective-Java-Study/blob/main/hoyun06/Chapter_2/item4.md)의 추가 발표 자료를 보자.

짧게 요약하자면 비정적 멤버 클래스는 외부 클래스와 내부 참조가 생기며 이는 자원 소모를 야기하고 이 내부 참조를
제대로 발견하지 못한다면 메모리 누수까지 이어질 수 있다.

###### 익명 클래스
```java

public class TestClass {
  
    public void test() {
        InnerInterface test = new InnerInterface() {
            @Override
            public void print() {
                System.out.println("test print");
            }
        };
        
        test.print();
    }
  
    interface InnerInterface {
        void print();
    }
}
```
- 이름이 없다.
- 코드 상에서 어디서나 선언되는 동시에 생성된다.
- 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다.
- instanceof와 같이 클래스 이름이 필요한 작업은 사용 불가하다.
- 해당 익명 클래스가 상속받은 멤버만 사용 가능하다(임의로 추가하는 것이 불가능하다).
- 코드 중간에 주로 등장하므로 길이가 길어지면 가독성이 떨어진다.

요즘엔 익명 클래스의 역할을 **람다**가 맡아서 해결하고 있다.

###### 지역 클래스
- 지역 변수를 선언할 수 있는 곳이라면 어디든 선언 가능.
- 유효 범위도 지역 변수와 동일하다.
- 위에서 설명한 중첩 클래스와 하나씩 공통점이 있다.
  - 멤버 클래스와 마찬가지로 이름을 가진다.
  - 익명 클래스와 마찬가지로 비정적 문맥에서만 바깥 클래스 멤버에 접근 가능하다.

## 핵심 정리
    한 클래스 내부에서만 사용하되 여러 메서드에서 사용되면 멤버 클래스로 선언하자. 
    이때 외부 클래스 참조가 필요한 경우 비정적, 필요없다면 정적으로 선언하자.
    한 메서드 내부에서만 쓰이면서 길이가 짧다면 익명 클래스, 그렇지 않다면 지역 클래스로 선언하자.