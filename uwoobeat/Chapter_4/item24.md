# Item 24. 멤버 클래스는 되도록 static으로 만들라

정적 멤버 클래스는 외부 클래스와 함께 쓰이는 헬퍼 클래스로 사용된다. 바깥 클래스의 private 멤버에 접근할 수 있다. 비정적 멤버 클래스는 static 키워드가 없어진 것이고 '이 클래스(외부)의 인스턴스'에 속한다는 의미를 가지게 된다. 이때 this와 정규화된 this를 구별해서 사용해야 한다. 비정적 멤버(내부 클래스)에서 this를 사용하는 경우 자기 자신(내부)를 의미하지만, 외부 클래스의 this를 사용하고 싶다면 정규화된 this를 사용해야 한다. 

```java
public class Outer {

    private final String name;

    public Outer(String name) {
        this.name = name;
    }

    private class Inner {
        public final String name;

        public Inner(String name) {
            this.name = name;
        }

        public void print() {
            System.out.println("Outer.this.name = " + Outer.this.name);
            System.out.println("this.name = " + this.name);
        }
    }
}
``````