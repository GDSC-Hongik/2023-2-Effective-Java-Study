### Item33 : 타입 안전 이종 컨테이너를 고려하라

###### 타입 안전 이종 컨테이너(type safe heterogeneous container)
제네릭 클래스는 보통 필요한 만큼만 타입 매개변수를 가지도록 제한된다.  
Set 같은 경우 내부에 들어갈 원소를 나타내는 용도로 1개만 사용하고, Map 같은 경우
key 와 value를 나타낼 용도로 2개만 사용한다. 하지만 좀 더 유연하게 사용할 수 있는 방법이 없을까?

책에서는 이를 위해 `타입 안전 이종 컨테이너`를 제시한다.

`타입 안전 이종 컨테이너`
- 클래스 자체를 매개변수화 하는 것이 아니라 key를 매개변수화 한 뒤에 해당 클래스에 값을 넣고 뺄 때 해당 키를 제공하는 방식

말로만 설명하면 무슨 소린지 하나도 모르겠다. 다음 예시 코드를 보자.
```java
public class Favorites {
    
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance){
        favorites.put(Objects.requireNonNull(type), instance);
    } 
    
    public <T> T getFavorite(Class<T> type){
        return type.cast(favorites.get(type));
    }

    public static void main(String[] args) {
        Favorites favorites = new Favorites();
        favorites.putFavorite(String.class, "morning");
        favorites.putFavorite(Integer.class, 0xcafebabe);
        favorites.putFavorite(Class.class, Favorites.class);
    
        String favoriteString = favorites.getFavorite(String.class);
        Integer favoriteInteger = favorites.getFavorite(Integer.class);
        Class<?> favoriteClass = favorites.getFavorite(Class.class);
    
        System.out.printf("%s %x %s", favoriteString, favoriteInteger, favoriteClass.getName());
    }
}
```
위 코드가 가능한 이유는 .class 타입이 제네릭 클래스이기 때문이다. String.class 는 `Class<String>`, Integer.class는 `Class<Integer>`이고 이때 실제로 인수로 넘기는
String.class와 Integer.class는 타입 토큰이라고 칭한다.  
그래서 Favorites 클래스 내부에 Map을 두고 그곳에 매개변수화된 키와 그에 대응되는 value를 저장할 수 있다. 이 경우엔 일반적인 Map과 다르게
클라이언트가 원하는 타입의 객체를 마음대로 저장할 수 있다.

- favorites를 보면 key에 비한정적 와일드카드를 사용한 것을 알 수 있다. 이전 item에서 비한정적 와일드카드를 사용하면 null을 제외하곤 아무런 값도 넣을 수 없다고
했으니 위 코드는 잘못된 것 아닌가하는 생각이 들 수 있다. 하지만 위 코드의 경우 map 자체에 와일드카드가 적용된 것이 아니라 키에 적용된 것이므로 실제로 map안에 원소를 넣음에 있어
아무런 제약이 없다.
- 마찬가지로 favorites를 보면 Object 타입으로 value를 유지한다. 즉, key 와 value 간의 타입 관계를 유지하지 않는다는 뜻이다. 그럼 무슨 타입인지 어떻게 인지하는지 궁금할 수 있는데
이는 getFavorite 메서드에서 꺼내려고 하는 Class 객체를 받아서 cast 메서드를 사용함으로써 해결한다.  
cast 메서드는 형변환 연산자의 런타임 버전이라고 생각하면 된다.

###### 위 코드의 제약
1. 악의적인 클라이언트가 Class 객체를 제네릭이 아닌 로타입으로 넘기는 경우 타입 안전성이 깨진다.
   - 물론 컴파일러는 이에 대해 비검사 경고를 띄우긴 한다. 하지만 이보다 더 확실한 방지책이 있다.
    ```java
    public <T> void putFavorite(Class<T> type, T instance){
        favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }
    ```
   put 메서드에서 동적으로 타입 캐스팅 후 Object로 넣어주는 것이다. 이러면 만약 다른 타입이 들어오면 ClassCastException을 발생시키니 비교적 낫다고 볼 수 있다.
2. 위 클래스는 실체화 불가 타입에는 사용할 수 없다.
   - 즉, map 내부에 String 혹은 String[]은 저장할 수 있어도 `List<String>`은 저장할 수 없다는 뜻이다. 이유는 간단하다. `List<String>`용 Class 객체를 얻을 수 없기 때문이다.
   `List<String>`과 `List<Integer>`의 Class 객체는 모두 List.class로 동일하다. 즉, 같은 컬렉션이라면 어떤 타입 매개변수를 가지고 있는지에 상관없이 같은 Class 객체를 공유하므로 map에 
   저장했다가는 큰 문제가 발생할 것이다.

###### 한정적 타입 토큰
위 클래스에서는 비한정적 타입 토큰을 사용했다. 이전 item에서와 마찬가지로 여기서도 한정적 타입 토큰을 사용할 수 있다.
```java
public <T extends Annotation> 
    T getAnnotation(Class<T> annotationType);
```
이렇게 한정을 지어주면 매개변수로 들어오는 타입 토큰은 Annotation 혹은 그 하위 타입만 가능하다.

## 핵심 정리
    일반적인 제네릭 코드의 경우 사용할 수 있는 타입 매개변수 개수가 제한되어 있다. 
    하지만 클래스 자체를 매개변수화 하는 대신 key를 매개변수화 하여 값을 저장한다면 조금 더 유연한
    타입 안전 이종 컨테이너를 사용할 수 있다.

