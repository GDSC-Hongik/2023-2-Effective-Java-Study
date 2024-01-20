### Item1 : 생성자 대신 정적 팩터리 메서드를 고려하라

###### 클래스 인스턴스 생성 방법
1. public 생성자 이용
    ```java
   Car car = new Car();
    ```
2. 정적 팩터리 메서드 이용
    ```java
    public static Boolean valueOf(boolean b) {
        return b ? Boolean.TRUE : Boolean.FALSE;
    }
    ```
   정적 팩터리 메서드란?
    - 해당 클레스 인스턴스를 반환하는 단순한 static 메소드
###### 정적 팩터리 메서드가 생성자보다 더 좋은 점
1. 이름을 가질 수 있다
    - 생성자: 무조건 클래스 이름과 동일해야 하므로 같은 시그니처를 가지되 다른 역할을 수행하는 생성자를 만들 수 없음.  
      ex) 오류 발생하는 코드
      ```java
      public class Car {
          private String name;
          private String type;
 
          public Car(String name) {
              this.name = name;
          }
 
          public Car(String type) {
              this.type = type;
          }
      }
      ```
      매개변수의 순서를 다르게 한 생성자를 추가할 순 있으나 바람직하지 않은 방법.
    - 정적 팩터리 메서드: 이름을 가질 수 있으므로 이름을 통해 해당 메서드의 의도를 표시할 수 있음.   
      ex)
      ```java
      public class Car {
          private String name;
          private String type;
      
          private Car(String name, String type) {
              this.name = name;   
              this.type = type;     
          }
      
          public static Car ofName(String name) {
              return new Car(name, null);
          }
 
          public static Car ofType(String type) {
              return new Car(null, type);
          }
      }
      ```
2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다
    - 불변 클래스(클래스 내부의 값이 변하지 않는 클래스)는 인스턴스를 미리 만들어 놓고 재활용하는 식으로 불필요한 객체 생성을 막을 수 있음.  
      ex) 불변 클래스는 인스턴스를 미리 생성하여 캐싱한 뒤에 지속적으로 재사용 가능.
      ```java
      public class Car {
          private static final Car INSTANCE= new Car();
        
          private Car() {}
 
          public static Car getInstance() {
              return INSTANCE;
          }
      }
      ```
      이렇게 반복되는 요청에 같은 인스턴스를 반환하는 클래스를 '인스턴스 통제(Instance controlled) 클래스'라고 함
        - 인스턴스를 통제함으로써 '싱글턴'과 '인스턴스화 불가'를 달성할 수 있음.
    - 객체 생성 작업이 무거운 클래스의 경우에는 인스턴스를 굳이 생성할 필요가 없는 정적 팩터리 메서드가 더 유용함.
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다
    - 생성자: 반환 타입이 해당 클래스 타입로 항상 정해져 있음.
    - 정적 팩터리 메서드: 반환 타입에 제약이 줄어듦.  
      반환 타입을 인터페이스로 설정하고 실제로는 해당 인터페이스의 구현체 인스턴스 반환 가능.
        - api 클라이언트: 정적 팩터리 메서드를 호출하여 인터페이스 레퍼런스만을 이용하여 실제 인스턴스 사용 가능.  
          자신이 사용하는 인스턴스가 어떤 구현체인지 알 수 없고 알 필요도 없음.
        - api 제공자: 구현체들에 대한 세부 사항을 공개할 필요없이 반환형을 인터페이스로 설정함.  
          이를 통해 인터페이스 기반 프레임워크 방식으로 설계 가능
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다
      ```java
      public interface Phone {
          void call();
          void sendMessage();

          static Phone create(String type) {
              if (type.equals("mobile")) {
                  return new MobilePhone();
              } else if (type.equals("home")){
                  return new HomePhone();
              }
              return null;
          }
     }
     ``` 
     ```java
    public class Main {
        public static void main(String[] args) {
            String type = "mobile";
            Phone mobliePhone = Phone.create(type);

            type = "home";
            Phone homePhone = Phone.create(type);
        }
    }
     ```
    - java 8부터 인터페이스 내부에 정적 메서드를 선언할 수 있으므로 매개변수에 따라 매번 다른 구현체 인스턴스를 반환 가능.
    - 필요에 따라 MobliePhone을 제거하고 새로운 Phone 구현체를 추가하더라도 api 클라이언트는 이 사실을 알 수 없고 알 필요도 없음.
