# Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라

정적 팩토리 메서드와 생성자가 공통적으로 가지는 제약사항이 하나 있다.  
선택적 매개변수가 많을 때 적절하게 대응하기 어렵다는 점이다.  
```java
public class NutritionFacts {
    // 필수
    private final int servingSize;
    private final int servings;
    
    // 선택
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
}
```
위와 같은 클래스가 있다고 하자.  
이 클래스의 인스턴스를 만드는 방법으로 우선 두 가지를 떠올릴 수 있다.  
점층적 생성자 패턴과 자바 빈즈 패턴이다.  

### 점층적 생성자 패턴
점층적 생성자 패턴이란 받을 매개변수의 종류에 따라 생성자를 다르게 만드는 것이다.  
이 패턴의 단점은 사용자가 설정하고 싶지 않은 값까지 매개변수에 있으므로 어쩔 수 없이 값을 설정해야한다는 점이다.
```java
public class NutritionFacts {
    // 필수
    private final int servingSize;
    private final int servings;

    // 선택
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings, int calories) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
    }
    
    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
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

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 0, 10);  
```
**단점**  
- 각각이 무슨 값인지 눈에 잘 들어오지도 않아 의미가 불분명.
- 매개변수가 몇 개인지도 주의해서 세어봐야 한다.
- 억지로 값을 넣어줘야 하는 경우가 발생하므로 유연성이 떨어짐.

### 자바 빈즈 패턴
점층적 생성자 패턴은 선택 매개변수가 많아지니 효율적이지 못하다.  
자바 빈즈 패턴은 기본 생성자로 객체를 만들고 setter들로 매개변수들을 설정한다.
```java
public class NutritionFacts {
    // 필수
    private int servingSize;
    private int servings;

    // 선택
    private int calories;
    private int fat;
    private int sodium;
    private int carbohydrate;

    public NutritionFacts() {}

    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }
}
```
이렇게 하면 인스턴스를 쉽게 만들고 코드의 가독성도 좋아진다.  
fat의 값을 설정하기 위해 억지로 calories까지 초기화하지 않아도 된다.
```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingsSize(240);
cocaCola.setServings(8);
cocaCola.setFat(10);
```
**단점**  
- 설정할 매개변수의 개수만큼 메서드를 호출해야 함.
- 마지막 setter가 끝날 때까지 객체는 일관성이 없는 상태에 놓이게 됨   
    => 클래스를 불변으로 만들 수도 없어서 스레드 안전성을 얻을 수 없다.  

## 빌더 패턴
이 모든 문제를 해결하기 위해 빌더 패턴을 사용할 수 있다.  

빌더 패턴은 아래와 같은 순서로 작동한다.
1. 필수 매개변수만으로 생성자를 호출해 빌더 객체 받고
2. 빌더 객체의 setter들로 선택 매개변수를 설정하고
3. build()로 객체를 반환

위의 NutritionFacts class를 빌더로 바꿔보자.

```java
public class NutritionFacts {
    // 필수
    private final int servingSize;
    private final int servings;

    // 선택
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        private final int servingSize;
        private final int servings;
        private int calories;
        private int fat;
        private int sodium;
        private int carbohydrate;

        // 1번
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        // 2번
        public Builder calories(int calories) {
            this.calories = calories;
            return this;
        }

        public Builder fat(int fat) {
            this.fat = fat;
            return this;
        }

        public Builder sodium(int sodium) {
            this.sodium = sodium;
            return this;
        }

        public Builder carbohydrate(int carbohydrate) {
            this.carbohydrate = carbohydrate;
            return this;
        }

        // 3번
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.calories = builder.calories;
        this.fat = builder.fat;
        this.sodium = builder.sodium;
        this.carbohydrate = builder.carbohydrate;
    }
}
```

빌더의 세터들은 자기 자신(Builder)을 반환하기 때문에 선택 매개변수들을 연쇄적으로 설정할 수 있다.  
위의 코드에서 ```return this;```로 반환하는 것을 확인할 수 있다.  
이걸 fluentAPI 혹은 메서드 연쇄라고 부른다고 한다.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                                .fat(0)
                                .build();
```

점층적 생성자 패턴보다 코드가 장황하다는 이유로 책에서는 매개변수가 4개 이상인 상황에서 사용하기를 권하지만 롬복의 @Builder 어노테이션 하나면 깔끔하게 정리되므로 빌더를 안 쓸 이유가 없는 것 같다.  

---
@Builder 어노테이션을 사용하면서 조금 더 다양한 기능을 사용하기 위해 아래의 내용을 기록한다.  
### toBuilder()
빌더의 toBuilder 속성을 true로 하면 toBuilder() 메서드를 사용할 수 있다.  
toBuilder()는 기존 객체의 값으로 빌더를 초기화한다.
```java
NutritionFacts nutritionFacts = NutritionFacts.builder(240, 8)
                                            .fat(8)
                                            .build();

NutritionFacts updateNutritionFacts = nutritionFacts.toBuilder()
                                            .fat(10)
                                            .build();
```
### @Builder에서 List나 Set 등 활용하기
빌더 패턴을 사용하고 List 타입의 필드에 값을 지정하려면 자료형을 맞춰줘야만 값을 넣을 수 있었다.
```java
@Builder
public class User {
    private String name;
    private List<Role> roles;
}

User user = User.builder()
                    .name("chamsae")
                    .roles(Arrays.asList(ADMIN, GUEST))
                    .build();
```
위의 상황에서는 역할이 여러 가지이므로 roles에 [ADMIN, GUEST]을 통째로 넣었지만,역할이 하나라면 사이즈가 1인 List로 만들어서 넣는 것이
마냥 예뻐보이지는 않는다.  
@Singular 어노테이션을 필드에 붙여주면 해결할 수 있다.
```java
@Builder
public class User {
    private String name;
    @Singular
    private List<Role> roles;
}

User user = User.builder()
                    .name("chamsae")
                    .role(GUEST)
                    .build();
```
@Singular 어노테이션을 붙였기 때문에 하나씩 추가하는 것이 가능하다.  
바이트 코드를 열어보면 내부에서 List를 만들거나 이미 roles 변수에 List가 할당되어 있다면 해당 List에 값을 add한다.

Singular는 일부 collection type에 대해서만 사용가능하고 자세한 타입은
[@Singular 사용 가능한 타입](https://projectlombok.org/features/Builder#:~:text=%40Singular%20can%20only,of%20ImmutableTable)
에서 확인할 수 있다.

### 빌더의 DefaultValue
class에 @Builder 어노테이션을 붙여두면 멤버 변수의 초깃값을 지정할 수 있다.
```java
@Builder
public class Circle {
    @Builder.Default
    private Double radius = 0.0;
    private String name;
    @Singular
    private List<Integer> numbers;
}
```

radius()를 호출하지 않아도 0.0이라는 값이 잘 들어간다.