### Class 클래스
우리가 작성한 class 혹은 interface들의 메타데이터를 관리하는 객체라고 생각하면 된다. 꼭 클래스나 인터페이스가 아니더라도 그에 대응되는 Class 객체가 있다.  
예를 들어 원시 타입(ex. int, double, byte, boolean ...)이나 배열도 각각 그에 대응되는 Class가 있다.
```java
public static void main(String[] args) {
    int a[] = new int[5];
    double d[] = new double[6];
    int arr[][] = new int[2][2];
	
    System.out.println(int.class);
    System.out.println(double.class);
    System.out.println(a.getClass());
    System.out.println(d.getClass());
    System.out.println(arr.getClass());
}
```
```java
// 출력 결과
int
double
class [I
class [D
class [[I
```
보다시피 배열의 .getClass()도 가능하다. Class 클래스에 나와있는 설명에 의하면 배열의 경우 같은 원소 타입 그리고 같은 차원을 가지는 배열이라면
같은 Class를 가진다고 한다.

###### Class 객체 얻어오는 법
우선 위에서 봤듯이 특정 클래스의 Class 객체를 얻기 위해선 Object 클래스의 getClass() 메서드를 호출하면 된다. 그래서 예상할 수 있듯이 당연히 인스턴스를 생성한 뒤에 호출해야 한다.
```java
Car car = new Car();
Class clazz = car.getClass();
```
하지만 인스턴스 없이도 얻어올 수 있는 방법이 있다. 책에서도 나왔지만 forName() 메서드를 사용하는 것이다.
```java
Class clazz = Class.forName("hoyun06.Car")
```
forName 메서드를 이용해서 Class 객체를 얻을 때는 패키지를 포함한 클래스명을 직접 명시해야 한다.

###### Class 객체를 사용하여 할 수 있는 것들
위 방법을 통해서 특정 클래스에 대한 Class 객체를 얻었다면 할 수 있는 작업이 꽤나 많다.
```java
public class Car {

    private String name;
    private int numOfWheels;
    private int horsePower;
   
    public Car() {}
    public Car(String name, int numOfWheels, int horsePower) {
        this.name = name;
        this.numOfWheels = numOfWheels;
        this.horsePower = horsePower;
    }
   
    private void printName() {
        System.out.println("name = " + name);
    }
   
    public int getNumOfWheels() {
        return numOfWheels;
    }
   
    public int getHorsePower() {
        return horsePower;
    }
}
```
```java
public class Main {

    public static void main(String[] args) { // throws문 생략
        Car car = new Car("hello", 4, 2000);
    
        Class<? extends Car> clazz = car.getClass();
    
        Constructor<?>[] constructors = clazz.getDeclaredConstructors();
        Method[] methods = clazz.getDeclaredMethods();
        Field[] fields = clazz.getDeclaredFields();
    
        for (Constructor<?> constructor : constructors) {
           Class<?>[] types = constructor.getParameterTypes();
           for (Class<?> type : types) {
              System.out.print("type = " + type + "    ");
           }
           System.out.println();
        }
    
        for (Method method : methods) {
           System.out.println(method.getName());
        }
    
        for (Field field : fields) {
           System.out.println(field.getName());
        }
    }
}
```
출력 결과
```java
// 생성자의 매개변수 타입들을 모두 출력
type = class java.lang.String    type = int    type = int
        
// 선언된 메서드 모두 출력
printName
getNumOfWheels
getHorsePower
        
// 선언된 필드 모두 출력
name
numOfWheels
horsePower
```

###### 근데 이걸 어디다 써먹나요?
사실 Class 객체를 사용해서 private 필드와 메서드에 접근할 수 있다. 물론 위 코드처럼 그냥 메서드 이름만 알아오는 것만으로는 안되고 Class 객체가 제공하는 다른
메서드를 사용하면 된다.
```java
public class Main {

   public static void main(String[] args) {
      Car car = new Car("hello", 4, 2000);
      Class<? extends Car> clazz = car.getClass();
   
      Field horsePower = clazz.getDeclaredField("horsePower");
      horsePower.setAccessible(true); // 자바에서 접근 권한 체크하는 것을 무시하도록 한다.
      int privateField = (int)horsePower.get(car); // 인자로 넘긴 Car 객체에서 해당 필드를 가져온다.
      System.out.println("privateField = " + privateField);
	  
      Method privateMethod = clazz.getDeclaredMethod("printName");
      privateMethod.setAccessible(true);
      privateMethod.invoke(car); // 인자로 넘긴 Car 객체의 메서드를 invoke한다.
   }
}
```
출력 결과
```java
// private 필드 및 메서드에 접근할 수 있다!
privateField = 2000
name = hello
```

###### 우테코에서 테스트 코드 작성하며 사용했던 리플렉션
```java
@Test
public void test() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
     //given
     String input = "-1";
   
     //when
     RacingCarGameRunner runner = new RacingCarGameRunner();
     Method validateInput = RacingCarGameRunner.class
             .getDeclaredMethod("validateInput", String.class);
     validateInput.setAccessible(true);
   
     //then
     assertThrows(InvocationTargetException.class, () -> validateInput.invoke(runner, input));
}
```