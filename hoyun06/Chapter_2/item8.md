### Item8 : finalizer와 cleaner 사용을 피하라

###### 자바의 소멸자와 C++의 소멸자
이번 아이템에서 다루는 자바의 소멸자와 C++의 소멸자를 헷갈리면 안된다.

`C++의 소멸자`
- C++ 생성자의 대척점으로서 객체와 관련된 자원을 회수하는 보편적인 방법이다.
- 비메모리 자원을 회수하는 방법이기도 하다(ex. 네트워크 커넥션, 입출력 stream 등).

`자바의 소멸자`
- 자바는 GC에 의해서 미사용 객체가 수거된다.
- 비메모리 자원을 회수할 때도 try-fianlly 혹은 try-with-resources를 사용한다.
- 따라서, 자바의 소멸자는 C++의 소멸자와는 사뭇 다르다.

###### 자바의 소멸자
1. finalizer
   - 예측할 수 없고, 상황에 따라 위험할 수 있어서 일반적으로 사용하지 않는다.
   - 오동작, 낮은 성능으로 인해 잘 사용되지 않는다.
   - 자바 9에서도 deprecated 되었다.

2. cleaner
   - finalizer 보다는 덜 위험하지만 일반적으로 불필요하다.
###### finalizer와 cleaner의 문제점
- **finalizer와 cleaner는 실행 시점을 보장할 수 없다** 
   - finalizer와 cleaner의 실행 시점은 전적으로 GC의 알고리즘에 달려있고 이는 GC의 구현에 따라 천차만별이다.  
   - finalizer는 다른 애플리케이션 스레드보다 우선 순위가 낮아서 실행 순서를 보장받지 못한다.
   - cleaner는 자신을 수행할 스레드에 대한 선택권은 가지고 있어서 조금은 낫다.  
     하지만 백그라운드에서 실행되며 GC의 통제하에 실행되므로 역시나 보장은 되지 않는다.

- **finalizer와 cleaner는 실행 여부조차도 보장할 수 없다**
  - 객체가 소멸되는 시점에 특정 로직이 실행되도록 구현해도 해당 로직이 실행되지 않을 수 있다.  
  - 따라서, **프로그램의 실행 흐름과 상관없이 상태를 영구적으로 수정하는 작업들은 절대로 finalizer와 cleaner를 사용하면 안된다.**  
    >   ex) DB의 명시적 락같은 경우 개발자가 직접 락 설정 및 해제를 해줘야 하는데 이를  finalizer나 cleaner에게 맡겼다가는 문제가 발생할 수 있다.  
  - System.gc 혹은 System.runFinalization 메소드는 finalizer와 cleaner의 실행 확률을 올려줄 순 있겠지만 여전히 보장하진 못한다.  

- **finalizer 동작 중에 발생하는 예외는 무시된다**
  - finalizer가 동작하는 도중에 발생하는 예외는 catch 되지 않는다. 예외가 발생해도 무시되며  
  그 이후에 꼭 실행되어야 할 작업들도 실행되지 않아서 객체가 불완전한 상태로 남게 된다.  
  당연히 그 불완전한 객체를 사용하게 되면 문제가 발생할 것은 분명하다.

- **finalizer와 cleaner는 성능상 문제도 있다**
  - 저자의 말에 따르면 finalizer나 cleaner를 사용하면 성능상 50배 정도 손해가 있었다고 한다.  
  대신 cleaner의 경우 `안전망` 형태로 사용하면 약 5배 정도의 성능상 손해 정도로 그친다고 한다.