5. 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클레스가 존재하지 않아도 된다
    - 결국 큰 흐름은 3번과 이어진다고 생각한다.
    - 저자는 이 부분을 JDBC를 통해서 설명하는데 나는 내가 이해한 방식대로 설명을 하고자 한다.
    ```java
    public interface Pizza {
        // 구현이 필요한 메서드들 정의
    }
    ```
    ```java
    public class PizzaFactory {
        private static final Map<String, Pizza> pizzaMap = new HashMap<>();
        
        static {
            // 이 내부의 구현은 Pizza 인터페이스를 구현하는 새로운 피자 클래스가 추가될 때마다 수정하면 된다.
            // 만약 이 부분이 비어있어도 아래의 getPizza 메서드를 작성함에 있어 문제가 없다.
            pizzaMap.put("newyork", new NewyorkPizza());
            pizzaMap.put("chicago", new ChicagoPizza());
            pizzaMap.put("philadelphia", new PhiladelphiaPizza());
        }
        
        // 이 메소드를 작성하는 시점에는 Pizza 인터페이스의 
        // 구현 클래스가 어떤 것인지 몰라도 되고 실제로 존재하지 않아도 문제 없다. 
        public static Pizza getPizza(String pizzaName) {
            Pizza pizza = pizzaMap.get(pizzaName);
            if (pizza == null) {
                throw new IllegalStateException("존재하지 않는 피자입니다.");
            }
            return pizza;
        }
    }
    ```
   위 PizzaFactory 클래스의 getPizza 메서드가 정적 팩터리 메서드의 역할을 수행한다.  
   어차피 반환 타입은 interface인 Pizza이므로 그 구현 클래스의 존재 여부와 상관없이 getPizza 메서드는
   구현할 수 있다.

###### 정적 팩터리 메서드의 단점
1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다
    - 서브 클래스의 인스턴스를 만들기 위해서는 슈퍼 클래스의 생성자도 결국 호출이 되어야 하는데  
      슈퍼 클래스에서 priavte 생성자와 정적 팩터리 메서드만 지원한다면 호출이 불가능 = 상속 불가능.
    - 하지만 아래 2가지를 적용하기 위해서는 이 제약 조건이 필수이므로 오히려 장점이 될 수도 있음.
        - 상속 대신 *컴포지션을 사용
        - 불변 클래스를 만들기  
          `컴포지션`: 기존 클래스를 확장하는 대신에 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하는 방식
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다
    - 생성자와는 다르게 정적 팩터리 메서드는 api 명세에 나와있지 않음.  
      정적 팩터리 메서드만 제공하는 클래스의 인스턴스를 생성할 때에는 사용자가 직접 생성 방법을 찾아야함.

###### 흔히 사용하는 정적 팩터리 메서드 명명법
- from: **매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드**
```java
Date d = Date.from(instant);
```
- of: **여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드**
```java
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
```
- instance 혹은 getInstance: **매개변수로 명시한 인스턴스를 반환, 같은 인스턴스임을 보장하진 않는다.**
```java
StackWalker luke = StackWalker.getInstance(options);
```
- create 혹은 newInstance: **instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스 생성을 보장**
```java
Object newArray = Array.newInstance(classObject, arrayLen);
```
- getType: **getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용**
```java
FileStore fs = Files.getFileStore(path); ("Type"은 팩터리 메서드가 반환할 객체의 타입)
```
- newType: **newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용**
```java
BufferedReader br = Files.newBufferedReader(path);
```
- type: **getType과 newType의 간결한 버전**
```java
List<Complaint> litany = Collections.list(legacyLitany);
```
## 핵심 정리
    public 생성자와 정적 팩터리 메서드는 각자의 쓰임새가 있으니 상황에 맞게 잘 사용하면 된다.