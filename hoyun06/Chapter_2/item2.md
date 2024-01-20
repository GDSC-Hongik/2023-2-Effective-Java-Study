### Item2 : 생성자에 매개변수가 많다면 빌더를 고려하라

###### 정적 팩터리 메서드와 생성자가 공통적으로 가지는 제약
- 선택적 매개변수가 많은 경우 적절히 대응하기가 어렵다.

###### 선택적 매개변수가 많은 경우에 생성자를 이용하는 방법
`점층적 생성자 패턴`
```java
public class NutritionFacts {

    private final int servingSize; //  필수

    private final int servings; //  필수

    private final int calories; // 선택

    private final int fat; // 선택

    private final int sodium; // 선택

    private final int carbohydrate; // 선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```
- 필수 매개변수만 받는 생성자부터 해서 선택 매개변수를 1개,2개... 전부 받는 생성자를 모두 선언하는 방식이다.
- 해당 생성자를 이용하는 클라이언트 코드 자체는 짧다.
- 하지만 매개변수가 여러 개 있는 경우 사용자가 실수로 순서를 바꿔 전달해도 컴파일러는 오류를 발생시키지 않는다.
- 그리고 여러 개의 매개변수를 한 번에 같이 넘기기 때문에 각 매개변수의 의미를 파악하는 것도 어려워진다.

###### 선택적 매개변수가 많은 경우에 자바 빈즈 패턴을 이용하는 방법
`자바 빈즈 패턴`
```java
public class NutritionFacts {

    private int servingSize = -1; // 필수; 기본 값 없음.

    private int servings = -1; // 필수; 기본 값 없음.

    private int calories = 0;

    private int fat = 0;

    private int sodium = 0;

    private int carbohydrate = 0;

    public void setServingSize(int val) {
        this.servingSize = val;
    }

    public void setServings(int val) {
        this.servings = val;
    }

    public void setCalories(int val) {
        this.calories = val;
    }

    public void setFat(int val) {
        this.fat = val;
    }

    public void setSodium(int val) {
        this.sodium = val;
    }

    public void setCarbohydrate(int val) {
        this.carbohydrate = val;
    }
}
```
- 기본 생성자를 이용하여 인스턴스를 먼저 생성한 뒤에 setter 메서드를 이용하여 각 필드값을 세팅하는 방법이다.
- 생성자를 하나만 선언해도 되고 `점층적 생성자 패턴`을 사용할 때 보다 코드 가독성이 훨씬 올라간다.
- 하지만 `점층적 생성자 패턴` 이용한 방법과는 다르게 하나의 인스턴스를 생성하기 위해 여러 개의 메서드를 호출해야 한다.
- `점층적 생성자 패턴`에서는 생성자를 호출하면 필수 필드와 더불어 선택적 필드들까지도 완벽하게 초기화가 되는데  
  `자바 빈즈 패턴`에서는 각 필드에 대한 setter 메서드의 호출이 강제되지 않기 때문에 값을 설정하기 위해서는 직접 setter 메서드를 호출하므로
  자칫 누락될 위험이 있다.
- 혹여 초기화가 필요한 필드에 대해서 setter 메서드를 호출하지 않게 되면 `일관성`이 무너진 상태가 되고 그렇게 `일관성`이 무너진 인스턴스를
  코드 전반부에서 사용하게 되면 오류가 발생할 확률이 높다.
- 인스턴스 생성 후에 setter로 값을 설정해줘야 하므로 각 필드에 final 키워드를 사용할 수 없다. 그래서 클래스를 불변으로 만들 수 없다.
- 그래서 굳이 setter를 사용하고 싶다면...
    - 필수 매개변수를 받는 생성자만 선언하여 기본 생성자를 통한 생성을 방지하고
    - 선택 매개변수들은 필드 자체에 디폴트 값을 설정해주고
    - setter 메서드를 사용하여 값을 설정할 수 있도록 한다.

###### 그럼 어떤 방법을 써야 하나요..
> 빌더 패턴을 사용하면 된다

- 빌더 패턴은 `점층적 생성자 패턴`의 안정성과 `자바 빈즈 패턴`의 가독성을 모두 갖췄다.
- 클라이언트는 원하는 객체를 직접 생성하지 않고 `빌더`객체를 사용하여 필수 필드를 먼저 초기화를 하고  
  `빌더`객체에서 제공하는 일종의 setter 메서드를 사용하여 선택 매개변수값을 설정할 수 있다.