- **finalizer attack에 취약하다**
  - `finalizer attack`은 super 클래스의 생성자 실행 혹은 직렬화 중간에 예외가 발생했을 때
  서브 클래스의 악의적인 finalize 메서드가 실행되도록 하는 공격을 뜻한다.
      ```java
      public class Pizza {
    
          private int size;
    
          public Pizza(int size) {
              if (size <= 0) {
                  this.size = -1000; // size가 음수면 -1000으로 세팅
                  throw new IllegalStateException();
              }
              this.size = size;
          }
    
          public void printPizza() {
              System.out.println("size of the pizza : " + size);
          }
      }
      ```
      ```java
      public class NyPizza extends Pizza {
          public static Pizza pizza;
    
          public NyPizza(int size) {
              super(size);
          }

          // GC에 의해 수거될 때 finalize 메서드 실행
          @Override
          protected void finalize() throws Throwable {
              super.finalize();
              pizza = this; // 주목할 부분
          }
      }
      ```
      ```java
      public class Main {
          public static void main(String[] args) throws InterruptedException {
              NyPizza nyPizza;
              try {
                  nyPizza = new NyPizza(-1); // 생성자에서 exception 발생
              } catch (IllegalStateException e) {
                  System.out.println("e = " + e);
                  System.gc(); // 여기서 nyPizza 참조 변수는 null이므로 GC의 수거 대상이다. 
                  Thread.sleep(5000); // GC가 동작할 수 있도록 5초간 thread sleep
                  NyPizza.pizza.printPizza();
              }
          }
      }
      ```
    - 위 코드를 보면 Pizza 클래스의 생성자에서는 size가 음수이면 임의로 -1000을 할당하고 예외를 던진다.
    - 하지만 클라이언트 코드인 main에서 System.gc를 호출하면 NyPizza 클래스의 finalize 메서드가 실행된다.
    - 이 finalize 메서드는 자신의 정적 멤버에 this를 할당하도록 해서 `일그러진 객체`의 참조를 살려둔다.
    - 이렇게 하면 원래는 사용하지도 못할 객체에 접근할 수 있을 뿐만 아니라 그 객체를 통해 메서드 호출까지 가능해진다.
    - 실제로 main 메서드에서 NyPizza 클래스의 정적 멤버인 `pizza`를 통해 `printPizza`메서드를 호출하는 것을 볼 수 있다.
    - `실제 호출 결과`
        ```
        e = java.lang.IllegalStateException
        size of the pizza : -1000
        ```
  - finalizer attack을 막고 싶다면 클래스를 final로 선언하면 된다. 하지만 이는 확장 자체를 막으므로..
  - 슈퍼 클래스에 아무것도 하지 않는 finalize 메서드를 오버라이딩하고 final로 선언하자.  
  그러면 서브 클래스에서 악의적인 finalize
  메서드를 정의할 수 없다!

###### 그렇다면 finalizer와 cleaner의 대안은?
- 파일 스트림이나 쓰레드등 객체와 관련된 자원을 회수해야 하는 경우 해당 클래스가 AutoClosable을 구현하도록 한다.  
- 그리고 객체가 더이상 사용되지 않는 시점에 close 메서드를 호출하도록 하면 된다(무조건적인 실행을 위해 try-with-resources 사용).  
+) close 메서드 실행 시에 일종의 flag 변수에 해당 객체가 더이상 사용 불가함을 표시한다.  
클래스 내부 다른 메서드에선 첫 줄에서 해당 flag 변수를 확인하는 로직을 추가하면 미사용 객체를 사용하는 불상사를 막을 수 있다.

###### 그럼 finalizer와 cleaner의 용도는 뭔가요?
1. 사용자가 close 메서드를 호출하지 않을 경우를 대비한 안전망 역할
    - 물론 finalizer와 cleaner는 성능도 떨어지고 안전성도 떨어진다. 그래도 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 용도로 사용할 수 있다.
   
2. 네이티브 피어의 회수
    - `네이티브 피어`란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 뜻한다.
    - `네이티브 메서드`란 native 키워드가 붙은 메서드를 뜻한다. 자바에서는 native 키워드를 붙인 메서드를 선언만 하고 메서드 바디는 구현하지 않는다.  
   대부분의 이런 native 메서드의 구현은 C 혹은 C++로 구현되어 있으므로 자바에서는 다음과 같이 해당 라이브러리를 로드해서 이미 구현된 메서드를 사용한다.
    ```java
    public class NativeExample {
        static {
            System.loadLibrary("NativeExample");
        }
    
        public native void nativeMethod();
    }
    ```
   - 이때 "NativeExample" 라이브러리는 C 혹은 C++와 같은 언어로 작성되어 있다.  
   이를 자바에서 사용할 수 있도록 해주는 중간 다리 역할을 JNI(Java Native Interface)가 해준다.
   - `네이티브 피어`는 `네이티브 메서드`에 의해서 초기화된다. 따라서, 네이티브 피어는 자바 객체가 아니고 그에 따라 GC의 수거 대상이 아니다.
   이때 이 네이티브 피어의 회수와 관련된 작업을 맡기 좋은 것이 finalizer와 cleaner다.

## 핵심 정리
    finalizer 혹은 cleaner는 안전망 역할 또는 네이티브 피어의 회수 목적으로만 사용하자. 
    물론, 이 경우에도 성능 저하와 불확실성을 염두해야 한다. 