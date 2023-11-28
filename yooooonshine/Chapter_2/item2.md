## 생성자에 매개변수가 많으면 빌더를 고려하라.
클래스  변수가 많을 경우, 이 많은 케이스에 대한 여러 생성자를 만들어 관리하는 것은, 보기 힘들며, 지저분해진다.

## 해결

### 점층적 생상자 패턴
이럴 경우 보통 점층적 생산자 패턴(기본 생성자 -> 매개변수 한개 생성자)등으로 많이 작성하나 이 또한 지저분하다.

### 자바빈즈 패턴
자바빈즈 패턴을 사용하여, 매개변수가 없는 생성자로 만든 후 세터로 매개변수 값을 설정하는 방법이다. 
다만 이렇게 하면 메서드를 여러번 호출해야 하며, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태가 된다. 
또한 이 경우는 클래스를 불변으로 만들 수 없고, setter가 열려 있기에 setter를 잘 못 사용하면 추후 유지보수가 어려울 수있다.

### 빌더 패턴
객체를 직접 만드는 대신에, 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다. 빌더 객체가 제공하는 세터 매서드들로 원하는 매개변수들만 선택하낟.
마지막으로 bulid메서드를 호출해 불변 객체를 얻는다.
```java
public class Abc {  
      
    private final int a;  
    private final int b;  
  
    public static class Builder {  
        // 필수 매개변수  
        private final int a;  
  
        //선택 매개변수, 기본값으로 초기화  
        private int b = 0;  
  
        public Builder(int a) {  
            this.a = a;  
        }  
          
        public Builder b(int val) {  
            b = val;  
            return this;        }  
   
        public Abc build() {  
            return new Abc(this);  
        }  
    }  
  
    private Abc(Builder builder) {  
        a = builder.a;  
        b = builder.b;  
    }  
}

public static void main(){
	Abc abc = new Abc.Builder(1).b(2).build();
}


```
이런 방식으로 사용한다.

참고로 Builder(1).b(2).bulid는 메소드 호출이 연결된다는 뜻으로 플루언트 API 혹은 메서드 연쇄라고 한다.

## 자바의 @Builder
자바 Lombok플러그인은 Builder를 지원해줘서, 위 코드처럼 지저분하게 작성하지 않아도 깔끔하게 다양한 종류의 생성자를 구성해준다.
``` java
public class Person { 
	private final String name; 
	private final int age; 
	private final int phone;
	 
	@Builder 
	private Person(String name, int age, int phone) {
		this.name = name;
		this.age = age;
		this.phone = phone;
	}
}


//사용
Person person = Person.builder()
.name("seungjin") 
.age(25) 
.build();
```
위 3개의 매개변수 name, age, phone에 대한 모든 가능한 경우의 생성자를 사용할 수 있다.


## 빌더의 단점
* 객체를 만들려면 빌더부터 만들어야 하므로, 빌더 생성 비용이 추가로 든다.
* 또한 코드가 장황해서 매개변수가 4개 이상인 정도는 되어야 값어치를 한다.


## 참고 불변과 불변식
불변(immutable)은 어떠한 변경도 허용하지 않는 것이며, 예로 String 객체는 한번 만들어지면 절대 값을 바꿀 수 없다.
불변식(invariant)는 프로그램이 실행되는 동안, 반드시 만족해야 하는 조건을 말한다. 예를 들어 period의 경우 start <= end 이며 역전되면 불변식이 깨지는 것.

## 참고 String객체는 왜 불변일까
불변 객체라 하면 한번 값이 할당 되면 내부 상태를 수정할 수 없는 것이다.
자바에서는 String은 String pool이라는 공간에 String을 포함시켜서, 값이 같은 String의 경우, String pool에 있는 값을 준다.
즉
```java
String a1 = "a";
String a2 = "a";
a1 == a2;
a1.equals(a2);
```
라고 하면 모두 true가 나오는 것이다.
만약 여기서 a2를 "b"로 바꾼다면 a2는 "b"라는 다른 스트링 객체를 가리키게 된다. 여기서 만약 mutable하다면 a2를 "b"로 바꿨어도 a1 == a2 이어야 하지만, immutable이므로 객체가 아예 바뀌는 것이다.