- 마지막으로 build 메서드를 실행하여 원하는 객체를 반환할 수 있다.
- 주로 빌더 클래스는 우리가 생성하고자 하는 클래스의 내부에 static class로 선언한다.

`빌더 패턴`
```java
public class NutritionFacts {

    private int servingSize; // 필수; 기본 값 없음.

    private int servings; // 필수; 기본 값 없음.

    private int calories;

    private int fat;

    private int sodium;

    private int carbohydrate;
    
    public static class Builder {
        // 필수
        private final int servingSize;
        private final int servings;
        // 선택
        private int calories;
        private int fat;
        private int sodium;
        private int carbohydrate;
        
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        
        public Builder calories(int val) {
            calories = val;
            return this;
        }  
        public Builder fat(int val) {
            fat = val;
            return this;
        } 
        public Builder sodium(int val) {
            sodium = val;
            return this;
        } 
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
    
    public NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```
Builder에서 제공하는 setter 메서드들은 반환형이 Builder이므로 method chaining이 가능하다.  
매개변수에 대한 검증은 Builder 내부 생성자, setter 메서드에서 진행하는 것이 좋다.  
<클라이언트 코드에서 실제로 Builder를 이용하여 객체를 생성하는 예시>
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
        .calories(100).sodium(35).carbohydrte(27).build();
```
###### 빌더 패턴은 계층적으로 설계된 클래스와 함께 사용하기 좋다
```java

public abstract class Pizza {

    public enum Topping {MUSHROOM, HAM, ONION, PEPPER, SAUSAGE}

    final Set<Topping> toppings;

    // 재귀적 타입 한정 이용
    public abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        // self()를 통해 하위 클래스에서 
        // 형변환 없이 method chaning 가능 
        public T addTopping(Topping topping) {
            toppings.add(topping);
            return self();
        }

        abstract Pizza build();

        protected abstract T self();
    }

    public Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
```
```java
public class NyPizza extends Pizza{

    public enum Size {SMALL, MEDIUM, LARGE}
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {

        private final Size size;

        public Builder(Size size) {
            this.size = size;
        }

        // 각 하위 클래스에서는 자신 타입의 인스턴스를 반환하도록 구현한다.
        @Override
        Pizza build() {
            return new NyPizza(this);
        }

        // this를 return 하도록 구현하여서 따로 형변환이 필요없다.
        @Override
        protected Builder self() {
            return this;
        }
    }

    public NyPizza(Builder builder) {
        super(builder);
        this.size = builder.size;
    }
}
```
```java
public class Calzone extends Pizza{
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {

        private boolean sauceInside = false;
        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

      // 각 하위 클래스에서는 자신 타입의 인스턴스를 반환하도록 구현한다.
        @Override
        Pizza build() {
            return new Calzone(this);
        }

      // this를 return 하도록 구현하여서 따로 형변환이 필요없다.
        @Override
        protected Builder self() {
            return this;
        }
    }

    public Calzone(Builder builder) {
        super(builder);
        this.sauceInside = builder.sauceInside;
    }
}
```
코드가 굉장히 장황하지만 중요 포인트만 보자면..
- `재귀적 타입 한정`을 이용하여 NyPizza와 Calzone 각각의 Builder는 self 메서드로 자기 자신들을 반환하도록 한다.
  (method chaining 용이함)
- NyPizza와 Calzone Builder의 build 메서드는 각각 NyPizza와 Calzone 인스턴스를 반환한다.
  (하위 타입 반환)
- 바로 위에서 언급한 것처럼 슈퍼 클래스에서 지정한 반환 타입이 아닌 그 하위 타입을 반환하는 것을 `공변 반환 타이핑`이라고 한다.

###### 위와 같은 빌더 패턴의 장점
- 가변인수 매개변수를 여러 개 사용할 수 있다.
    - 보통의 생성자라면 여러 개의 매개변수중 가장 마지막 매개변수에만 가변인수를 사용할 수 있다.
    - 하지만 빌더 패턴에서는 메서드를 여러 개로 쪼개면 각 메서드마다 가변인수를 사용할 수 있다.

###### 위와 같은 빌더 패턴의 단점
- 당연하지만 우선 빌더를 구현해야 한다.
- `점층적 생성자 패턴`보다 코드가 장황해진다.

## 핵심 정리
    특정 인스턴스를 생성할 때 넘겨줘야할 매개변수가 많다면(특히나 같은 타입이) 빌더 패턴을 고려하자.
    빌더 패턴은 점층적 생성자 패턴처럼 안전하면서 자바 빈즈처럼 가독성이 좋다.